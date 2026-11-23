+++
title = "LAB 7 - Integração com banco de dados via JPA - PARTE 2"
description = "Guia passo a passo para integrar uma aplicação Spring Boot com um banco relacional usando JDBC (JdbcTemplate) e JPA (Jakara Persistence / Hibernate)."
date = 2025-10-16
draft = true
author = "Enoque Leal"
tags = [ "java", "spring", "jpa", "jdbc", "h2" ]
+++

## Visão geral e objetivos do laboratório

Neste laboratório vamos adicionar e configurar a persistência na nossa aplicação spring-boot   (`carstore-spring-boot`). O objetivo é apresentar duas abordagens comuns para trabalhar com bancos relacionais em aplicações Spring:

- Uso de JPA (Jakarta Persistence / Hibernate) diretamente via `EntityManager`, com JPA puro.

Ao final desse laboratório você deverá ser capaz de:

- Adicionar dependências e configurar um banco H2 para desenvolvimento;
- Usar `JdbcTemplate` para operações básicas (insert, select, update, delete);
- Criar entidades JPA e usar `EntityManager` para persistência;
- Testar a aplicação e acessar o console H2.


## Pré-requisitos

* JDK 11+ e Maven instalados;
* IDE (IntelliJ, VS Code) para editar/rodar o projeto.
* Projeto do Laboratório 5 concluído


## Visão geral dos passos

Esta atividade foi organizada em duas partes independentes, Parte 1 implementando com **JDBC** e Parte 2 implementando com **JPA** (Java Persistence API).

Parte 2 — JPA (EntityManager)
1. Adicionar dependências JPA (provider) no `pom.xml`.
2. Ajustar `application.properties` com propriedades JPA (ddl-auto, dialect) se necessário.
3. Criar a `@Entity` `CarEntity` e refatorar o `Service` para usar `EntityManager`.
4. Testar localmente (validar persistência via JPA e transações).

Na parte 1 dele laboratório, implementamos a comunicação com o banco de dados utilizando JDBC.

Agora na parte 2, iremos avançar no nível de implementação, para utilizar o JPA (Java Persistence API)

## PARTE 2 — JPA (EntityManager)

Complete a Parte 1 antes de seguir para a Parte 2. A Parte 2 demonstra JPA puro com `EntityManager`.

### Tarefa 1: Adicionando a dependências do JPA

1. Adicione as dependências do *starter* do JPA para Spring Boot, conforme exemplo abaixo:

```xml
<!-- H2 (runtime para desenvolvimento) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Suporte JPA (inclui Hibernate como provider) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

2. Salve as alterações no arquivo `pom.xml`


### Tarefa 2: Configurando o JPA

Adicione as propriedades JPA no arquivo `application.properties`:

```

# DataSource (H2 file-based, recomendado para o laboratório)
spring.datasource.url=jdbc:h2:~/test;AUTO_SERVER=TRUE
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=sa

# H2 console
spring.h2.console.enabled=true
spring.h2.console.path=/console

# JPA / Hibernate (dev only)
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Nota: `spring.jpa.hibernate.ddl-auto=update` facilita o laboratório, mas não é recomendado para produção.


### Tarefa 3: Criando uma Entity do JPA

Agora criaremos a entidade JPA. Nesta aula usaremos o `EntityManager` no service para persistência (focando em JPA puro).

1. Crie `src/main/java/br/com/carstore/entity/CarEntity.java`:

```java
package br.com.carstore.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "car")
public class CarEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String color;

    public CarEntity() {}

    public CarEntity(String name, String color) {
        this.name = name;
        this.color = color;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

}
```

A tabela `car` será mapeada com as colunas `id`, `name` e `color`.


### Tarefa 4: Integrando JPA ao Service

Adapte `CarServiceImpl` para usar `EntityManager` e mapear entre `CarDTO` e `CarEntity`.

Neste laboratório vamos usar uma classe intermediária (um pequeno DAO JPA) que encapsula o `EntityManager` e já é responsável por iniciar/participar de transações. Em seguida, atualizaremos `CarServiceImpl` para delegar para essa classe.

1) Crie um pacote `br.com.carstore.dao` e adicione a classe `CarJpaDao`:

