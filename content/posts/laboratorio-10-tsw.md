+++
title = "LAB 10 - Melhorando a protenção do sistema com ROLEs"
description = "Substituir UserDetailsService em memória por implementação que busca usuários e roles no banco (JPA) e proteger controllers com anotações de autorização." 
date = 2025-11-06
draft = false
author = "Enoque Leal"
tags = [ "java", "spring", "security", "jpa", "roles", "authorization" ]
+++

## 🎯 Objetivo

Neste laboratório vamos substituir a implementação em memória do **UserDetailsService** por uma implementação que carrega usuários e roles do banco de dados usando JPA. Em seguida, veremos como usar anotações de método (**@PreAuthorize**, **@Secured**) nos Controllers da API para restringir o acesso com base nas roles (ex: **ROLE_ADMIN**).

Ao final você deverá saber:

- Criar entidades JPA para **User** e **Role** e os repositórios correspondentes;
- Implementar um **UserDetailsService** que carrega dados via JPA;
- Expor **PasswordEncoder** e configurar o **AuthenticationManager** para usar o **UserDetailsService** JPA;
- Proteger métodos de controllers com **@PreAuthorize** e **@Secured** e habilitar a segurança por método;
- Popular dados iniciais (roles e usuários) e testar o acesso.

## Pré-requisitos

- Ter completado LAB 7 (integração com banco) e LAB 8 (segurança básica);
- Projeto **carstore-spring-boot** compilando localmente;
- H2 (ou outro banco) configurado para desenvolvimento (se usou o LAB 7, já está pronto).

---

## Contrato (curto)

- Entrada: requests HTTP autenticados com username/password via login tradicional (form) ou através do fluxo já existente;
- Saída: usuários autenticados vindos do banco com roles atribuídas que definem autorização por método;
- Erros: usuário não encontrado -> 401; acesso negado -> 403;
- Critério de sucesso: endpoints anotados com **@PreAuthorize** e **@Secured** aceitam/negam acesso conforme roles inseridas no banco.

## Panorama das alterações (arquivos-chave)

- src/main/java/br/com/carstore/model/RoleEntity.java
- src/main/java/br/com/carstore/model/UserEntity.java
- src/main/java/br/com/carstore/repository/RoleRepository.java
- src/main/java/br/com/carstore/repository/UserRepository.java
- src/main/java/br/com/carstore/service/JpaUserDetailsService.java
- src/main/java/br/com/carstore/config/SecurityConfig.java (ajustes)
- src/main/java/br/com/carstore/config/DataInitializer.java (CommandLineRunner para popular dados)
- Controllers: exemplos com **@PreAuthorize** / **@Secured**.

> Observação: muitos projetos já possuem H2 e JPA configurados (veja LAB 7). Se faltar algo em **application.properties**, veja a seção "Configuração" abaixo.

---
## Tarefa 1: Model e Repositórios (JPA)

Crie a entidade **RoleEntity** no pacote **br.com.carstore.model**:

- Campos principais: **id** (Long), **name** (String, ex: **ROLE_ADMIN**, **ROLE_USER**).
- Relacionamento ManyToMany bidirecional com **UserEntity** (opcional manter unidirecional dependendo do seu design).

Exemplo de classe (resumo):

```java
package br.com.carstore.model;

import jakarta.persistence.*;
import java.util.Set;

@Entity
@Table(name = "roles")
public class RoleEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name; // ex: ROLE_ADMIN

    public RoleEntity() {
        
    }

    public RoleEntity(Long id, String name) {
        this.id = id;
        this.name = name;
    }
    
    // ... getters e setters ...
}
```

> Coloque a entidade em **src/main/java/br/com/carstore/model/RoleEntity.java**.


### Parte 2: Entidade **User** (UserEntity)

Crie a entidade **UserEntity** com os campos mínimos **id**, **username**, **password** e **roles** (ManyToMany).

Exemplo resumido:

```java
package br.com.carstore.model;

import jakarta.persistence.*;
import java.util.Set;

@Entity
@Table(name = "users")
public class UserEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password; // armazenado com BCrypt

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
      name = "users_roles",
      joinColumns = @JoinColumn(name = "user_id"),
      inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<RoleEntity> roles;

    // ... getters e setters ...
}
```

> Coloque em **src/main/java/br/com/carstore/model/UserEntity.java**.


### Parte 3: Repositórios

Crie **RoleRepository** e **UserRepository** no pacote **br.com.carstore.repository**:

- **RoleRepository extends JpaRepository<RoleEntity, Long>** com método opcional **Optional<RoleEntity> findByName(String name)**;
- **UserRepository extends JpaRepository<UserEntity, Long>** com **Optional<UserEntity> findByUsername(String username)**.

Exemplos (resumo):

```java
package br.com.carstore.repository;

import br.com.carstore.entity.RoleEntity;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface RoleRepository extends JpaRepository<RoleEntity, Long> {

    Optional<RoleEntity> findByName(String name);

}

```

```java
package br.com.carstore.repository;

import br.com.carstore.entity.UserEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<UserEntity, Long> {
    
    Optional<UserEntity> findByUsername(String username);
    
}
```

> Arquivos: **src/main/java/br/com/carstore/repository/RoleRepository.java** e **UserRepository.java**.

---

## Terefa 2: Implementando o **UserDetailsService** via JPA

### Parte 1: Serviço **JpaUserDetailsService**

No pacote **br.com.carstore.service** crie a classe **JpaUserDetailsService** que implementa **org.springframework.security.core.userdetails.UserDetailsService**.

Responsabilidades:
- Carregar **UserEntity** pelo username usando **UserRepository**.
- Mapear **RoleEntity.name** para **SimpleGrantedAuthority** (atenção ao prefixo **ROLE_** — mantenha a convenção).
- Retornar um **org.springframework.security.core.userdetails.User** (ou um UserDetails customizado) com username, password e authorities.

Exemplo resumido:

```java
package br.com.carstore.service;

import br.com.carstore.entity.UserEntity;
import br.com.carstore.repository.UserRepository;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.stream.Collectors;

@Service
public class JpaUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public JpaUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserEntity user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado: " + username));

        var authorities = user.getRoles().stream()
                .map(r -> new SimpleGrantedAuthority(r.getName()))
                .collect(Collectors.toSet());

        return new org.springframework.security.core.userdetails.User(
                user.getUsername(), user.getPassword(), authorities
        );
    }
}
```

> Arquivo: **src/main/java/br/com/carstore/service/JpaUserDetailsService.java**.


### Parte 2: Remover/Alterar UserDetailsService em memória

Se o seu projeto ainda declara um **UserDetailsService** em memória (ex: **InMemoryUserDetailsManager** no **SecurityConfig**), remova ou comente essa configuração. Em vez disso, o **JpaUserDetailsService** será detectado pelo Spring como bean **UserDetailsService**.

---

## Tarefa 3: Segurança: beans e habilitação de método

Agora precisamos ajustar a implementação da classe **SecurityConfig**

Para isso, abra a classe **SecurityConfig** e adicione a anotacão **@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)**. O restante da implementação, deve permanecer inalterado.

```java
package br.com.carstore.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;

@Configuration
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // implementação existente...
    }

    @Bean
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        // implementação existente...
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
        return authenticationConfiguration.getAuthenticationManager();
    }
}
```

> Observação: não é necessário referenciar explicitamente o **UserDetailsService** aqui; o Spring usará o bean **JpaUserDetailsService** automaticamente para autenticação. Portanto, remova o método users()
> 
```java
// remova esse método ou deixe comentado
@Bean
public UserDetailsService users(PasswordEncoder passwordEncoder) { ... }
```

---

## Tarefa 4: Protegendo controllers com anotações

### Parte 1: Usando **@PreAuthorize**

No seu Controller da API (ex: **CarController** ou **AdminController**) você pode usar **@PreAuthorize** acima do método:

```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/api/admin/stats")
public ResponseEntity<?> stats() { return ResponseEntity.ok().build(); }
```

**@PreAuthorize** usa SpEL — é muito expressivo. Exemplos:
- **@PreAuthorize("hasRole('ADMIN')")** — apenas ADMIN;
- **@PreAuthorize("hasAnyRole('ADMIN','MANAGER')")**;
- **@PreAuthorize("hasAuthority('ROLE_ADMIN') and #id == principal.username")** (exemplo com expressão avançada).

### Parte 2: Usando **@Secured**

**@Secured** é mais simples e exige nomes com prefixo **ROLE_**:

```java
@Secured("ROLE_ADMIN")
@PostMapping("/api/admin/create")
public ResponseEntity<?> create() { return ResponseEntity.ok().build(); }
```

> Lembre-se de habilitar **securedEnabled = true** em **@EnableMethodSecurity** (veja Parte C).

---

