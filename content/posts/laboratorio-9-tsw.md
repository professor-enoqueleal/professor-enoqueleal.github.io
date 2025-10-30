+++
title = "LAB 9 - Protegendo a Camada API (REST) com Spring Security e JWT"
description = "Guia passo a passo para configurar a segurança em endpoints REST, utilizando o Spring Security de forma stateless e implementando autenticação baseada em JSON Web Tokens (JWT)."
date = 2025-10-30
draft = false
author = "Enoque Leal"
tags = [ "java", "spring", "security", "api", "rest", "jwt", "stateless" ]
+++

## 🔒 Objetivo

Neste laboratório, iremos configurar o Spring Security para proteger a camada de API da aplicação `carstore-spring-boot`. Ao contrário da camada web (LAB 8), a API será **stateless** (sem sessão) e usará o protocolo de autenticação **JWT (JSON Web Tokens)**.

Ao final deste laboratório, você deverá saber:

- Configurar o Spring Security para um fluxo **stateless** (sem sessão).
- Criar um **Endpoint de Autenticação** (`/api/auth/login`) que retorna um JWT.
- Implementar um **Filtro de Autorização JWT** para validar tokens em todas as requisições da API.
- Proteger rotas específicas da API (ex: `/api/admin/**`).
- Testar a autenticação de API usando ferramentas como Postman.

## Pré-requisitos

- Ter completado os laboratórios anteriores (LAB 7 e LAB 8).
- Projeto `carstore-spring-boot` compilando localmente.
- Ferramenta para testar APIs (Postman).

---

## Parte A: Configurando a Camada de API (Stateless)

### Tarefa 1: Dependências Adicionais

Adicione a dependência do JWT no `pom.xml` da aplicação:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

```

### Tarefa 2: Implementação da Classe LoginRequest (DTO)

Crie uma nova classe DTO chamada `LoginRequest` em um pacote apropriado para DTOs, por exemplo, `br.com.carstore.dto.LoginRequest.java`:

```java

package br.com.carstore.dto;

public class LoginRequest {

    private String username;
    private String password;

    // Construtor vazio (necessário para desserialização do JSON)
    public LoginRequest() {
    }

    // Construtor com todos os campos (útil para testes)
    public LoginRequest(String username, String password) {
        this.username = username;
        this.password = password;
    }

    // Getters e Setters

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

}
```

### Tarefa 3: Refatorando a classe SecurityConfig (Separação Web vs API)

Na classe `SecurityConfig.java`, precisamos criar um novo método filter para a proteção da camada de API (stateless e JWT).

Para isso, a baixo do método `filterChain`, crie um novo método chamado `apiFilterChain`, conforme o exemplo a seguir:

```java

