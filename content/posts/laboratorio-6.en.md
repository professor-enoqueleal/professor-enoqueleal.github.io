+++
title = "LAB 6 - Configuring the connection pool"
description = "This lab aims to present a basic approach on how to configure a connection pool to manage database connections!"
date = 2025-04-25
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

Este laboratório tem como objetivo apresentar uma forma básica sobre como configurar um pool de conexões para gerenciar as conexões com o banco de dados!

A connection pool is a technique used to improve the performance of applications that frequently use database connections.

After completing this lab, you should be able to:

- Implement a connection pool using the library [Apache Commons DBCP](https://github.com/apache/commons-dbcp);

## This lab is also available as a video lesson on YouTube

{{< youtube HcAkUWNJxbQ >}}

## Task 1: Adding the dependency do Apache Commons DBCP

1 - Adding the dependency

In IntelliJ IDEA, open the file de configuração do projeto chamado "pom.xml" (usually located in the project root).

Locate the section <dependencies> e add the following dependency:

```xml

<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-dbcp2</artifactId>
  <version>2.9.0</version>
</dependency>


```
NOTE: No code should be removed at this stage. Just add the new dependency no *pom.xml*.

2 - Save all changes **(CTRL + S)**

NOTE: After saving the changes, o IntelliJ IDEA should automatically synchronize the changes do arquivo de configuração and download the library Apache Commons DBCP.


## Task 2: Creating the Connection Pool class

1 - Create a configuration class for the connection pool

In IntelliJ IDEA, navigate to the package principal *br.com.carstore.servlet*, right-click e selecione *New / Package* type **config** e press the ENTER.

Após ter criado o pacote *config*, right-click no pacote **config** e selecione *New / Java Class*.

Set the class name to "ConnectionPoolConfig" and click "OK".

2 - Open the class que acabamos de criar e implement a static method (static) chamado *getDataSource* that takes no parameters and returns a **BasicDataSource** as shown in the following code:

```java

import org.apache.commons.dbcp2.BasicDataSource;

public class ConnectionPoolConfig {
    
    private static BasicDataSource dataSource;

    public static BasicDataSource getDataSource() {

    }
    
}
```

NOTE: Don't forget to importar (import) a classe BasicDataSource do pacote *org.apache.commons.dbcp2*. To import using IntelliJ, right-click em cima do nome da classe e utilize o atalho **(ALT + ENTER)** e select the option **import class**.

3 - Now that we have nossa classe **BasicDataSource** e nosso método **getDataSource()** properly created, vamos iniciar nossa implementacão. The first part consists of uma conditional validation that checks se a variável *dataSource* é nula (null).

The resulting code should be equal to the following:

```java

package br.com.carstore.config;

import org.apache.commons.dbcp2.BasicDataSource;

public class ConnectionPoolConfig {

    private static BasicDataSource dataSource;
    
    private static BasicDataSource getDataSource() {

        if (dataSource == null) {
            
        }

        return dataSource;

    }
    
}

```

If the result of this conditional validation is true, nós iremos criar um novo *dataSource* (será demonstrado na próxima seção). If the return is false, it means there already exists um dataSource criado e portando nós iremos retornar ele, sem executar nenhuma ação adicional. 

4 - Assumindo que o retorno da validação condicional foi verdadeiro (true), nós precisamos criar um novo dataSource. para isso nós iremos criar uma nova instância de **BasicDataSource** e passar alguns parâmetros sendo eles:

* URL
* Username
* Password
* Min Idle
* Max Idle
* Max total

Notice that some of these parameters we already Utilizávamos nas implementações anteriores como (url, username e password). However, now we are adding three new parameters sendo eles (Min Idle, Max Idle e Max total).

These parameters are necessary for our pool to be configured e eles podem variar de acordo com as características da sua aplicação. 

The following is a brief description of the role of these parameters:

* MinIdle: Minimum number of idle connections in the pool
* MaxIdle: Maximum number of idle connections in the pool
* MaxTotal: Maximum number of total connections in the pool

Finally, let's add uma mensagem de feedback para nossos usuários sinalizando que um novo pool de conexões foi criado com sucesso.

The resulting code should be equal to the following:

```java
package br.com.carstore.config;

import org.apache.commons.dbcp2.BasicDataSource;

import java.sql.Connection;
import java.sql.SQLException;

public class ConnectionPoolConfig {

    private static BasicDataSource dataSource;

    private static BasicDataSource getDataSource() {

        if (dataSource == null) {
            dataSource = new BasicDataSource();
            dataSource.setUrl("jdbc:h2:~/test");
            dataSource.setUsername("sa");
            dataSource.setPassword("sa");
            dataSource.setMinIdle(5);   // Número mínimo de conexões ociosas no pool
            dataSource.setMaxIdle(10);  // Número máximo de conexões ociosas no pool
            dataSource.setMaxTotal(50); // Número máximo de conexões totais no pool

            System.out.println("New connection pool created with successful");

        }

        return dataSource;

    }

}

```

5 - Creating the method getConnection

Now that we have o método getDataSource properly implemented, we need to create the method that returns connections to users.

To do this, let's create um novo método estático (static) chamado getConnection que devolve uma connection.

The resulting code should be equal to the following:

```java
public static Connection getConnection() throws SQLException {

    return getDataSource().getConnection();

}
```

6 - Creating a private constructor

Now that we have o método getDataSource() e getConnection() properly created, we need to create a private constructor that calls the method getDataSource to initialize a new connection pool as soon as our class is called for the first time.

The resulting code should be equal to the following:

```java
private ConnectionPoolConfig() {
    getDataSource();
}
```

With the entire implementation done, the code of the class **ConnectionPoolConfig** should be equal to the following code:

```java
package br.com.carstore.config;

import org.apache.commons.dbcp2.BasicDataSource;

import java.sql.Connection;
import java.sql.SQLException;

public class ConnectionPoolConfig {

    private static BasicDataSource dataSource;

    private ConnectionPoolConfig() {
        getDataSource();
    }

    private static BasicDataSource getDataSource() {

        if (dataSource == null) {
            dataSource = new BasicDataSource();
            dataSource.setUrl("jdbc:h2:~/test");
            dataSource.setUsername("sa");
            dataSource.setPassword("sa");
            dataSource.setMinIdle(5);   // Número mínimo de conexões ociosas no pool
            dataSource.setMaxIdle(10);  // Número máximo de conexões ociosas no pool
            dataSource.setMaxTotal(50); // Número máximo de conexões totais no pool

            System.out.println("New connection pool created with successful");

        }

        return dataSource;

    }

    public static Connection getConnection() throws SQLException {

        return getDataSource().getConnection();

    }

}

```

Save all changes **(CTRL + S)**

## Task 3: Refactoring the DAO class to use the connection pool

1 - Now that we have nosso pool de conexões properly created e configurado, it's time to refatorar a classe **CarDAO** so that it uses connections provided through our connection pool.

To do this, no IntelliJ IDEA, navigate to the package *br.com.carstore.dao* and with a quick double-click open the **CarDAO**.

With the CarDAO class open we can start our refactoring.

2 - The CarDAO class has several methods, namely:

* createCar()
* findAllCars()
* deleteCarById()
* updateCar()

All of them open a new connection to the database, which is not a good practice.

NOTE: This was done intentionally.

Now with the implementation of our connection pool, no DAO method will create connections anymore, instead, it will request a connection from our pool que por sua vez irá fazer o reaproveitamento de conexões existesntes que estiverem disponíveis e caso não existe, irá criar uma nova conexão.

This way we will save computational resources and as a consequence our application will be more performant.

Inside each method mentioned above, locate the line that opens the connection and make the replacement:

REMOVE:
  
```java
Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa","sa");
  
System.out.println("success in database connection");
```
  
ADD: 

```java
Connection connection = ConnectionPoolConfig.getConnection();
```

Now, the methods will no longer open connections directly e portando não escreverá a mensagem ("success in database connection") porque a abertura de novas conexões agora é responsabilidade da nossa classe **BasicDataSource**.

Here we are implementing the **S** do [SOLID](https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english/).

3 - Review everything that has been done so far!

4 - Save all changes **(CTRL + S)** and run your application *(tomcat7:run)*. 

Access your application through the link http://localhost:8080 e faça o cadastro de um carro. Repare que o comportamento da aplicação não mudou, porém a mensagem ("New connection pool created with successful") só é escrita uma vez no *stdout*. Esse é o compartamento esperado porque agora a nossa aplicação reaproveita as conexões que já foram abertas e estão disponíveis sempre que possível e tudo isso é gerenciado pelo nosso pool de conexões.

---

Congratulations! :+1:

Você adicionou um pool de conexões na sua aplicação utilizando a biblioteca **Apache Commons DBCP** e agora sua aplicação gerencia as conexões com o banco de dados de forma eficiente. 

Voltar para [LABORATÓRIO 5](./LABORATORIO-5.md) *ou* ir para [LABORATÓRIO 7](./LABORATORIO-7.md)