## Tarefa 5: Popular o banco (DataInitializer)

Na classe **StartupRunner**, adicione o método **init(...)** para popular roles e usuários iniciais. 

```java
package br.com.carstore.runner;

import br.com.carstore.dao.CarJpaDao;
import br.com.carstore.entity.RoleEntity;
import br.com.carstore.model.CarDTO;
import br.com.carstore.repository.RoleRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import br.com.carstore.entity.UserEntity;
import br.com.carstore.repository.UserRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.password.PasswordEncoder;


import java.util.Set;

@Component
public class StartupRunner implements CommandLineRunner {

    private final CarJpaDao carDao;

    public StartupRunner(CarJpaDao carDao) {
        this.carDao = carDao;
    }

    @Override
    public void run(String... args) throws Exception {
        carDao.save(new CarDTO("Onix", "Vermelho", "Chevrolet", "LTZ", "2020", "2021"));
        carDao.save(new CarDTO("Celta", "Cinza", "Chevrolet", "LT", "2008", "2009"));
        System.out.println(carDao.findAll());
    }

    @Bean
    public CommandLineRunner init(RoleRepository roleRepo, UserRepository userRepo, PasswordEncoder encoder) {
        return args -> {
            if (roleRepo.findByName("ROLE_ADMIN").isEmpty()) {
                roleRepo.save(new RoleEntity(null, "ROLE_ADMIN"));
            }
            if (roleRepo.findByName("ROLE_USER").isEmpty()) {
                roleRepo.save(new RoleEntity(null, "ROLE_USER"));
            }

            if (userRepo.findByUsername("admin").isEmpty()) {
                RoleEntity adminRole = roleRepo.findByName("ROLE_ADMIN").orElseThrow();
                RoleEntity userRole = roleRepo.findByName("ROLE_USER").orElseThrow();

                UserEntity admin = new UserEntity();
                admin.setUsername("admin");
                admin.setPassword(encoder.encode("admin"));
                admin.setRoles(Set.of(adminRole, userRole));

                userRepo.save(admin);
            }

            if (userRepo.findByUsername("user").isEmpty()) {
                RoleEntity userRole = roleRepo.findByName("ROLE_USER").orElseThrow();

                UserEntity user = new UserEntity();
                user.setUsername("user");
                user.setPassword(encoder.encode("user"));
                user.setRoles(Set.of(userRole));

                userRepo.save(user);
            }
        };
    }

}
```

> Senha de exemplo: **admin** / **user**. Em produção, use senhas fortes e mecanismos de cadastro/autenticação adequados.

---

## Tarefa 6: Configuração mínima (**application.properties**)

Se necessário, adicione as seguintes propriedades para habilitar JPA e o console H2 (ajuste conforme seu setup):

```
# Datasource (se estiver usando H2 em file/Memory) - ver LAB 7
spring.datasource.url=jdbc:h2:~/test;AUTO_SERVER=TRUE
spring.datasource.username=sa
spring.datasource.password=sa
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA
spring.jpa.hibernate.ddl-auto=drop-and-create
spring.jpa.show-sql=true
```

---

## Tarefa 7: Testes manuais

1. Rode a aplicação: **mvn spring-boot:run** (ou diretamente pela IDE).
2. Verifique no console que os roles e usuários foram criados (ou consulte o H2 console).
3. Teste login / endpoints:
   - Acesse um endpoint protegido por role admin (ex: **GET /api/admin/stats**) sem autenticação -> deverá retornar 401/403.
   - Autentique (pelo formulário web ou endpoint **/login**) como **admin**/**admin123** e acesse o endpoint -> deverá permitir.
   - Tente acessar com **user**/**user123** -> deverá ser negado para endpoints **ROLE_ADMIN**.

Exemplo com curl (se sua app usa sessão/form-login):

### Parte 1 — Solicitar login via form (exemplo genérico, ajuste conforme sua app)

```shell
curl -v -X POST -d "username=admin&password=admin123" http://localhost:8080/login
```

### Parte 2 — Acessar endpoint protegido com cookie/CSRF (ou use o fluxo JWT se tiver implementado)

> Observação: se você estiver usando a API stateless do LAB 9 com JWT, adapte o fluxo: gere o token e envie no header **Authorization: Bearer <token>**.

---

## Tarefa 8: Protegendo a camada Web (Thymeleaf + Spring Security)

Nesta seção final mostramos, de forma prática, como proteger a camada web (views Thymeleaf)