```java
package br.com.carstore.dao;

import br.com.carstore.entity.CarEntity;
import br.com.carstore.model.CarDTO;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import java.util.List;
import java.util.stream.Collectors;

@Repository
@Transactional
public class CarJpaDao {

    @PersistenceContext
    private EntityManager em;

    public List<CarDTO> findAll() {

        List<CarEntity> list = em.createQuery("SELECT c FROM CarEntity c", CarEntity.class).getResultList();

        return list.stream().map(this::toDto).collect(Collectors.toList());

    }

    public void save(CarDTO dto) {

        CarEntity e = new CarEntity();
        e.setName(dto.getName());
        e.setColor(dto.getColor());

        em.persist(e);
        // após o persist, o id gerado está disponível
        dto.setId(e.getId() != null ? String.valueOf(e.getId()) : null);

    }

    public void deleteById(String id) {
        
        Long pk = Long.valueOf(id);
        CarEntity e = em.find(CarEntity.class, pk);

        if (e != null) {
            em.remove(e);
        }

    }

    public void update(String id, CarDTO dto) {

        Long pk = Long.valueOf(id);
        CarEntity e = em.find(CarEntity.class, pk);

        if (e != null) {
            e.setName(dto.getName());
            e.setColor(dto.getColor());
            em.merge(e);
        }

    }

    private CarDTO toDto(CarEntity e) {

        CarDTO d = new CarDTO();
        d.setId(e.getId() != null ? String.valueOf(e.getId()) : null);
        d.setName(e.getName());
        d.setColor(e.getColor());

        return d;

    }

}
```

Notas:
- A classe `CarJpaDao` é anotada com `@Repository` e `@Transactional`. Isso garante que métodos que chamam `em.persist`/`merge` rodem dentro de uma transação.
- Usamos `@PersistenceContext` para injetar o `EntityManager` gerenciado pelo Spring.
- Convertendo entre `CarEntity` e `CarDTO` simplifica a camada de serviço/controle.

2) Atualize `CarServiceImpl` para delegar para `CarJpaDao`.

Substitua a implementação em memória (`ArrayList`) por uma implementação que injeta e usa `CarJpaDao`. Exemplo:

```java
package br.com.carstore.service;

import br.com.carstore.dao.CarJpaDao;
import br.com.carstore.model.CarDTO;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CarServiceImpl implements CarService {

    private final CarJpaDao carJpaDao;

    public CarServiceImpl(CarJpaDao carJpaDao) {
        this.carJpaDao = carJpaDao;
    }

    @Override
    public List<CarDTO> findAll() {
        return carJpaDao.findAll();
    }

    @Override
    public void save(CarDTO carDTO) {
        carJpaDao.save(carDTO);
    }

    @Override
    public void deleteById(String id) {
        carJpaDao.deleteById(id);
    }

    @Override
    public void update(String id, CarDTO carDTO) {
        carJpaDao.update(id, carDTO);
    }

}
```

Observações finais:

- Como o `CarJpaDao` já é transacional, não é estritamente necessário anotar `CarServiceImpl` com `@Transactional`. No entanto, se o serviço for coordenar várias operações (ex.: atualizar várias entidades em uma única operação), é recomendável colocar `@Transactional` no service.
- Mantenha o DAO `JdbcTemplate` (Parte 1) e o DAO JPA (Parte 2) lado a lado para comparar abordagens; escolha uma para produção.


Notas:
- Conversões String -> Long são didáticas; prefira tipos numerais nos DTOs/APIs;
- Para mapeamento automático considere MapStruct.


### Tarefa 5: Testando localmente

1. Após adicionar as dependências e configurações JPA, inicie a aplicação.
2. Use a UI ou endpoints para salvar entidades via `CarServiceImpl` (que usa `EntityManager`).
3. Verifique se não ocorrem exceções de transação (ex.: TransactionRequiredException). Se ocorrer, garanta `@Transactional` no service.
4. Habilite `spring.jpa.show-sql=true` para ver os SQLs gerados pelo Hibernate.


## Conclusão e próximos passos

Neste laboratório você adicionou H2, implementou um DAO com `JdbcTemplate` e criou uma camada JPA usando `EntityManager`. Próximos tópicos recomendados:

- Transações explicadas (`@Transactional`);
- Migrações de schema com Flyway;
- Validação (`@Valid`) e DTOs de entrada/saída;
- Testes com `@DataJpaTest` e Testcontainers.

---

Revise as alterações e execute a aplicação para garantir que o console H2 esteja acessível e que operações CRUD funcionem via JPA.

Parabéns! :+1: