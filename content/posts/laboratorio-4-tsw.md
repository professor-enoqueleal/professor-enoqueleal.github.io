+++
title = "LAB 4 - Controllers e Views com Thymeleaf no Spring Boot"
description = "Este laboratório apresenta o passo a passo para criar controllers e páginas dinâmicas com Thymeleaf em uma aplicação Spring Boot."
date = "2025-09-11"
draft = false
author = "Enoque Leal"
tags = [ "java", "html", "web", "spring", "thymeleaf" ]
+++

## Visão geral e objetivos do laboratório

Neste laboratório, você vai aprender como criar controllers para processar dados e páginas dinâmicas usando Thymeleaf em uma aplicação Spring Boot. O objetivo é dar continuidade ao sistema de cadastro de carros, permitindo cadastrar e listar carros em páginas web.

## Pré-requisitos
* Java 17 ou superior
* Maven
* Uma IDE (IntelliJ, VS Code, Eclipse, etc.)
* Projeto do Laboratório 3 concluído

---

## Tarefa 1: Estrutura do Projeto

Certifique-se de que seu projeto possui as dependências do Spring Boot, Thymeleaf e Bean Validation, conforme os laboratórios anteriores. O diretório de templates deve estar em `src/main/resources/templates`.


## Tarefa 2: Criando a Classe de Serviço (Service)

Antes de implementar a classe de serviço, é uma boa prática criar uma interface para definir os métodos que a classe deve ter. Isso facilita a manutenção e futuras expansões do projeto.

**Exemplo de interface CarService:**

```java
package br.com.carstore.service;

import br.com.carstore.dto.CarDTO;
import java.util.List;

public interface CarService {

    List<CarDTO> findAll();

    void save(CarDTO carDTO);

    void deleteById(String id);

    void update(String id, CarDTO carDTO);

}
```

No padrão do Spring Boot, a camada de serviço é responsável pela lógica de negócio e manipulação dos dados. Vamos criar a classe responsável por gerenciar os carros cadastrados, essa classe vai implementar a interface `CarService` que foi mostrada anteriormente.

Implemente essa interface na classe `CarServiceImpl` conforme o exemplo abaixo:

Vamos implementar cada método da interface separadamente para facilitar o entendimento.

### Exemplo 1: Listar todos os carros

```java
public List<CarDTO> findAll() {

    return this.carList;

}
```
Esse método retorna todos os carros cadastrados.

### Exemplo 2: Salvar um novo carro

```java
public void save(CarDTO carDTO) {

    if (carDTO.getId() == null) {

        carDTO.setId(UUID.randomUUID().toString());

    }

    this.carList.add(carDTO);

}
```

Esse método adiciona um novo carro à lista, gerando um id único se necessário.

### Exemplo 3: Remover um carro pelo id

```java
public void deleteById(String id) {

    this.carList.removeIf(car -> car.getId().equals(id));

}
```

Esse método remove o carro com o id informado.

### Exemplo 4: Atualizar um carro pelo id

```java
public void update(String id, CarDTO carDTO) {

    this.carList.replaceAll(car -> car.getId().equals(id) ? carDTO : car);

}
```
Esse método atualiza os dados do carro com o id informado.

---

### Implementação final da classe CarServiceImpl

Depois de entender cada método, sua classe de serviço ficará assim:

```java
package br.com.carstore.service;

import br.com.carstore.dto.CarDTO;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@Service
public class CarServiceImpl implements CarService {

    private final List<CarDTO> carList;

    public CarServiceImpl() {
        this.carList = new ArrayList<>();
    }

    public List<CarDTO> findAll() {
        return this.carList;
    }

    public void save(CarDTO carDTO) {
        if (carDTO.getId() == null) {
            carDTO.setId(UUID.randomUUID().toString());
        }
        this.carList.add(carDTO);
    }

    public void deleteById(String id) {
        this.carList.removeIf(car -> car.getId().equals(id));
    }

    public void update(String id, CarDTO carDTO) {
        this.carList.replaceAll(car -> car.getId().equals(id) ? carDTO : car);
    }

}
```

**Passos:**

1. Crie o pacote `br.com.carstore.service` se ainda não existir.

