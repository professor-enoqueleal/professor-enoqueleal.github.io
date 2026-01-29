+++
title = "LAB 7 - Integração com banco de dados via JDBC - PARTE 1"
description = "Guia passo a passo para integrar uma aplicação Spring Boot com um banco relacional usando JDBC (JdbcTemplate) e JPA (Jakarta Persistence / Hibernate)."
date = 2025-10-09
draft = true
author = "Enoque Leal"
tags = [ "java", "spring", "jpa", "jdbc", "h2" ]
+++

## Visão geral e objetivos do laboratório

Neste laboratório vamos adicionar e configurar a persistência na nossa aplicação spring-boot   (`carstore-spring-boot`). O objetivo é apresentar duas abordagens comuns para trabalhar com bancos relacionais em aplicações Spring:

 - Uso direto de JDBC com `JdbcTemplate` (menos boilerplate que DriverManager);
 - Uso de JPA (Jakarta Persistence / Hibernate) diretamente via `EntityManager` (nesta aula não usaremos repositórios automáticos; focaremos em JPA puro).

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

Esta atividade foi organizada em duas partes independentes. Libere a Parte 1 primeiro (JDBC) e, depois que os alunos concluírem, libere a Parte 2 (JPA).

Parte 1 — JDBC (JdbcTemplate)
1. Adicionar dependências necessárias para JDBC e H2 no `pom.xml`.
2. Configurar `application.properties` para H2 (DataSource e console).
3. Criar um DAO com `JdbcTemplate` e testar operações básicas.
4. Testar localmente (inserir via formulário, verificar H2 console).

## PARTE 1 — JDBC (JdbcTemplate)

Na parte 1 desse laboratório, iremos implementar a comunicação com o banco de dados utilizando JDBC.

### Tarefa 1: Dependências (JDBC)

1. Abra `dsw-projects/carstore-spring-boot/pom.xml`.

2. Adicione (se necessário) as dependências abaixo dentro de `<dependencies>` para a Parte 1 (JDBC + H2):

```xml
<!-- H2 (runtime para desenvolvimento) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Spring JDBC (JdbcTemplate) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

3. Salve as alterações no arquivo `pom.xml` e deixe a IDE baixar as dependências.


### Tarefa 2: Configuração (H2 / DataSource para JDBC)

1. Abra o arquivo `application.proprerties`que fica no diretório ` (ou crie) ``carstore-spring-boot/src/main/resources`.

2. Adicione as configurações mínimas para o banco de dados H2 DB:

```
# DataSource (H2 file-based, recomendado para o laboratório)
spring.datasource.url=jdbc:h2:~/test;AUTO_SERVER=TRUE
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=sa

# H2 console
spring.h2.console.enabled=true
spring.h2.console.path=/console
```

Quando a aplicação for executada, será possível acessar a console de gerenciamento web do H2 DB, a partir do path: `/console`.


### Tarefa 3: Implementando a conexão com JdbcTemplate

Nesta etapa criaremos um classe DAO (Data Access Object) que irá utilizar o `JdbcTemplate` para operações básicas.

1. No pacote principal `br.com.carstore`, crie um novo pacote chamado `dao`

2. Dentro desse novo pacote, crie uma nova classe Java chamada `CarDao` em `src/main/java/br/com/carstore/dao/CarDao.java`

3. Na pasta `src/main/resources/`, crie um novo arquivo chamado `schema.sql` e adicione o seguinte conteúdo:

```sql
CREATE TABLE IF NOT EXISTS car (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255),
  color VARCHAR(255)
);
```

Esse arquivo contém um código SQL para uma operação DDL. Essa operação vai garantir que uma tabela chamada **CAR** seja criada automáticamente quando a aplicação for inicializada.

**OBS**: Colocar esse arquivo em `resources` faz com que o Spring Boot execute o script no startup (DataSource initializer).

4. Implemente a classe `CarDao` com o `JdbcTemplate` e os métodos solicitados, conforme exeplo abaixo. 

```java
package br.com.carstore.dao;

import br.com.carstore.model.CarDTO;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Repository
public class CarDao {

    private final JdbcTemplate jdbc;

    public CarDao(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    private final RowMapper<CarDTO> rowMapper = new RowMapper<>() {

        @Override
        public CarDTO mapRow(ResultSet rs, int rowNum) throws SQLException {
            CarDTO dto = new CarDTO();
            dto.setName(rs.getString("name"));
            dto.setColor(rs.getString("color"));
            return dto;
        }

    };

    // SELECT * FROM car -> List<CarDTO>
    public List<CarDTO> findAll() {

        String sql = "SELECT name, color FROM car";

        return jdbc.query(sql, rowMapper);

    }

    // INSERT INTO car (name, color)
    public void save(CarDTO carDTO) {

        String sql = "INSERT INTO car (name, color) VALUES (?, ?)";

        jdbc.update(sql, carDTO.getName(), carDTO.getColor());

    }

    // DELETE FROM car WHERE id = ?
    public void deleteById(String id) {

        String sql = "DELETE FROM car WHERE id = ?";

        jdbc.update(sql, Long.valueOf(id));

    }

    // UPDATE car SET name = ?, color = ? WHERE id = ?
    public void update(String id, CarDTO carDTO) {

        String sql = "UPDATE car SET name = ?, color = ? WHERE id = ?";

        jdbc.update(sql, carDTO.getName(), carDTO.getColor(), Long.valueOf(id));

    }

}
```

Notas sobre a implementação

- RowMapper: converte cada linha do ResultSet em `CarDTO`.
- `findAll()` retorna uma lista de DTOs contendo apenas os campos `name` e `color` (ajuste se seu `CarDTO` tiver `id`).
- `save()` usa `jdbc.update(...)` com parâmetros preparados — evita SQL injection.
- `deleteById` e `update` recebem `id` como String (para combinar com camadas superiores) e convertem para Long; prefira APIs que já aceitem Long.
- Se quiser recuperar o id gerado no insert, use `KeyHolder` (exercício avançado).

5. Agora ajuste a classe `CarServiceImpl`, para usar a classe CarDAO para gravar e consultar os dados no banco, conforme exemplo abaixo:

```java
package br.com.carstore.service;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.CarDTO;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CarServiceImpl implements CarService {

    private final CarDao carDao;

    public CarServiceImpl(CarDao carDao) {
        this.carDao = carDao;
    }

    @Override
    public List<CarDTO> findAll() {

        return carDao.findAll();

    }

    @Override
    public void save(CarDTO carDTO) {

        carDao.save(carDTO);

    }

    @Override
    public void deleteById(String id) {

        carDao.deleteById(id);

    }

    @Override
    public void update(String id, CarDTO carDTO) {

        carDao.update(id, carDTO);

    }

}
```

### Tarefa 4: Testando localmente

1. Agora que a implementação da classe CarDAO foi finalizada, rode a aplicação e teste as alterações.

2. Use o formulário web (página inicial) para inserir um veículo — o `CarDao` fará o `INSERT`.

3. Abra o H2 console: http://localhost:8080/console

4. Verifique a tabela `car` e os registros inseridos.

Exemplo rápido de conexão no H2 console:
    - JDBC URL: jdbc:h2:~/test
    - User: sa
    - Password: sa


### Tarefa 5: Implementar o CommandLineRunner (Opcioanl)

Adicione um `CommandLineRunner` (`StartupRunner`) para popular dados automaticamente e validar `CarDao`.

```java
@Component
public class StartupRunner implements CommandLineRunner {

    private final CarDao carDao;

    public StartupRunner(CarDao carDao) { 

        this.carDao = carDao; 

    }

    @Override
    public void run(String... args) throws Exception {

        carDao.create("Fusca", "Azul");
        carDao.create("Civic", "Preto");

        System.out.println(carDao.findAll());
        
    }

}
```

## Conclusão e próximos passos

Neste laboratório você adicionou H2, implementou um DAO com `JdbcTemplate`.

---

Revise as alterações e execute a aplicação para garantir que o console H2 esteja acessível e que operações CRUD funcionem via JDBC.

Parabéns! :+1: