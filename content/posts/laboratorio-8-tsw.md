
++
title = "LAB 8 - Implementando a funcionalidade de Login com Spring Security"
description = "Passo a passo para adicionar autenticação via formulário na camada web usando Spring Security (starter)."
date = 2025-10-22
draft = false
author = "Enoque Leal"
tags = [ "java", "spring", "security", "login", "h2" ]
++

## Objetivo

Neste laboratório iremos implementar a funcionalidade de autenticação pora os formulários da camada web da aplicação `carstore-spring-boot`. Para isso, iremos utilizar o Spring Security Starter. O foco é permitir que usuários façam login via uma página HTML (Thymeleaf) e que a sessão seja mantida pelo container de sessão do Spring Security.

Ao final você deverá saber:

- Incluir dependências do Spring Security no `pom.xml`;
- Configurar um `SecurityConfig` moderno (filter chain) com `PasswordEncoder`;
- Criar um formulário de login com Thymeleaf e integrar ao fluxo do Spring Security;
- Fornecer um usuário em memória para testes;
- Testar a autenticação via navegador usando a UI e o console H2.


## Pré-requisitos

- Ter completado os laboratórios anteriores (Parte 1 JDBC e Parte 2 JPA);
- Projeto `carstore-spring-boot` compilando localmente;
- IDE (IntelliJ/VS Code) com Maven e JDK 11+ configurados.


## Visão geral das tarefas

1. Adicionar dependência `spring-boot-starter-security` no `pom.xml`.
2. Criar `SecurityConfig` com `SecurityFilterChain` e `PasswordEncoder` (BCrypt).
3. Criar um template `login.html` e roteador `LoginController`.
4. Testar com um usuário em memória.

---

### Tarefa 1: Importando a dependência do Spring Security

No `pom.xml` do projeto `carstore-spring-boot`, adicione os starters `spring-boot-starter-security` e `thymeleaf-extras-springsecurity6`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
```

### Tarefa 2: Criando a classe de configuração do Spring Security

Crie a classe `SecurityConfig` no pacote `src/main/java/br/com/carstore/config/`.

Esta configuração irá:

1. Permitir o acesso a recursos estáticos e rotas públicas.

2. Exigir autenticação (`.authenticated()`) apenas para rotas que iniciam com `/admin/**`.

3. Permitir todo o resto (`.anyRequest().permitAll()`).

```java
package br.com.carstore.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity; 
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // 1. Permite acesso a recursos estáticos (CSS, JS, Imagens, etc.)
                .requestMatchers("/css/**", "/js/**", "/images/**").permitAll()
                
                // 2. REQUER AUTENTICAÇÃO para qualquer rota que comece com /admin/**
                .requestMatchers("/admin/**").authenticated()
                
                // 3. Todas as outras rotas (incluindo /, /login, /home) são permitidas (públicas)
                .anyRequest().permitAll()
            )
            .formLogin(form -> form
                .loginPage("/login")               // URL para exibir o formulário de login customizado
                .failureUrl("/login?error")        // URL em caso de falha no login
                // Redireciona para a home após login (Se não houver URL de origem armazenada)
                .defaultSuccessUrl("/", true)      
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")              // URL que o formulário de logout fará POST
                .logoutSuccessUrl("/login?logout") // Redireciona após logout bem-sucedido
            )
            // Desativa CSRF para simplificar o desenvolvimento. Mantenha em produção se usar formulários.
            .csrf(AbstractHttpConfigurer::disable); 

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        // Usa BCrypt, que é o padrão seguro para hashear senhas.
        return new BCryptPasswordEncoder();
    }
}
```

### Tarefa 3: Criando um usuário para testes (in-memory)

Para testar o fluxo de autenticação rapidamente, vamos configurar um usuário em memória. Adicione o seguinte método @Bean dentro da classe SecurityConfig criada na Tarefa 2.

```java
// Este método deve ser adicionado DENTRO da classe SecurityConfig
@Bean
public org.springframework.security.core.userdetails.UserDetailsService users(PasswordEncoder passwordEncoder) {
  
  // Detalhes do usuário de teste
  org.springframework.security.core.userdetails.UserDetails user = 
    org.springframework.security.core.userdetails.User.builder()
      .username("admin")
      // A senha 'admin' será codificada pelo BCryptPasswordEncoder
      .password(passwordEncoder.encode("admin")) 
      .roles("USER", "ADMIN") // Roles para uso futuro em autorização
      .build();
  
  // Gerenciador em memória (apenas para testes)
  return new org.springframework.security.provisioning.InMemoryUserDetailsManager(user);

}
```

Credenciais de teste: `admin` / `admin`

### Tarefa 4: Criando a página de login com Thymeleaf

Crie o template `src/main/resources/templates/login.html`. O formulário deve ter `action="@{/login}"`, `method="post"`, e campos `name="username"` e `name="password"`.

```html
<!doctype html>
<html xmlns:th="[http://www.thymeleaf.org](http://www.thymeleaf.org)">
  <head>
    <meta charset="UTF-8" />
    <title>CarStore</title>
    <link rel="stylesheet" th:href="@{/css/main.css}" />
  </head>
  <body>
    <h1>Acesso Restrito</h1>

    <form th:action="@{/login}" method="post">
      <div>
        <label for="username">Usuário</label>
        <input type="text" id="username" name="username" required/>
      </div>
      <div>
        <label for="password">Senha</label>
        <input type="password" id="password" name="password" required/>
      </div>
      <div>
        <button type="submit">Entrar</button>
      </div>
      
      <div th:if="${param.error}" style="color: red; margin-top: 10px;">
        Usuário ou senha inválidos.
      </div>
      <div th:if="${param.logout}" style="color: green; margin-top: 10px;">
        Você saiu com sucesso.
      </div>
    </form>
  </body>
</html>

```

### Tarefa 5: Criando a Controller responsável pelo fluxo de login

Crie a classe `LoginController` no pacote `src/main/java/br/com/carstore/controller` para resolver explicitamente a rota GET `/login`.

```java
package br.com.carstore.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {

        return "login"; // templates/login.html

    }

}

```

### Tarefa 6: Validando a implementação

1. Inicie a aplicação via CLI utilizando o comando `mvn spring-boot:run` ou pela IDE.

2. Teste 1 - Acesso Público: Acesse http://localhost:8080/. O acesso deve ser liberado.

3. Teste 2 - Acesso Restrito: Tente acessar uma rota restrita, como http://localhost:8080/admin/. Você deverá ser redirecionado para http://localhost:8080/login pois trata-se de uma rota protegida pelo spring security.

4. Faça o login: Na página de login, use as credenciais: admin / admin.

5. Acesso Pós-Login: Após o login, você deve ser levado à rota que tentou acessar originalmente (/admin/cars) ou, caso contrário, para a rota de sucesso padrão (/).

6. Logout: Para testar o logout, adicione na navbar que foi criada na fragment, o formulário de POST para a rota de logout:

```html
<form th:action="@{/logout}" method="post">
    <button type="submit">Sair do Sistema</button>
</form>
```

Após clicar, você deve ser redirecionado para `http://localhost:8080/login?logout`.

## Conclusão e próximos passos

Neste laboratório você configurou o Spring Security, protegeu rotas de administração `(/admin/**)` e implementou o fluxo de autenticação por formulário com Thymeleaf.


### LAB 9 (breve visão)

Objetivos do LAB 9:

- Proteger endpoints REST (URL base `/api/**`);
- Implementar autenticação baseada em JWT e cabeçalhos Authorization;

---

Parabéns — Você implementou a jornada de login na camada web! :)
