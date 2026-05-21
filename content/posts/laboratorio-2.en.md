+++
title = "LAB 2 - Creating the INSERT method and configuring the database connection"
description = "This lab introduces the basic concepts for creating a Java Web application with a data persistence layer!"
date = 2025-02-28
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

This lab introduces the basic concepts for creating a Java Web application with a data persistence layer!

After completing this lab, you should be able to:

- Provision a persistence layer for the Java Web application;
- Start an embedded Tomcat server (Servlet Container) and an in-memory database (H2 DB) to run your Java application and persist your data;
- Make HTTP requests through an HTML form and capture the request data in a Servlet;
- Save data captured from an HTML form and persist it in a database (insert).

## This lab is also available as a video lesson on YouTube

### PART 1
{{< youtube Q9G_xCI7fq0 >}}

### PART 2
{{< youtube Du4fwItGElU >}}

## Task 1: Add a new dependency to your project

Now that you already have your application properly created and your web server is running, it's time to add a data persistence layer to your application.

1 - Open the **pom.xml** file

2 - Locate the **dependencies** block. You should add a new dependency inside this block.

**NOTE**: No dependency should be removed in this process. Add the following dependency inside the **dependencies** block:

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

You added the H2 DB (in-memory database) dependency.


## Task 2: Registering the H2 DB listener in the web.xml file

:warning: To learn more about H2 DB, visit the official documentation through this link: [H2 Database Engine](https://www.h2database.com)

1 - Now that the H2 dependency has been added to the project, it will be necessary to register the H2 *listener* in the web.xml file. To do this, navigate to the web.xml file. This file is located in the directory: car-store/src/main/webapp/WEB-INF/web.xml

2 - With the web.xml file open, add the *listener* inside the *web-app* block as shown in the following code:

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

**NOTE**: No configuration should be removed in this process.

3 - The final result should be equal to the following code:

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

4 - Save all changes **(CTRL + S)** and run your application *(tomcat7:run)*. With the **H2 DB** *listener* properly registered, you can access the management console through the link: *http://localhost:8080/console*

5 - After accessing the **H2 DB** console, log in using the following information:

    * Driver Class: org.h2.Driver
    * JDBC URL: jdbc:h2:~/test
    * User Name: sa
    * Password: sa

6 - After logging in, you can create your first table. You should create a table called **CAR**. To do this, use the following **SQL** command:

```sql
CREATE TABLE CAR(ID INT PRIMARY KEY AUTO_INCREMENT, NAME VARCHAR(255));
```

![animated gif demonstrating how to create a table in H2 DB](/gifs/11-criando-tabela-no-h2.gif)

7 - Review everything that has been done so far!

Congratulations! :+1:

You added the dependency for an in-memory database (H2 DB) to your Java Web application. Now that you have added the H2 DB dependency to your application's pom.xml, when you start *(tomcat7:run)*, a relational database is available and we also have a management console.


## Task 3: Creating the first Model and the first DAO

Now that we have a database properly configured, it's time to create the first model class (Car) and the first DAO class (CarDao).

1 - Now let's create two new packages, the first one called *dao* and the second one *model*. To do this, navigate to the main package (br.com.carstore). In your project, navigate to the directory: car-store-guide/app/src/main/java/br.com.carstore, right-click on the main package (br.com.carstore) and choose the *New / Package* option and enter the name of the first package (dao). Repeat the operation to create the second package (model).

At the end of creation, the expected result is that we now have three sub-packages inside the main package following the hierarchy:

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

![image demonstrating the project structure](/images/01-project-structure.png)

2 - Now with the new packages properly created, let's create our model class called Car. To do this, right-click on the model package, choose the *New, Java Class* option, then type *Car* and press ENTER. The IntelliJ creation wizard will create a new Java class called **Car**.

3 - With the Car class properly created, let's create an attribute of type *String* with the *private* access modifier and named **name**.

4 - Create the accessor methods (getters and setters) for the attribute we just created *(private String name)*.

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

5 - Now that we have the Car class properly created, we can create our CarDao class. To do this, right-click on the dao package, choose the *New, Java Class* option, then type *CarDao* and press ENTER. The IntelliJ creation wizard will create a new Java class called **CarDao**.

6 - With the **CarDao** class created, let's implement a method called **createCar** that returns **void** (returns nothing) and receives an object of type Car as a parameter.

At this stage, the resulting code should be equal to the following:

```java
package br.com.carstore.dao;

import br.com.carstore.model.Car;

public class CarDao {

    public void createCar(Car car) {

    }

}

```

7 - Now with the *createCar* method properly created, let's implement the persistence logic for the car object values in our database. At this stage, the resulting code should be equal to the following:

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

8 - Pay attention to the entire implementation and imports. At this stage, no errors should be flagged by your IDE. If your IDE flags any errors, review all previous steps to ensure everything is correct.

9 - With the entire implementation properly done, you can now test the project to ensure everything works correctly.

![animated gif demonstrating the project working and saving data to H2 DB](/gifs/12-mostrando-o-projeto-funcionando.gif)

10 - Review everything that has been done so far!

---

Congratulations! :+1:

You added the dependency for an in-memory database (H2 DB) to your Java Web application, created the first model and DAO classes. You implemented the logic to open a connection with the database (H2) and the persistence command (insert) for data into the database.

Now when you start *(tomcat7:run)*, when the application starts we have a relational database at our disposal and we also have a management console. At this point, the create part of our CRUD is implemented and working.
