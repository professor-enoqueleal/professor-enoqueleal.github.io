+++
title = "LAB 5 – Melhorando a interface com Bootstrap (WebJars)"
description = "Este laboratório apresenta o passo a passo para adicionar o Bootstrap ao projeto CarStore com Spring Boot e Thymeleaf, melhorando a interface da aplicação."
date = "2025-09-25"
draft = false
author = "Enoque Leal"
tags = [ "java", "html", "web", "spring", "thymeleaf", "bootstrap" ]
+++

## 🎯 Visão geral e objetivos do laboratório

Neste laboratório, vamos adicionar o **Bootstrap** ao projeto **CarStore**, utilizando o **WebJars**.  

O objetivo é melhorar a **interface visual** da aplicação, deixando as telas mais bonitas e responsivas sem precisar escrever todo o CSS manualmente.  


## Pré-requisitos
* Java 17 ou superior
* Maven
* Uma IDE (IntelliJ, VS Code, Eclipse, etc.)
* Projeto do Laboratório 4 concluído

---

## 📝 O que é o WebJar?

Um WebJar é uma forma de empacotar bibliotecas web (como Bootstrap, jQuery ou FontAwesome) em arquivos JAR, que podem ser gerenciados pelo Maven/Gradle.  
Isso facilita a importação e o versionamento das dependências, evitando que precisemos baixar manualmente os arquivos de CSS e JavaScript.  

Nesse laboratório, nós iremos adicionar essas bibliotecas ao nosso projeto, para melhorar o aspecto visual da nossa aplicação **Car Store**.

---

## Tarefa 1: Adicionar o WebJar do Bootstrap

No arquivo **`pom.xml`**, adicione a seguinte dependência:

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>5.3.3</version>
</dependency>
```

Também vamos precisar do jQuery, que é uma dependência do Bootstrap:

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.7.1</version>
</dependency>
```

Salve o `pom.xml` e execute o comando Maven → Reload Project para baixar as dependências.

## Tarefa 2: Referenciar os arquivos no HTML

No Spring Boot, os arquivos do WebJar ficam disponíveis no seguinte caminho:

`/webjars/{biblioteca}/{versão}/{arquivo}`

Por exemplo:

* Bootstrap CSS → `/webjars/bootstrap/5.3.3/css/bootstrap.min.css`

* Bootstrap JS → `/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js`

* jQuery → `/webjars/jquery/3.7.1/jquery.min.js`

👉 Quando usamos Thymeleaf, devemos usar `th:href` e `th:src` para fazer os importes em nossas páginas `html` e garantir que os caminhos funcionem mesmo em ambientes diferentes.

Agora, abra o arquivo `index.html`, localizado no diretório: `src/main/resources/templates/index.html` e adicione o import do `bootstrap.min.css`, do javascript do `jquery.min.js` e também do `bootstrap.bundle.min.js` conforme exemplo abaixo.

```html
<!DOCTYPE html>
<html lang="pt-br" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
    <link rel="stylesheet" th:href="@{/webjars/bootstrap/5.3.3/css/bootstrap.min.css}">
</head>
<body>

<form action="#" th:action="@{/cars}" th:object="${carDTO}" method="post">
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

<script th:src="@{/webjars/jquery/3.7.1/jquery.min.js}"></script>
<script th:src="@{/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js}"></script>
</body>
</html>
```

## Tarefa 2.1: Ajustando o formulário de Cadastro

Agora que já temos os imports no nosso formulário de cadastro, vamos fazer uma pequena refatoração, para deixar o estilo da nossa página de cadastro mais agradável utilizando o bootstrap.

O primeiro passo consiste em envolver o formulário de cadastro já existente em uma `div`. Nessa `div`, nós iremos referenciar a classe `container` do `bootstrap`. 

A classe `container` do Bootstrap serve para centralizar e limitar a largura do conteúdo da sua página de forma responsiva.

Ela é responsável por: 

- Criar um "bloco central" na página, com margens automáticas à esquerda e à direita.

- Definir uma largura máxima que varia de acordo com o tamanho da tela (mobile, tablet, desktop, etc.).

- Adicionar padding horizontal (espaçamento interno), evitando que o conteúdo "encoste" nas bordas da tela.

```html
<div class="container mt-5">

    <!-- 
     Aqui o conteúdo do formulário já existente ...
    -->

</div>
```

Agora, nas `divs` que envolvem os campos `label` e `input`, vamos referenciar a classe `mb-3` do bootstrap. Essa classe aplica uma margem inferior (margin-bottom) de acordo com a escala de espaçamentos do Bootstrap. Ela vai fazer com que os campos não fiquem esteticamente colados uns nos outros.

```html
<div class="mb-3">

    <!-- 
     Aqui o conteúdo dos campos label e input 
    -->

</div>
```
A última parte consiste em referenciar a classe `form-label` nos campos do tipo label e a classe `form-control` nos campos do tipo input.

- form-label: estiliza os rótulos (label) de formulários, ajustando tipografia, alinhamento e espaçamento para manter consistência visual.

- form-control: estiliza os campos de entrada (input, select, textarea), fazendo-os ocupar toda a largura do container, aplicando padding, bordas arredondadas e foco padronizado.

Após todas as alterações, a página index.html deve ficar da seguinte maneira:

```html
<!DOCTYPE html>
<html lang="pt-br" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
    <link rel="stylesheet" th:href="@{/webjars/bootstrap/5.3.3/css/bootstrap.min.css}">
</head>
<body>

<div class="container">

    <form action="#" th:action="@{/cars}" th:object="${carDTO}" method="post">
        <div class="mb-3">
            <label for="name" class="form-label">Nome:</label>
            <input type="text" id="name" th:field="*{name}" class="form-control">
            <span class="error" th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
        </div>

        <div class="mb-3">
            <label for="color" class="form-label">Cor:</label>
            <input type="text" id="color" th:field="*{color}" class="form-control">
            <span class="error" th:if="${#fields.hasErrors('color')}" th:errors="*{color}"></span>
        </div>
        <div>
            <button type="submit" class="btn btn-primary">Salvar</button>
        </div>
    </form>

</div>

<script th:src="@{/webjars/jquery/3.7.1/jquery.min.js}"></script>
<script th:src="@{/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js}"></script>
</body>
</html>
```

Agora, se rodarmos a aplicação (mvn spring-boot:run) e acessarmos http://localhost:8080/, veremos a página já estilizada com Bootstrap.

## Tarefa 3: Melhorando nossa tela de listagem

Agora vamos melhorar o aspecto visual da nossa página de listagem `dashboard.html`. Para isso, abra o arquivo `dashboard.html`, localizado no diretório: `src/main/resources/templates/dashboard.html` e adicione o import do `bootstrap.min.css`, do javascript do `jquery.min.js` e também do `bootstrap.bundle.min.js`, assim como foi feito na página index.html na tarefa 2.

Agora podemos aplicar as classes do Bootstrap para deixar a tabela mais bonita.

Assim como nós fizemos na página index.html, o primeiro passo consiste em envolver a nossa tabela em uma `div`. Nessa div, nós iremos referenciar a classe `container` do Bootstrap.

```html
<div class="container mt-5">

    <!-- 
     Aqui o conteúdo da tabela já existente ...
    -->

</div>
```

Na sequência, na nossa `table` nós iremos referenciar as classes `table` e `table-striped`.

A classe `table` aplica a formatação base de tabelas no Bootstrap, como largura total (width: 100%), espaçamento adequado, bordas horizontais leves e tipografia consistente.

Já a classe `table-striped` adiciona listras (faixas) de cor alternada nas linhas da tabela para melhorar a legibilidade dos dados.

```html

<table class="table table-striped">

    <!-- 
     Aqui o conteúdo da tabela já existente ...
    -->

</table>
```

Após todas as alterações, a página `dashboard.html` deve ficar da seguinte maneira:

```html
<!DOCTYPE html>
<html lang="pt-br" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
    <link rel="stylesheet" th:href="@{/webjars/bootstrap/5.3.3/css/bootstrap.min.css}">
</head>
<body>

    <div class="container mt-5">

      <h1 class="mb-4">Dashboard</h1>

      <table class="table table-striped">
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
              <td><!-- Ações futuras --></td>
          </tr>
          </tbody>
      </table>

    </div>

  <script th:src="@{/webjars/jquery/3.7.1/jquery.min.js}"></script>
  <script th:src="@{/webjars/bootstrap/5.3.3/js/bootstrap.bundle.min.js}"></script>

</body>
</html>
```

---

## Tarefa 4: Criando um menu com Bootstrap e reutilizando com Thymeleaf

Nesta tarefa, vamos criar um **menu de navegação (navbar)** usando Bootstrap e reaproveitá-lo em várias páginas do projeto **Car Store** utilizando o conceito de **fragmentos do Thymeleaf**.  

A utilização de `fragments` do Thymeleaf evita repetição de código e mantém a consistência visual da aplicação.

### 4.1: Criar o fragmento do menu

1 - Dentro da pasta `src/main/resources/templates/`, crie um arquivo chamado `fragments.html`.  

2 - Nesse arquivo, vamos definir o fragmento da navbar:

```html
<!DOCTYPE html>
<html lang="pt-br" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
</head>
<body>
<!-- Fragmento da Navbar -->
<div th:fragment="navbar">
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">CarStore</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/}">New Car</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/cars}">List Cars</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</div>
</body>
</html>
```

### 4.2: Reaproveitar o fragmento nas páginas

Para referenciar esse novo fragmento que acabou de ser criado, na página `index.html` e `dashboard.html`, basta adicionar uma nova `div` com a tag thymeleaf `th:replace="fragments :: navbar"`.

```html
<!-- Inserindo a navbar -->
<div th:replace="fragments :: navbar"></div>
```

Dessa maneira, conseguimos importar o menu em todas as páginas, sem ficar repetindo código. Isso facilita a manutenção e evita duplicação de código (boilerplate).

---
## ✅ Exercício (Opcional)

1 - Adicione o WebJar do FontAwesome para incluir ícones na sua aplicação.

Dependência no `pom.xml`:

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>font-awesome</artifactId>
    <version>6.4.2</version>
</dependency>
```

Use um ícone em um botão da sua aplicação, por exemplo:

```html
<button class="btn btn-primary">
    <i class="fa fa-user"></i> Novo Usuário
</button>
```

2 - Transforme os botões simples do seu projeto em botões estilizados (btn `btn-primary`, `btn btn-danger`, etc.).

3 - Aplique as classes `table-striped` e `table-hover` à tabela de listagem que criamos.

Para consultar todas as possibilidades de ícones da FontAwesome, consulte o site oficial: [Clique aqui!](https://fontawesome.com/icons)

---

Parabéns! :+1:

Pronto! Agora sua aplicação Spring Boot não apenas funciona, mas também tem uma interface moderna e responsiva com a ajuda do Bootstrap.

---

## 📝 Resumo do que você aprendeu

Neste laboratório, você aprendeu a:

- Adicionar dependências de bibliotecas web (Bootstrap, jQuery e FontAwesome) ao projeto usando WebJars e Maven.
- Referenciar corretamente arquivos CSS e JS de WebJars em páginas Thymeleaf usando `th:href` e `th:src`.
- Aplicar classes do Bootstrap para melhorar o visual de formulários e tabelas, tornando a interface mais bonita e responsiva.
- Utilizar utilitários do Bootstrap para espaçamento, alinhamento e estilização de botões.
- Criar e reutilizar fragmentos de interface (navbar) com Thymeleaf, evitando repetição de código e facilitando a manutenção.
- Explorar o uso de ícones com FontAwesome para enriquecer a experiência visual da aplicação.

Essas práticas são fundamentais para criar aplicações web modernas, organizadas e com boa experiência de usuário.
