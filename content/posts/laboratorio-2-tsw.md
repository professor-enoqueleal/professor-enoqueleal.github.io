+++
title = "LAB 2 - Criando sua Primeira API com Spring Boot"
description = "Este laboratório apresenta os conceitos básicos para criar uma aplicação Web utilizando Java."
date = "2025-08-21"
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Visão geral e objetivos do laboratório

Criar um projeto Spring Boot utilizando o Spring Initializr.

Configurar as dependências necessárias para uma aplicação web (spring-boot-starter-web) e de monitoramento (spring-boot-starter-actuator).

Implementar um Controller para criar um endpoint REST que retorne uma mensagem simples.

Executar e testar a aplicação no navegador ou com uma ferramenta como o Postman.

## Tarefa 1: Criando o Projeto com o Spring Initializr
O Spring Initializr é a maneira mais fácil e recomendada de iniciar um novo projeto Spring Boot. Ele gera automaticamente toda a estrutura e arquivos de configuração básicos.

Acesse o site Spring Initializr: https://start.spring.io/.

Preencha as opções conforme a tabela abaixo:

| Campo        | Valor                           | Observação                                       |
| :----------- | :-----------------------------: | -----------------------------------------------: |
| Project      | Maven                           | Padrão para Java e mais utilizado                |
| Language     | Java                            | A linguagem da nossa disciplina.                 |
| Spring Boot  | A versão mais recente e estável | Escolha a que não tem sufixo como SNAPSHOT ou M1.|
| Group        | br.com.carstore                 | Identificador da sua organização ou universidade.|
| Artifact     | carstore                        | Nome do seu projeto.                             | 
| Name         | carstore                        | Nome amigável da aplicação.                      | 
| Package name | br.com.carstore.carstore        | Gerado automaticamente.                          | 
| Packaging    | Jar                             | Padrão para Spring Boot com servidor embarcado.  | 
| Java         | 17                              | Versão do JDK que será utilizada no projeto.     |

## Tarefa 2: Adicionando as Dependências (Starters)

As dependências (também chamadas de Starters no Spring) são bibliotecas que adicionam funcionalidades ao seu projeto.

No painel direito, clique em "Add Dependencies".

Adicione as seguintes dependências (digite o nome no campo de busca):

**Spring Web**: Esse é o starter fundamental para construir aplicações web e APIs REST. Ele já inclui bibliotecas como o Tomcat embarcado e o Spring MVC.

**Spring Boot Actuator**: Este starter adiciona recursos de monitoramento e gerenciamento, permitindo que você inspecione o estado da aplicação em tempo de execução.

Após selecionar as dependências, clique no botão "Generate". Um arquivo .zip será baixado para o seu computador.

## Tarefa 3: Importando o Projeto na IDE e Criando o Controller

Descompacte o arquivo .zip em uma pasta de sua escolha.

Abra sua IDE (IntelliJ IDEA, Eclipse, VS Code) e importe o projeto como um projeto Maven existente.

Navegue até o pacote principal (br.com.carstore). Dentro desse pacote, crie um novo pacote chamado controller 

Dentro desse novo pacote, crie uma nova classe Java chamada HelloController.java.

Copiando o código do Controller:

Este Controller terá um endpoint que responde à URL base (/).

Ele usa a anotação **@RestController** para indicar que a classe é um controlador REST e a anotação **@GetMapping** para mapear a requisição HTTP GET para o método hello().

O método retornará uma String simples, que será enviada como a resposta HTTP.

```java
package br.com.carstore.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello, world!";
    }

}
```

## Tarefa 4: Executando e Testando a Aplicação

Execute a classe principal **Application.java** dentro da sua IDE. O servidor Tomcat embarcado será iniciado. Você deve ver logs no console indicando que a aplicação está rodando na porta 8080.

Abra seu navegador ou uma ferramenta de teste de API (como o Postman ou Insomnia).

Acesse a URL: http://localhost:8080/.

Você deve ver a mensagem: "Hello, world!"

Para testar o Actuator, acesse a URL: http://localhost:8080/actuator. Você verá um JSON com links para os endpoints de monitoramento da sua aplicação.

---
Parabéns! :+1:

Pronto! Agora você tem seu primeiro projeto Spring Boot funcional.