2. Crie a interface `CarService` (opcional para projetos simples, mas recomendada para boas práticas).

3. Crie a classe `CarServiceImpl` e anote com `@Service`.

4. Implemente os métodos para salvar, listar, atualizar e remover carros.

**Exemplo de implementação:**

```java
package br.com.carstore.service;

import br.com.carstore.dto.CarDTO;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@Service
public class CarServiceImpl implements CarService {

    private final List<CarDTO> carList;

    public CarServiceImpl() {
        this.carList = new ArrayList<>();
    }

    public List<CarDTO> findAll() {
        return this.carList;
    }

    public void save(CarDTO carDTO) {
        if (carDTO.getId() == null) {
            carDTO.setId(UUID.randomUUID().toString());
        }
        this.carList.add(carDTO);
    }

    public void deleteById(String id) {
        this.carList.removeIf(car -> car.getId().equals(id));
    }

    public void update(String id, CarDTO carDTO) {
        this.carList.replaceAll(car -> car.getId().equals(id) ? carDTO : car);
    }
}
```

**Explicação dos métodos:**

`findAll()`: retorna todos os carros cadastrados.

`save(CarDTO carDTO)`: adiciona um novo carro à lista, gerando um id único se necessário.

`deleteById(String id)`: remove o carro com o id informado.

`update(String id, CarDTO carDTO)`: atualiza os dados do carro com o id informado.

---


## Tarefa 2: Implementando o Controller

Vamos criar o controller responsável por exibir o formulário, processar o cadastro e listar os carros.

**Passos:**

1. Crie a classe `HomeController` no pacote `br.com.carstore.controller`.

2. Anote a classe com `@Controller`.

3. Injete o serviço responsável por salvar e buscar carros (`CarServiceImpl`). Para isso, basta adicionar o parâmetro no construtor do controller:

```java
private final CarServiceImpl service;

public HomeController(CarServiceImpl service) {
    this.service = service;
}
```

O Spring Boot irá criar e injetar automaticamente a instância do serviço na sua controller.

**Exemplo:**

```java
// src/main/java/br/com/carstore/controller/HomeController.java
package br.com.carstore.controller;

import br.com.carstore.dto.CarDTO;
import br.com.carstore.service.CarServiceImpl;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.List;

@Controller
public class HomeController {

    private final CarServiceImpl service;

    public HomeController(CarServiceImpl service) {
        this.service = service;
    }

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("carDTO", new CarDTO());
        return "index";
    }

    @PostMapping("/cars")
    public String createCar(CarDTO carDTO, BindingResult result) {
        service.save(carDTO);
        return "redirect:/cars";
    }

    @GetMapping("/cars")
    public String getCars(Model model) {
        List<CarDTO> allCars = service.findAll();
        model.addAttribute("cars", allCars);
        return "dashboard";
    }
}
```


## Tarefa 3: Criando as Views com Thymeleaf

### 3.1. Verificando o Formulário de Cadastro (`index.html`)

A página `index.html` já existe no projeto. Verifique se ela está localizada em `src/main/resources/templates/index.html` e se contém o formulário para cadastrar um novo carro, utilizando as tags do Thymeleaf (`th:action`, `th:object`, `th:field`).

Se necessário, ajuste os campos para garantir que o formulário está correto e utiliza o objeto `carDTO`.

### 3.2. Criando a Página de Listagem (`dashboard.html`)

Agora, crie a página `dashboard.html` para exibir a lista de carros cadastrados.

Local: `src/main/resources/templates/dashboard.html`

```html
<!DOCTYPE html>
<html lang="pt-br" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Carros Cadastrados</title>
</head>
<body>
    <h1>Dashboard de Carros</h1>
    <table border="1">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Cor</th>
                <th>Ações</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="car : ${cars}">
                <td th:text="${car.name}"></td>
                <td th:text="${car.color}"></td>
                <td><!-- Ações futuras: editar/remover --></td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```


## Tarefa 4: Testando a Aplicação

1. Execute sua aplicação Spring Boot.

2. Acesse http://localhost:8080/ para cadastrar um novo carro.

3. Após cadastrar, você será redirecionado para http://localhost:8080/cars, onde verá a lista de carros cadastrados.


## Tarefa 5: Próximos Passos

---

## Tarefa 6: Implementando o CRUD REST na API

Além da interface web, é importante que sua aplicação também ofereça uma API REST para manipulação dos dados de carros. Para isso, vamos editar a classe `IndexController` já existente no projeto e implementar os métodos do CRUD para a parte de API REST.

Vamos implementar cada método da API REST separadamente para facilitar o entendimento. Veja os exemplos abaixo:

### Exemplo 1: Listar todos os carros (GET)
```java
@GetMapping("/api/cars")
public ResponseEntity<CarResponseBody> home() {
    List<CarDTO> allCars = carService.findAll();
    CarResponseBody carResponseBody = new CarResponseBody(allCars);
    return ResponseEntity.ok(carResponseBody);
}
```
Esse método retorna todos os carros cadastrados em formato JSON.

### Exemplo 2: Cadastrar um novo carro (POST)
```java
@PostMapping("/api/cars")
public ResponseEntity<CarDTO> createCar(@RequestBody CarDTO car) {
    this.carService.save(car);
    return ResponseEntity.ok().build();
}
```
Esse método cadastra um novo carro via API.

### Exemplo 3: Remover um carro pelo id (DELETE)
```java
@DeleteMapping("/api/cars/{id}")
public ResponseEntity<CarDTO> deleteCar(@PathVariable String id) {
    this.carService.deleteById(id);
    return ResponseEntity.noContent().build();
}
```
Esse método remove o carro com o id informado.

### Exemplo 4: Atualizar um carro pelo id (PUT)
```java
@PutMapping("/api/cars/{id}")
public ResponseEntity<CarDTO> updateCar(@PathVariable String id, @RequestBody CarDTO carDTO) {
    this.carService.update(id, carDTO);
    return ResponseEntity.ok(carDTO);
}
```
Esse método atualiza os dados do carro com o id informado.

---

### Implementação final da classe IndexController

Depois de entender cada método, sua classe REST controller ficará assim:

```java
package br.com.carstore.controller;

import br.com.carstore.dto.CarDTO;
import br.com.carstore.dto.CarResponseBody;
import br.com.carstore.service.CarService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class IndexController {

    private final CarService carService;

    public IndexController(CarService service) {
        this.carService = service;
    }

    @GetMapping("/api/cars")
    public ResponseEntity<CarResponseBody> home() {
        List<CarDTO> allCars = carService.findAll();
        CarResponseBody carResponseBody = new CarResponseBody(allCars);
        return ResponseEntity.ok(carResponseBody);
    }

    @PostMapping("/api/cars")
    public ResponseEntity<CarDTO> createCar(@RequestBody CarDTO car) {
        this.carService.save(car);
        return ResponseEntity.ok().build();
    }

    @DeleteMapping("/api/cars/{id}")
    public ResponseEntity<CarDTO> deleteCar(@PathVariable String id) {
        this.carService.deleteById(id);
        return ResponseEntity.noContent().build();
    }

    @PutMapping("/api/cars/{id}")
    public ResponseEntity<CarDTO> updateCar(@PathVariable String id, @RequestBody CarDTO carDTO) {
        this.carService.update(id, carDTO);
        return ResponseEntity.ok(carDTO);
    }
}
```
Adicionar ações de editar e remover no dashboard.
Implementar validação e tratamento de erros nos formulários.

---

Parabéns! :+1:


Neste laboratório, você aprendeu a:

- Criar e verificar páginas web dinâmicas com Thymeleaf, incluindo formulários e listagem de dados.
- Implementar controllers para processar requisições web e conectar a interface com a lógica de negócio.
- Definir uma interface de serviço e implementar a classe de serviço responsável pelo CRUD de carros.
- Integrar o service nas controllers, separando responsabilidades e facilitando a manutenção do código.
- Criar uma API REST completa, editando uma RestController para expor os métodos do CRUD de carros via endpoints HTTP.

Com isso, seu projeto Spring Boot está pronto para atender tanto usuários web quanto integrações via API, seguindo boas práticas de organização e desenvolvimento!