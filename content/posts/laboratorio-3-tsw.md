+++
title = "LAB 3 - Adicionando o Thymeleaf e o Bean validation na aplicação Spring Boot"
description = "Este laboratório apresenta os conceitos do thymeleaf e do Bean Validation"
date = "2025-09-04"
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++
## Visão geral e objetivos do laboratório

Olá, aluno! Neste laboratório, vamos aprofundar nossos conhecimentos em Spring Boot, saindo de uma API REST pura para construir uma interface web simples e funcional. O foco será em como usar o **Thymeleaf** para renderizar páginas HTML dinâmicas e como integrar o **Bean Validation** para garantir a integridade dos dados enviados pelo usuário através de um formulário.

## Pré-requisitos
* Java 17 ou superior ++
* Maven
* Uma IDE (IntelliJ, VS Code, etc.)
* Projeto do Laboratório 2 concluído.

---

### Tarefa 1: Adicionar a Dependência do Thymeleaf

No Spring Boot, as dependências principais são chamadas de Starters. Vamos adicionar as dependências do Thymeleaf e do Bean Validation para que nossa aplicação possa renderizar páginas HTML e validar os dados do formulário.

Abra o arquivo `pom.xml` do seu projeto e, dentro da tag `<dependencies>`, adicione as seguintes linhas:

| Starter                      | Para quê serve?                                      |
|------------------------------|-----------------------------------------------------|
| spring-boot-starter-thymeleaf| Permite criar páginas HTML dinâmicas com Thymeleaf   |
| spring-boot-starter-validation| Adiciona suporte à validação de dados com Bean Validation |

**Exemplo:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

O starter do Bean Validation (`spring-boot-starter-validation`) é essencial para que as anotações de validação funcionem no DTO e nos formulários.

Após adicionar as dependências, clique com o botão direito no projeto e selecione "Reload Project" ou "Update Maven Project" na sua IDE para garantir que tudo seja baixado corretamente.

Após adicionar a dependência, atualize o seu projeto Maven.


## Tarefa 2: Criar a Estrutura de Diretórios

O **Spring Boot** segue uma convenção para a localização dos templates do **Thymeleaf**. Crie o diretório templates dentro da pasta src/main/resources.

```
src/main/resources/templates
```

É nesse diretório que nossos arquivos HTML ficarão.


## Tarefa 3: Criar um DTO com Anotações de Validação

Em vez de usar uma entidade, vamos criar um DTO simples que representará os dados do formulário e conterá as regras de validação. Crie a classe CarDTO com as anotações de Bean Validation.

```
src/main/java/br/com/carstore/dto/CarDTO.java
```

```java
package br.com.carstore.dto;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

public class CarDTO {

    @NotBlank(message = "O nome é obrigatório.")
    @Size(min = 3, max = 50, message = "O nome deve ter entre 3 e 50 caracteres.")
    private String name;

    @NotBlank(message = "A cor é obrigatória.")
    private String color;

    // Getters e Setters
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


## Tarefa 4: Criar o Controller para as Páginas

Agora, crie um novo `Controller` que irá lidar com as requisições de páginas `HTML`, em vez de `JSON`. 

Usaremos a anotação `@Controller` (e não `@RestController)`.

```
src/main/java/br/com/carstore/controller/CarController.java
```

```java
package br.com.carstore.controller;

import br.com.carstore.dto.CarDTO;
import javax.validation.Valid;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class CarController {

    @GetMapping("/index")
    public String exibirFormulario(Model model) {
        model.addAttribute("carDTO", new CarDTO());
        return "index"; // Retorna o nome do template
    }

    @PostMapping("/carros")
    public String salvarCarro(@Valid CarDTO carDTO, BindingResult result) {
        if (result.hasErrors()) {
            return "index"; // Em caso de erro, retorna ao formulário com as mensagens
        }

        // Se a validação for bem-sucedida, você pode processar os dados
        System.out.println("Carro validado com sucesso: " + carDTO.getName() + ", " + carDTO.getColor());

        return "redirect:/sucesso";
    }

    @GetMapping("/sucesso")
    public String sucesso() {
        return "sucesso";
    }

}
```

## Tarefa 5: Criar os Templates HTML

Agora, vamos criar os arquivos HTML com a sintaxe do **Thymeleaf**, atualizando-os para o `CarDTO`.

```
src/main/resources/templates/index.html
```

```html

<!DOCTYPE html>
<html lang="pt-br" xmlns:th="[http://www.thymeleaf.org](http://www.thymeleaf.org)">
<head>
    <meta charset="UTF-8">
    <title>Cadastro de Carro</title>
    <style>
        .error { color: red; font-size: 0.8em; }
    </style>
</head>
<body>
    <h1>Cadastrar Novo Carro</h1>

    <form action="#" th:action="@{/carros}" th:object="${carDTO}" method="post">
        <div>
            <label for="name">Nome:</label>
            <input type="text" id="name" th:field="*{name}">
            <span class="error" th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
        </div>
        <div>
            <label for="color">Cor:</label>
            <input type="text" id="color" th:field="*{color}">
            <span class="error" th:if="${#fields.hasErrors('color')}" th:errors="*{color}"></span>
        </div>
        <div>
            <button type="submit">Salvar</button>
        </div>
    </form>
</body>
</html>

```

```
src/main/resources/templates/sucesso.html
```

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Sucesso!</title>
</head>
<body>
    <h1>Cadastro Realizado com Sucesso!</h1>
    <p>Seu carro foi cadastrado e validado.</p>
</body>
</html>
```


## Tarefa 6: Testar a Aplicação

1. Execute sua aplicação Spring Boot.

2. Abra seu navegador e acesse a URL: http://localhost:8080/index

3. Tente enviar o formulário com dados inválidos (campos vazios). Observe as mensagens de erro na tela.

4. Tente enviar o formulário com dados válidos. Você será redirecionado para a página sucesso.html.

---

Parabéns! :+1:


* Integrar o Thymeleaf para criar interfaces web.

* Utilizar Bean Validation para definir regras de validação nos DTOs.

* Processar formulários, capturar e exibir mensagens de erro de forma amigável para o usuário.

