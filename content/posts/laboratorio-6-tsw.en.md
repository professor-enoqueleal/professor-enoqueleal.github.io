+++
title = "LAB 6 - Creating and configuring the database connection"
description = "This lab introduces the basic concepts on how to create and configure a database connection using JDBC!"
date = 2026-04-22
draft = false
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

Este laboratório apresenta os conceitos básicos para criar uma aplicação Java Web contendo uma camada de persitêcia de dados!

After completing this lab, you should be able to:

- Provisionar uma camada de persistência para a aplicação Java Web;
- Subir um servidor Tomcat (Servlet Container) e um banco de dados em memória (H2 DB) embed para executar sua aplicação Java e persistir seus dados;
- Fazer requisições http através de um formulário HTML e capturar os dados dessa requisição em uma Servlet;
- Gravar os dados que foram capturados de um formulário HTML e persisti-los em um banco de dados (insert).
- Consultar no banco de dados os dados que foram gravados a partir de um formulário HTML (select).
- Alterar no banco de dados os dados que foram gravados a partir de um formulário HTML (update).
- Deletar no banco de dados os dados que foram gravados a partir de um formulário HTML (delete).

# LAB 6 - PARTE 1

## This lab is also available as a video lesson on YouTube

### PART 1
{{< youtube Q9G_xCI7fq0 >}}

### PART 2
{{< youtube Du4fwItGElU >}}

## Atividade 1

## Task 1: Adicionar uma nova dependência ao seu projeto

Now that you already have your application properly created, your web server is running, it's time to adicionar uma camada de persistência de dados na sua aplicação. 

1 - Open the file arquivo **pom.xml**

2 - Localize o bloco **dependencies**. Você deverá adicionar uma nova dependência dentro deste bloco.

**OBS**: Nenhuma dependência deve ser removida nesse processo. Adicione a dependência a seguir dentro do bloco **dependencies**: 

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.1.214</version>
</dependency>
```

3) The resulting code should be equal to the following:

```xml
<dependencies>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>3.8.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>2.1.214</version>
    </dependency>

</dependencies>

```

4 - Review everything that has been done so far!

Congratulations! :+1:

Você adicionou a dependência do H2 DB (Banco de dados em memória).


## Task 2: Registrando o listener do H2 DB no arquivo web.xml

:warning: To learn more about o H2 DB, visit the official documentation through this link: [H2 Database Engine](https://www.h2database.com)

1 - Agora que a dependência do H2 foi adicionada ao projeto, será necessário registrar o *listener* do H2 no arquivo web.xml. To do this navegue até o arquivo web.xml. Este arquivo fica localizado no diretório: car-store/src/main/webapp/WEB-INF/web.xml

2 - Com o arquivo web.xml aberto, adicione o *listener* dentro do bloco *web-app* as shown in the following code:

```xml
<listener>
    <listener-class>org.h2.server.web.DbStarter</listener-class>
</listener>