### Parte 1: Configurar namespace do Spring Security no Thymeleaf
Agora que já temos nossa camada de API devidamente protegida, chegou o momento de garantir que a interface web também respeite as roles dos usuários autenticados. Dessa maneira, podemos melhorar a experiência do usuário, exibindo ou ocultando elementos da UI conforme as permissões atribuídas.

Para isso, siga os seguintes passos, no arquivo **src/main/resources/templates/admin/dashboard.html**, troque o importe padrão do thymeleaf, pelo namespace do Spring Security:

```html
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

### Parte 2: Esconder/mostrar elementos na view com **sec:authorize**

Com o objetivo de proteger o sistema, o menu de atulizar ou deletar um carro devem ser exibidos apenas para usuários com a role **ROLE_ADMIN**.

Para isso, utilize o atributo **sec:authorize** nos elementos HTML que deseja proteger.

```html
<td>

    <a th:href="@{/admin/cars/edit(id=${car.id})}" class="btn btn-secondary btn-sm" sec:authorize="hasRole('ADMIN')">Update</a>

    <form th:action="@{/admin/cars/delete}" method="post" style="display:inline" sec:authorize="hasRole('ADMIN')">
        <input type="hidden" name="id" th:value="${car.id}" />
        <button type="submit" class="btn btn-danger btn-sm">Delete</button>
    </form>
    
</td>
```

Vamos proteger também o menu de navegação para criar um novo carro:

Para isso, siga os seguintes passos, no arquivo **src/main/resources/templates/fragments.html** e envolva o item de menu com a tag **sec:authorize**:

```html
<li class="nav-item" sec:authorize="hasRole('ADMIN')">
  <a class="nav-link" th:href="@{/admin}">Novo Carro</a>
</li>
```

### Parte 3: Proteção obrigatória no servidor

Mesmo que um botão não apareça na UI, um usuário pode enviar requisições diretamente, portando, vamos proteger também os métodos da controller **AdminController**:

```java
@PreAuthorize("hasRole('ADMIN')")
@PostMapping("/admin/cars")
public String createCar(@ModelAttribute CarDTO car, Model model) { ... }

@PreAuthorize("hasRole('ADMIN')")
@PostMapping("/admin/cars/delete")
public String deleteCar(@RequestParam("id") String id, Model model) { ... }
```

### Parte 4: Testes recomendados

- Rodar a aplicação (**mvn spring-boot:run**);
- Logar com usuário sem **ROLE_ADMIN** e verificar que botões protegidos não aparecem;
- Logar com **admin** e verificar que os botões aparecem;
- Tentar executar POST protegido com sessão de usuário sem **ROLE_ADMIN** — servidor deve retornar 403.

Boas práticas

- **sec:authorize** melhora a UX, mas a autorização real deve ocorrer no backend;
- Em frontends separados (SPA) garanta checagem de roles no backend (JWT/OAuth2).

Arquivos de referência (alterados no exemplo)

- **src/main/resources/templates/fragments.html** — navbar com **sec:authorize** e bloco diagnóstico temporário;
- **src/main/resources/templates/admin/dashboard.html** — namespace **xmlns:sec** e proteção de botões/links;
- **src/main/java/br/com/carstore/controller/AdminController.java** — métodos sensíveis anotados com **@PreAuthorize("hasRole('ADMIN')")**.

---

## Questões comuns / dicas

- Prefixo **ROLE_**: por convenção as roles devem ter **ROLE_** no nome quando usadas com **@Secured**; com **@PreAuthorize("hasRole('ADMIN')")** o Spring adiciona o prefixo automaticamente.
- PasswordEncoder: sempre armazene senhas com **BCryptPasswordEncoder** ou outro encoder seguro; caso migre de texto puro, faça uma migração controlada.
- Busca de authorities: garanta **FetchType.EAGER** nas roles (ou carregue explicitamente) para evitar **LazyInitializationException** durante a autenticação.
- Em ambientes de produção, não utilize **spring.jpa.hibernate.ddl-auto=update** sem entender as implicações.

---

### Conclusão

Neste laboratório você aprendeu a integrar Spring Security com JPA para carregar usuários e roles do banco de dados, além de proteger endpoints da API e elementos da interface web com base nas roles dos usuários. Essas práticas são essenciais para construir aplicações seguras e robustas.

---
Parabéns ✨

Você concluiu o LAB 10 e está pronto para construir seu sistema com roles e autorização baseada em Spring Security!