// ... imports existentes ...
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher; // Necessário para Spring Security 6+

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {

        // Regra 1: Configuração para API (Stateless) - Rotas /api/**
        http.securityMatcher("/api/**") // Aplica ESTA regra SOMENTE a /api/**
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Sem sessão no servidor
                )
                
                // Neste ponto, adicionaremos o Filtro JWT posteriormente.

                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/auth/**").permitAll() // Endpoint de Login API é público
                        .anyRequest().authenticated() // Todas as outras rotas API exigem autenticação
                );
            

        return http.build();

    }

    // Método já existente
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // Conteúdo omitido ...
    }

    // ... (Seus métodos @Bean passwordEncoder e users já existentes) ...

}

```

## Parte B: Implementação do JWT (JSON Web Tokens)

### Tarefa 4: Criação do JWT Token Service

Agora iremos cliar a classe de serviço que será responsável por gerar, assinar e validar o token JWT.

No pacote service, crie uma nova classe chamada `JwtTokenService` em `src/main/java/br/com/carstore/service/JwtTokenService.java`:

```java
package br.com.carstore.service;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

import java.security.Key;
import java.util.Date;

@Service
public class JwtTokenService {
    
    // ATENÇÃO: Use uma chave secreta forte. Esta é apenas um exemplo.
    private final String SECRET_KEY = "suaChaveSecretaDePeloMenos256BitsParaAssinaturaJWT"; // Mudar no app properties!
    private final long EXPIRATION_TIME = 300000; // 5 minutos em milissegundos

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public String generateToken(UserDetails userDetails) {

        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();

    }

    public String extractUsername(String token) {
        // Implementar lógica de extração de username do token
        // Jwts.parserBuilder()...
        return Jwts.parserBuilder().setSigningKey(getSigningKey()).build().parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        // Implementar lógica de validação: username e expiração
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    private boolean isTokenExpired(String token) {
        // Implementar lógica de verificação de expiração
        return Jwts.parserBuilder().setSigningKey(getSigningKey()).build().parseClaimsJws(token).getBody().getExpiration().before(new Date());
    }

}
```

### Tarefa 5: Criação do Controller de Autenticação (API)

Crie a classe Controller que será responsável por receber o login e senha e retornar o token.

No pacote controller, crie uma nova classe chamada `AuthController` em `src/main/java/br/com/carstore/api/AuthController.java`:

```java
package br.com.carstore.api;

import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtTokenService jwtTokenService;

    public AuthController(AuthenticationManager authenticationManager, JwtTokenService jwtTokenService) {
        this.authenticationManager = authenticationManager;
        this.jwtTokenService = jwtTokenService;
    }

    @PostMapping("/login")
    public ResponseEntity<String> authenticateUser(@RequestBody LoginRequest loginRequest) {
        
        // 1. Tenta autenticar o usuário
        Authentication authentication = authenticationManager.authenticate(

            new UsernamePasswordAuthenticationToken(
                loginRequest.getUsername(),
                loginRequest.getPassword()
            )

        );

        // 2. Se a autenticação for bem-sucedida, gera o token
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        String jwt = jwtTokenService.generateToken(userDetails);

        // 3. Retorna o token como resposta (pode ser um DTO JSON mais completo)
        return ResponseEntity.ok(jwt);
        
    }

}
```

**Nota**: Para que o Spring consiga injetar o `AuthenticationManager`, nós iremos expor um `@Bean` na classe `SecurityConfig`.


### Tarefa 6: Criação do Filtro JWT

Agora vamos implementar o filtro que irá intercepta todas as requisições da API para validar o token antes que a requisição chegue ao `Controller`.

No pacote config, crie uma nova classe chamada `JwtRequestFilter` em `src/main/java/br/com/carstore/config/JwtRequestFilter.java`:

```java

package br.com.carstore.config;

import br.com.carstore.service.JwtTokenService;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    private final UserDetailsService userDetailsService; // O mesmo usado no LAB 8 (autenticação in-memory)
    private final JwtTokenService jwtTokenService;

    public JwtRequestFilter(UserDetailsService userDetailsService, JwtTokenService jwtTokenService) {
        this.userDetailsService = userDetailsService;
        this.jwtTokenService = jwtTokenService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        final String authorizationHeader = request.getHeader("Authorization");

        String username = null;
        String jwt = null;

        // 1. Extrai o token do cabeçalho
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {

            jwt = authorizationHeader.substring(7);

            try {
                username = jwtTokenService.extractUsername(jwt);
            } catch (Exception e) {
                logger.warn("JWT Token inválido ou expirado.");
                // Continua o filtro; a exceção de autenticação será lançada depois.
            }

        }

        // 2. Valida o token e define a autenticação no contexto
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);

            if (jwtTokenService.validateToken(jwt, userDetails)) {
                
                // Cria o objeto de autenticação
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                
                // Define o usuário no contexto de segurança
                SecurityContextHolder.getContext().setAuthentication(authToken);

            }

        }

        filterChain.doFilter(request, response);

    }

}
```

### Tarefa 7: Integrando o Filtro no SecurityConfig

Agora voltamos a classe `SecurityConfig.java` para adicionar o `JwtRequestFilter` à nossa cadeia de filtros da API `apiFilterChain`. O filtro deve ser executado antes do filtro de autenticação padrão do Spring.

Ajuste o método `filterChain` no `SecurityConfig.java`:

```java

import br.com.carstore.config.JwtRequestFilter; 
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter; // Para definir a ordem

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    // Injetar o novo filtro
    private final JwtRequestFilter jwtRequestFilter;

    public SecurityConfig(@Lazy JwtRequestFilter jwtRequestFilter) {
        this.jwtRequestFilter = jwtRequestFilter;
    }

    @Bean
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        // Regra 1: Configuração para API (Stateless) - Rotas /api/**
        http.securityMatcher("/api/**") 
            // ... (Configurações de CSRF e SESSION_STATELESS) ...
            // ADIÇÃO DO FILTRO JWT
            .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class)
            // ... (Configuração authorizeHttpRequests da API) ...
            
        return http.build();
    }
    // ...
    
    // Expondo o AuthenticationManager como Bean (Necessário para AuthController)
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
        return authenticationConfiguration.getAuthenticationManager();
    }

}
```

## 🧪 Testes Manuais (Postman)

1. Reinicie a aplicação.

2. Faça uma requisição HTTP para obter um Token JWT válido para o login:

    * Método: POST

    * URL: http://localhost:8080/api/auth/login

    * Body (JSON): 
        ```json 
        { 
            "username": "admin", 
            "password": "admin" 
        }
        ```

    * Resultado: Você deve receber o JWT (uma string longa) como resposta. Salve este token.

3. Acessar recursos protegidos da API (Ex: /api/admin/cars):

    * Método: GET

    * URL: http://localhost:8080/api/admin/cars

    * Sem cabeçalho (Teste de Falha): A requisição deve falhar com status 401 Unauthorized.

    * Com cabeçalho (Teste de Sucesso):

        * Header: Authorization

        * Valor: Bearer [SEU_TOKEN_AQUI]

### Conclusão e Próximos Passos

Neste laboratório, você separou o fluxo de segurança, criando uma camada de API stateless protegida por tokens JWT. Você implementou a geração, validação e aplicação do token em um filtro customizado do Spring Security.

No LAB 10, iremos:

* Substituir o UserDetailsService em memória por uma implementação que busca usuários e roles no banco de dados (via JPA).

* Usar anotações de autorização (`@PreAuthorize`, `@Secured`) nos Controllers da API para restringir o acesso com base nos Roles (ex: ```ROLE_ADMIN```).

---

Parabéns — Você implementou a jornada de autenticaçãio na camada de API! :)