<servlet>
    <servlet-name>H2Console</servlet-name>
    <servlet-class>org.h2.server.web.WebServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>H2Console</servlet-name>
    <url-pattern>/console/*</url-pattern>
</servlet-mapping>
```

**OBS**: Nenhuma configuração deverá ser removida nesse processo.

3 - O resultado final should be equal to the following code:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         id="WebApp_ID" version="3.0">

    <display-name>car-store</display-name>

    <listener>
        <listener-class>org.h2.server.web.DbStarter</listener-class>
    </listener>

    <servlet>
        <servlet-name>H2Console</servlet-name>
        <servlet-class>org.h2.server.web.WebServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>H2Console</servlet-name>
        <url-pattern>/console/*</url-pattern>
    </servlet-mapping>

</web-app>
```

4 - Save all changes **(CTRL + S)** and run your application *(tomcat7:run)*. Com o *listener* do **H2 DB** properly registered, é possível acessar a console de gerênciamento através do link: *http://localhost:8080/console*

5 - After accessing the console of the **H2 DB**, log in using the following information:

    * Driver Class: org.h2.Driver
    * JDBC URL: jdbc:h2:~/test
    * User Name: sa
    * Password: sa

6 - After logging in, you can create your primeira tabela. Você deverá criar uma tabela chamada **CAR**. To do this utilize o comando **SQL** a seguir:

```sql
CREATE TABLE CAR(ID INT PRIMARY KEY AUTO_INCREMENT, NAME VARCHAR(255));
```

![gif animado demonstrando como criar_uma_tabela_no_h2_db](/gifs/11-criando-tabela-no-h2.gif)

7 - Review everything that has been done so far!

Congratulations! :+1:

Você adicionou a dependência de um banco de dados em memória (H2 DB) na sua aplicação Java Web. Agora que você adicionou a dependência do H2 DB no arquivo pom.xml da sua aplicação, ao fazer o start *(tomcat7:run)*, quando a aplicação é iniciada temos um banco de dados relacional a nossa disposição e temos também uma console para gerenciamento do mesmo.


## Task 3: Criando a primeira Model e a primeira DAO

Agora que temos tem um banco de dados properly configured, it's time to criar a primeira classe model (Car) e a primeira classe DAO (CarDao).

1 - Agora let's create dois novos pacotes (packages), o primeiro se chama *dao* e o segundo *model*. To do this, navigate to the package principal (br.com.carstore). No seu projeto, navegue até o diretório: car-store-guide/app/src/main/java/br.com.carstore, right-click em cima do pacote principal (br.com.carstore) e escolha a opção *New / Package* e insira o nome do primeiro pacote (dao). Repita a operação para criar o segundo pacote (dao).

No final da criação, o resultado esperado é que tenhamos agora três subpacotes dentro do pacote principal seguindo a hierarquia:

```
car-store/
|..| src/
|  |..| main/
|  |  |..| java/
|  |  |  |..| br.com.carstore
|  |  |  |  |..| dao
|  |  |  |  |..| model
|  |  |  |  |..| servlet
|  |  |..| webapp/
|  |  |  |..| WEB-INF/
|  |  |  |  |..| web.xml
|  |  |  |..| index.html
```

![imagem demonstrando a estrutura do projeto](/images/01-project-structure.png)

2 - Agora com os novos pacotes properly created, let's create a nossa classe model chamada Car. To do this, right-click em cima do pacote model, escolha a opção *New, Java Class*, depois type *Car* e aperte a tecla ENTER. O assistente de criação do IntelliJ irá criar uma nova classe Java chamada **Car**.

3 - Com a classe Car properly created, let's create um atribute para ela do tipo *String* com o modificador de acesso *privado (private)* e com o nome de **name**.

4 - Crie os métodos acessores (getters and setters) para esse atributo que acabamos de criar *(private String name)*.

The resulting code should be equal to the following:

```java
package br.com.carstore.model;

public class Car {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}

```

5 - Now that we have a classe Car properly created, podemos criar nossa classe CarDao. To do this, right-click em cima do pacote dao, escolha a opção *New, Java Class*, depois type *CarDao* e aperte a tecla ENTER. O assistente de criação do IntelliJ irá criar uma nova classe Java chamada **CarDao**.

6 - Com a classe **CarDao** criada, let's implement um método chamado **createCar** que retorna **void** (não retorna nada) e que recebe por parâmetro um objeto do tipo Car.

Nessa etapa, the resulting code should be equal to the following code:

```java
package br.com.carstore.dao;

import br.com.carstore.model.Car;

public class CarDao {

    public void createCar(Car car) {

    }

}

```

7 - Agora com o método *createCar* properly created, let's implement a lógica de persistência dos valores do objeto car no nosso banco de dados. Nessa etapa, the resulting code should be equal to the following code:

```java
package br.com.carstore.dao;

import br.com.carstore.model.Car;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

public class CarDao {

    public void createCar(Car car) {

        String SQL = "INSERT INTO CAR (NAME) VALUES (?)";

        try {

            Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa","sa");

            System.out.println("success in database connection");

            PreparedStatement preparedStatement = connection.prepareStatement(SQL);

            preparedStatement.setString(1, car.getName());
            preparedStatement.execute();

            System.out.println("success in insert car");

            connection.close();

        } catch (Exception e) {

            System.out.println("fail in database connection");

        }

    }

}
```

8 - Atenção em toda a implementação e nos imports. Nessa etapa nenhum erro deve ser sinalizado pela sua IDE. Caso seu IDE sinalize algum erro, reveja todos os passos anteriores para garantir que esteja tudo correto.

9 - Com toda a implementação devidamente feita, já é possível testar o projeto para garantir que tudo funciona corretamente.

![gif animado demonstrando o projeto funcionando e gravando dados no h2 db](/gifs/12-mostrando-o-projeto-funcionando.gif)

10 - Review everything that has been done so far!

---

# LAB 6 - PARTE 2

## This lab is also available as a video lesson on YouTube

{{< youtube r9LFSecJlFE >}}

## Task 1: Creating the method findAllCar()

Now that you already have your application properly created, your web server is running. Já consegue gravar os dados no banco de dados, it's time to implementar uma camada de consulta dos dados persistidos no banco de dados da sua aplicação.

1 - To do this, let's refactor a classe model **Car**, removendo o método setter e criando um construtor sobrecarregado. Open the class Car, apague o método getName() e criei um construtor que recebe name como parâmetro.

The resulting code should be equal to the following:


```java
package br.com.carstore.model;

public class Car {

    private String name;

    public Car(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

}
```

2 - Devido a essa refatoração na classe **Car**, é necessário fazer um ajuste na classe **CreateCarServlet.java**, removendo a linha **car.setCarName(carName)** e passando o parâmetro carName via construtor **Car car = new Car(carName)**.

Após a refotoração, o código da classe **CreateCarServlet.java** should be equal to the following code:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.Car;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/create-car")
public class CreateCarServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        String carName = req.getParameter("car-name");

        Car car = new Car(carName);

        new CarDao().createCar(car);

        req.getRequestDispatcher("index.html").forward(req, resp);

    }

}
```

3 - Agora let's create a new method called **findAllCars** dentro da classe CarDao. To do this, open the class CarDao.

Create the method **findAllCars** que devolve uma lista do tipo **Car**. The resulting code should be equal to the following:

```java
public List<Car> findAllCars() {

}
```
Nesse momento o seu IDE vai esta alertando erro porque o retorno (return) ainda não foi implementado. Don't worry porque isso será resolvido no passo a seguir.

4 - Agora let's implement a lógica que faz a busca dos dados no banco de dados. The first step is criar nossa string SQL contento o comando SQL para a busca (SELECT * FROM).

The resulting code should be equal to the following:

```java
public List<Car> findAllCars() {

     String SQL = "SELECT * FROM CAR";

}
```

This command fará uma busca no nosso banco de dados, na tabela CAR e retornará todos os registros existentes nessa tabela.

5 - O restante da implementação será bem similar a implementação do método **createCar()**. Teremos o bloco *try / catch* e também as mensagens de feedback para que possamos saber se a operação foi bem sucedida ou não.

The resulting code should be equal to the following:

```java
 public List<Car> findAllCars() {

        String SQL = "SELECT * FROM CAR";

        try {

            Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa", "sa");

            System.out.println("success in database connection");

            PreparedStatement preparedStatement = connection.prepareStatement(SQL);

            ResultSet resultSet = preparedStatement.executeQuery();

            List<Car> cars = new ArrayList<>();

            while (resultSet.next()) {

                String carName = resultSet.getString("name");

                Car car = new Car(carName);

                cars.add(car);

            }

            System.out.println("success in select * car");

            connection.close();

            return cars;

        } catch (Exception e) {

            System.out.println("fail in database connection");

            return Collections.emptyList();

        }

    }
```

6 - Agora que já implementamos o método findAllCar que retorna uma lista com todos os carros que foram cadastrados na nossa tabela Car, podemos seguir com a criação da nossa Servlet responsável por receber as requisições de consulta.

No pacote servlet, select the option *New* e depois a opção *Java Class*. No assistente de criação, type o nome da classe: **ListCarServlet**.

A estrutura dessa servlet será semelhante a estrutura da servlet **CreateCarServlet**, porém o endpoint cadastrado na anotação **@WebServlet** deve ser **find-all-cars** e o método a ser sobrescrito (Override) é o método **doGet()**.

The resulting code should be equal to the following:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.Car;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/find-all-cars")
public class ListCarServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        super.doGet(req, resp);

    }

}
```

7 - Agora com a classe **ListCarServlet** criada e o método doGet sobrescrito, podemos implementar a lógica que chamará a classe CarDao e o médoto **findAllCars()**.

The resulting code should be equal to the following:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.Car;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/find-all-cars")
public class ListCarServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        List<Car> cars = new CarDao().findAllCars();

        req.setAttribute("cars", cars);

        req.getRequestDispatcher("dashboard.jsp").forward(req, resp);

    }

}
```

Perceba que agora estamos redirecionando o usuário para uma nóva página chamada dashboard.jsp. Essa página ainda não existe e vamos cria-la no passo a seguir.

8 - Agora let's create a página para exibir o resultado da nossa consulta em uma tabela (table) na nossa página HTML.

Crie uma nova página chamada **dashboard.jsp** dentro da pasta **webapp**. Dentro dessa página crie uma tabela (table) com duas colunas, uma coluna para a *propriedade* ID e a outra para a propriedade *Name*.

The resulting code should be equal to the following:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
</head>
<body>
  <div>
    <h1>Cars</h1>
    <table>
        <tr>
            <th>ID</th>
            <th>Name</th>
        </tr>
        <tr>
            <td></td>
            <td></td>
        </tr>
    </table>
  </div>
</body>
</html>
```

9 - Agora let's add o import da taglib do **JSTL** (The JavaServer Pages Standard Tag Library). É através dessa tag library que nós poderemos fazer o uso de IF ou FOR dentro da nossa página.

Após adicionar o import da tag library e o laço for, the resulting code should be equal to the following code:

```html
<!DOCTYPE html>
<html>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
</head>
<body>
  <div>
    <h1>Cars</h1>
    <table>
        <tr>
            <th>ID</th>
            <th>Name</th>
        </tr>
        <c:forEach var="car" items="${cars}">
            <tr>
                <td></td>
                <td>${car.name}</td>
            </tr>
        </c:forEach>
    </table>
  </div>
</body>
</html>
```

10 - O ultimo passo é redirecionar o usuário para a nova página a pós a criação (insert) de um novo carro no nosso banco de dados. To do this, na classe **CreateCarServlet**, troque a linha que contém o código:

```java
 req.getRequestDispatcher("index.html").forward(req, resp);
```
Pela linha a seguir:

```java
resp.sendRedirect("/find-all-cars");
```

Feito isso, após a criação de um novo carro, a requisição será redirecionada para a nossa nova servlet **ListCarServlet** que executará o método **doPost()**, fará a consulta no banco de dados e no final irá redirecionar o usuário para a página dashboard.jsp onde os cadastraados no banco de dados serão renderizados no browser.

11 - Review everything that has been done so far!

---

# LAB 6 - PARTE 3

## Lab overview and objectives

## This lab is also available as a video lesson on YouTube

{{< youtube b3LSVup5AMc >}}

## Task 1: Creating the method deleteCarById()

Now that you already have your application properly created, your web server is running e já consegue gravar e consultar os dados no banco de dados, it's time to implementar a operação que consiste em deletar / remover dados que foram persistidos no banco de dados da sua aplicação.

1 - To do this, let's create a new method called **deleteCarById()**. This method deverá ser criado dentro da classe **CarDao**.

NOTE: A classe **CarDao** já existe e possui dois métodos, sendo eles **createCar()** e **findAllCars**. Nesta seção nós vamos apenas criar /adicionar um novo método dentro dessa classe. O nome do método deve ser **deleteCarById()**. Este méotodo receberá uma String como parâmetro e o seu retorno será do tipo *void*.

To do this, open the class **CarDao** e implemente o método **deleteCarById()**. The resulting code should be equal to the following:


```java
public void deleteCarById(String carId) {

}
```

2 - Agora let's implement a lógica que faz a remocão / delete do dado no banco de dados. The first step is criar uma variável do tipo string que chamaremos de SQL. Essa variável recebe como valor uma string contento o comando SQL para fazer o delete (DELETE CAR WHERE ID = ?).

```java
public void deleteCarById(String carId) {

    String SQL = "DELETE CAR WHERE ID = ?";

}
```

NOTE: This command fará a remoção do dado armazenado na nossa tabela de acordo com o ID informado. 


3 - O restante da implementação do método será bem similar a implementação que fizemos anteriormente no método **createCar()**. Teremos o bloco *try / catch* e também as mensagens de feedback para que possamos saber se a operação foi bem sucedida ou não.

The resulting code should be equal to the following:

```java
public void deleteCarById(String carId) {

    String SQL = "DELETE CAR WHERE ID = ?";

    try {

        Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa", "sa");

        System.out.println("success in database connection");

        PreparedStatement preparedStatement = connection.prepareStatement(SQL);
        preparedStatement.setString(1, carId);
        preparedStatement.execute();

        System.out.println("success on delete car with id: " + carId);

        connection.close();

    } catch (Exception e) {

        System.out.println("fail in database connection");

    }

}
```

NOTE: Don't worry com repetição de código nesse momento. Nos próximos laboratórios nós faremos uma refatoração para remover código repetido *(Boilerplate)*.

4 - Agora que já implementamos o método **deleteCarById()** que recebe um ID por parâmetro e executa a lógica para a remoção do registro de um carro em nossa tabela, já podemos seguir com a criação da *Servlet* que ficará responsável por receber as requisições de *delete*.

No pacote br.com.carstore.servlet, select the option *New* e depois a opção *Java Class*. No assistente de criação, type o nome da classe: **DeleteCarServlet**.

A estrutura dessa servlet será semelhante a estrutura da servlet **CreateCarServlet**, porém o endpoint cadastrado na anotação **@WebServlet** deve ser **/delete-car** e o método a ser sobrescrito (Override) deve ser o método **doPost()**.

The resulting code should be equal to the following:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/delete-car")
public class DeleteCarServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        String carId = req.getParameter("id");

        new CarDao().deleteCarById(carId);

        resp.sendRedirect("/find-all-cars");

    }

}
```

Pronto, nossas classes *DAO* e *Servlet* estão prontas para receber a requisição e executar o comando de remoção / delete no nosso banco de dados.

Após executar a chamada para a classe **carDao** e no método **deleteCarById** uma requisição e feita para o endpoint **/find-all-cars** que por sua vez executa uma nova consulta no banco de dados e exibe os dados atualizados na página **dashboard.jsp**.

---

## Task 2: Refatorando o código para retornar o ID

Para que o comando de *remoção / delete* funcione corretamente, nós precisamos que o ID do carro cadastrado seja retornado no momento da consulta. To do this teremos que fazer uma refatoração no nosso código.

1 - The first step is refatorar a classe *model* **Car**, adicionando uma nova variável do tipo *String* chamada *id* e um novo construtor sobrecarregado que recebe o *ID* e o *Name* por parametrô. Open the class Car que esta localizada dentro do pacote *br.com.carstore.model*, crie uma nova variável privada / private do tipo String chamada ID e crie o seu respectivo método *(getter)* getId(). Na sequência, crie um construtor que recebe *id* e *name* como parâmetro.

NOTE: No code should be removed at this stage. Apenas adicione uma nova variável e o construtor sobrecarregado.

The resulting code should be equal to the following:


```java
package br.com.carstore.model;

public class Car {

    private String id;
    private String name;

    public Car(String name) {
        this.name = name;
    }

    public Car(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

}

```

2 - Agora será necessário refatorar o método **findAllCars()** que foi criado no Laboratório 3, para que ao varrer *(while)* o *resultSet*, ele pegue duas propriedades sendo elas *id* e *name*.

O código existente é o seguinte:

```java

while (resultSet.next()) {

    String carName = resultSet.getString("name");

    Car car = new Car(carName);

    cars.add(car);

}

```

Após a refatoração, the resulting code should be equal to the following code:


```java

while (resultSet.next()) {

    String carId = resultSet.getString("id");
    String carName = resultSet.getString("name");

    Car car = new Car(carId, carName);

    cars.add(car);

}

```

3 - Agora será necessário refatorar o formulário html (dashboard.jsp) para exibir o ID no momento da consulta. Abra a página **dashboard.jsp** e adicione a variável *${car.id}*

O código existente é o seguinte:

```html
<c:forEach var="car" items="${cars}">
    <tr>
        <td></td>
        <td>${car.name}</td>
    </tr>
</c:forEach>

```

Após a refatoração, the resulting code should be equal to the following code:

```html
<c:forEach var="car" items="${cars}">
    <tr>
        <td>${car.id}</td>
        <td>${car.name}</td>
    </tr>
</c:forEach>

```

4 - Review everything that has been done so far!

Congratulations! :+1:

5 - Garanta que tudo até aqui esteja funcionando adequadamente. Salve tudo (CTRL + S) e faça um teste em sua aplicação *tomcat7:run*, faça o cadastro de um carro e veja se o ID agora é exibido quando a página *dashboard.jsp* é renderizada as shown in the following image:

![gif animado demonstrando o id do carro sendo exibido na tabela](/gifs/13-exibindo-o-campo-id.gif)

## Task 3: Refatorando a página dashboard.jsp

Agora que o ID do carro é exibido na página, it's time to implementar a lógica para remoção do carro no formulário html *dashboard.jsp*.

1 - To do this, abra novamente a página *dashboard.jsp*. Dentro da tabela (table), adicione mais uma coluna (th). A label será *Actions*.

```html
<table>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Actions</th>
    </tr>
    <c:forEach var="car" items="${cars}">
        <tr>
            <td>${car.id}</td>
            <td>${car.name}</td>
        </tr>
    </c:forEach>
</table>
```

2 - O valor que será preenchido será um formulário (form) que ao ser submetido, fará uma requisição HTTP para a nossa nova servlet **DeleteCarServlet** passando o ID do veículo como parametrô.

The resulting code should be equal to the following:

```html
 <table>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Actions</th>
    </tr>
    <c:forEach var="car" items="${cars}">
        <tr>
            <td>${car.id}</td>
            <td>${car.name}</td>
            <td>
                <form action="/delete-car" method="post">
                    <input type="hidden" id="id" name="id" value="${car.id}">
                    <button type="submit">Delete</button>
                </form>
            </td>
        </tr>
    </c:forEach>
</table>
```

![imagem mostrando o botão delete sendo renderizado no formulário de listagem](/images/02-formulario-com-o-botao.png)

3 - Review everything that has been done so far!

![gif animado demonstrando a remoção funcionando](/gifs/14-delete-funcionando.gif)

---

# LAB 6 - PARTE 4

## Lab overview and objectives

## This lab is also available as a video lesson on YouTube

{{< youtube nZwDype_1p8 >}}

## Task 1: Creating the method updateCar()

Now that you already have your application properly created, your web server is running e já consegue gravar, consultar e deletar dados no banco, it's time to implementar a operação que consiste em atualizar dados que foram persistidos no banco de dados da sua aplicação.

1 - To do this, let's create a new method called **updateCar()** que recebe um objeto do tipo car por parâmetro. This method deverá ser criado dentro da classe **CarDao**.

NOTE: A classe **CarDao** já existe e possui três métodos, sendo eles **createCar()**, **findAllCars** e **deleteCarById()**. Nesta seção nós vamos apenas adicionar um novo método dentro dessa classe. O nome do método deve ser **updateCar()**. Este méotodo receberá um objeto do tipo *Car* como parâmetro e o seu retorno será do tipo *void*.

To do this, open the class **CarDao** e implemente o método **updateCar()**. The resulting code should be equal to the following:


```java
public void updateCar(Car car) {

}
```

2 - Agora let's implement a lógica que faz a atualização do dado na nossa tabela. O primeiro passo consiste em criar uma variável do tipo *string*. Chamaremos essa variável de SQL. Essa variável recebe como valor uma string contento o comando SQL para fazer o delete (UPDATE ... WHERE ID = ?).

The resulting code should be equal to the following:

```java
public void updateCar(Car car) {

    String SQL = "UPDATE CAR SET NAME = ? WHERE ID = ?";

}
```

NOTE: This command fará a atualização do dado armazenado na nossa tabela de acordo com o ID informado. Irá atualizar os dados contigos nos campos informados atravé do comando **SET**. Esse exemplo é extremamente simples. Muita atenção quando você for replicar essa função no seu projeto. 


3 - O restante da implementação do método será bem similar a implementação que fizemos anteriormente no método **createCar()**. Teremos o bloco *try / catch* e também as mensagens de feedback para que possamos saber se a operação foi bem-sucedida ou não.

The resulting code should be equal to the following:

```java
public void updateCar(Car car) {

    String SQL = "UPDATE CAR SET NAME = ? WHERE ID = ?";

    try {

        Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa","sa");

        System.out.println("success in database connection");

        PreparedStatement preparedStatement = connection.prepareStatement(SQL);

        preparedStatement.setString(1, car.getName());
        preparedStatement.setString(2, car.getId());
        preparedStatement.execute();

        System.out.println("success in update car");

        connection.close();

    } catch (Exception e) {

        System.out.println("fail in database connection");
        System.out.println("Error: " + e.getMessage());

    }

}
```

NOTE: Don't worry com repetição de código nesse momento. Nos próximos laboratórios nós faremos uma refatoração para remover código repetido *(Boilerplate)*.

4 - Agora que já implementamos o método **updateCar()** que recebe um objeto do tipo *Car* por parâmetro e executa a lógica para a atualização do registro de um carro em nossa tabela, já podemos seguir com a refatoração da *Servlet* que ficará responsável por receber as requisições de *update*. 

No pacote *br.com.carstore.servlet*, procure pela *Servlet* chamada **CreateCarServlet**.

Essa Servlet já existe e receberá algumas alterações. Agora ao receber uma requisição no método *doPost()*, além de coletar o parâmetro *car-name*, nós vamos coletar também o *ID* e armazená-lo em uma variável do tipo String chamada *carId*.

Após essa implementação, the resulting code should be equal to the following code:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.Car;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/create-car")
public class CreateCarServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        String carId = req.getParameter("id");
        String carName = req.getParameter("car-name");

        Car car = new Car(carName);

        new CarDao().createCar(car);

        resp.sendRedirect("/find-all-cars");

    }

}

```

5 - Agora precisamos validar se o valor da variável *carId* é vazio. Se for vazio (não contém um ID) significa que se trata de uma requisição de cadastro. E se o valor da variável *carId* não for um valor vazio (contém um ID) significa que se trata de uma requisição de atualização. 

Para a construção dessa lógica nós utilizaremos o método *isBlank()* da classe **String** que foi introduzido na versão 11 do Java. 

The resulting code should be equal to the following:

```java
package br.com.carstore.servlet;

import br.com.carstore.dao.CarDao;
import br.com.carstore.model.Car;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/create-car")
public class CreateCarServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        String carId = req.getParameter("id");
        String carName = req.getParameter("car-name");

        CarDao carDao = new CarDao();
        Car car = new Car(carId, carName);

        if (carId.isBlank()) {

            carDao.createCar(car);

        } else {

            carDao.updateCar(car);
        }


        resp.sendRedirect("/find-all-cars");

    }

}

```

NOTE: Repare que se for uma requisição de criação *(insert)*, nós continuaremos chamando o método **createCar()**, caso contrário, nós chamaremos agora o novo método **updateCar()**. O restante da lógica permanece igual.

Pronto, nossas classes *DAO* e *Servlet* estão prontas para receber as requisições e executar o comando de atualização ou criação no nosso banco de dados.

Após executar a chamada para a classe **carDao** e no método **updateCar** uma requisição e feita para o endpoint **/find-all-cars** que por sua vez executa uma nova consulta no banco de dados e exibe os dados atualizados na página **dashboard.jsp**.

## Task 2: Refatorando a página dashboard.jsp

Agora que já construímos a lógica para atualizar os dados de um carro, nós precisamos refatorar a página *dashboard.jsp* e adicionar o *anchor / hyperlink* que irá redirecionar a requisição para a página *index.jsp*, enviados os dados devidamente preenchidos nos campos de input.

1 - To do this, abra a página *dashboard.jsp*. Localize o *form* que criamos no laboratório anterior dentro da tabela *(table)* e adicione um *anchor* dentro da coluna Actions, ao lado do botão (button) de delete, antes do fechamento do form. Adicionaremos também um label para que visualmente fique dividido o *botão* e o *anchor*.

Após a implementação, the resulting code should be equal to the following code:

```html
<table>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Actions</th>
    </tr>
    <c:forEach var="car" items="${cars}">
        <tr>
            <td>${car.id}</td>
            <td>${car.name}</td>
            <td>
                <form action="/delete-car" method="post">
                    <input type="hidden" id="id" name="id" value="${car.id}">
                    <button type="submit">Delete</button>
                    <span> | </span>
                    <a href="index.jsp?id=${car.id}&name=${car.name}">Update</a>
                </form>
            </td>
        </tr>
    </c:forEach>
</table>
```

NOTE: Repare que o *anchor* irá fazer um *hyperlink* com a página *index.jsp*. Essa operação irá enviar alguns parametrôs como a variável *car.id* e *car.name*. Isso é necessário para que possamos acessar esses valores na página *index.jsp* conforme será demonstrado a seguir.

2 - Trocando a extensão da página **index**

The next step is modificar extensão da página index de **.html** para **.jsp**. Isso é necessário para que possamos acessar os valores que foram enviados por parâmetro na requisição feita através de *hyperlink* da página *dashboard.jsp* para a página *index.jsp*. 

In IntelliJ, navigate to the page **index.html** que fica no diretório: car-store-guide/app/src/main/webapp. Clique com o botão direito do mouse em cima do arquivo index.html, select the option *refactor / rename*, troque a extensão de *.html* para *.jsp*.

![gif animado demonstrando como trocar a extensão da página](/gifs/15-trocando-a-extensao.gif)

3 - Refatorando a página *index.jsp*

Agora que a página index contém a extensão *.jsp*, podemos fazer a refatoração necessária na página para que nossa operação de *update / atualização* possa funcionar corretamente.

Com a página *index.jsp* aberta, iremos adicionar um novo campo *input* do tipo *hidden* (escondido) para armazenar o variável ID ${param.id} e adicionar a propriedade value no campo input para armazenar a variável ${param.name}.

Após a refatoração, the resulting code should be equal to the following code:

```html
<html>
<body>
<h2>Create Car</h2>

<form action="/create-car" method="post">

    <label>Car Name</label>
    <input type="text" name="car-name" id="car-name" value="${param.name}">
    <input type="hidden" id="id" name="id" value="${param.id}">

    <button type="submit">Save</button>

</form>

</body>
</html>

```

4 - Review everything that has been done so far!

Save all changes **(CTRL + S)** and run your application *(tomcat7:run)*. Access your application through the link http://localhost:8080 e faça o cadastro de um carro. Repare que agora após a tela de cadastro, na tabela da página *dashbard.jsp* é exibido um *hyperlink* com o a label *Update*. Ao clicar nesse hyperlink você será redirecionado novamente para a *index.jsp*, porém os campos vão estar devidamente preenchidos com os valores exibidos na tabela. Basta alterar o valor e clicar no botão register. O usuário será redirecionando para a *dashboard.jsp* e os valores serão alterados no banco de dados as demonstrated no gif animado a baixo.

![gif animado demonstrando como trocar a extensão da página](/gifs/16-testando-update.gif)

5 - Mudando o texto do botão *(Opcional)*

Por questões de organização, vamos trocar o texto do botão na página *index.jsp* de *Register* para *Save*.

In IntelliJ, navigate to the page *index.jsp* que fica no diretório: car-store-guide/app/src/main/webapp. Open the file e localize o botão **register**. 

Troce a palavra *Register* por *Save*.

---

Congratulations! :+1:

Você adicionou a dependência de um banco de dados em memória (H2 DB) na sua aplicação Java Web, criou a primeira classe model e dao. Implementou a lógica para abrir uma conexão com o bando de dados (H2) e o comando de persistência (insert) dos dados no banco.

Agora ao fazer o start *(tomcat7:run)*, quando a aplicação é iniciada temos um banco de dados relacional a nossa disposição e temos também uma console para gerenciamento do mesmo. Nesse momento a parte do create do nosso CRUD está implementada e funcionando.
