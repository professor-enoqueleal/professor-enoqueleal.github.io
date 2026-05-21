+++
title = "LAB 1 - Creating the application"
description = "This lab introduces the basic concepts for creating a Web application using Java."
date = "2026-02-26"
draft = false
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

This lab introduces the basic concepts for creating a Web application using Java.

After completing this lab, you should be able to:

- Create a Web application with Java
- Start an embedded Tomcat server (Servlet Container) to run your Java application
- Make HTTP requests through an HTML form and capture the request data in a Servlet

## This lab is also available as a video lesson on YouTube

{{< youtube BtDiMAbq53E >}}

## Task 1: Create a Java application using IntelliJ

1 - Make sure you have IntelliJ installed on your computer. If you don't, install the *Community Edition*, as demonstrated in this YouTube video: [Installing IntelliJ on Windows](https://youtu.be/RBxAySum8UU). Or click the following link to *download* it: [IntelliJ download link](https://www.jetbrains.com/idea/download/download-thanks.html?platform=macM1&code=IIC)

![animated gif demonstrating how to install IntelliJ on Windows](/gifs/01-instalando-intellij.gif)

2 - Open IntelliJ

  NOTE: If this is the first time you are opening IntelliJ on your machine, you need to click the *checkbox* to confirm that you have read the tool's terms of use and then click the *continue* button.

  On the next screen, you need to choose whether you want to anonymously share your usage data. Click *Don't Send*.

  If this is not the first time you are using IntelliJ on this computer, disregard this information.

3 - After IntelliJ is open, on the welcome screen, click the *Projects* menu and then click the *New Project* button.

4 - On the new project wizard screen, click on **Maven Archetype**, in this section, configure:

  - **Name**: carstore
  - **Location**: Keep the default value
  - **JDK**: Choose the installed JDK from the dropdown menu
    NOTE: If you don't have any JDK installed, click *Download JDK* and in the dropdown menu, select version 11 in the version field and then click the *Download* button.
  - **Catalog**: Keep the default value
  - **Archetype**: maven-archetype-webapp

  In the **Advanced Settings** section, configure:
  - **GroupId**: br.com.carstore
  - **ArtifactId**: carstore
  - **Version**: 1.0-SNAPSHOT


5 - Click the *Create* button

![animated gif demonstrating how to create a new project using a Maven archetype for a web project](/gifs/02-criando-o-projeto.gif)

After that, just wait for the entire loading process to finish. At the end, your application will be ready.

NOTE: The loading process may take a few minutes; it is essential to wait for the entire loading stage to finish.

6 - Review everything that has been done so far!

---

## Task 2: Add the Tomcat plugin and the Maven War Plugin

Now that we have the application properly created, it's time to add the Tomcat plugin (Servlet Container). This way, it will be possible to run the Web application on the Tomcat server without additional effort.

1 - With the application created, open the *pom.xml* file

2 - The pom.xml file is used by Maven for the build and packaging process of a Java application. This file is composed of different sections. Find the *<build>* section. Inside this section, add the following code block.

NOTE: No code should be removed at this stage. Just add the following code inside the *<build>* block:

```xml
<plugins>
  <plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.1</version>
    <configuration>
      <path>/</path>
    </configuration>
    <executions>
      <execution>
        <id>tomcat-run</id>
        <goals>
          <goal>exec-war</goal>
        </goals>
        <phase>package</phase>
        <configuration>
            <enableNaming>false</enableNaming>
        </configuration>
      </execution>
    </executions>
  </plugin>
</plugins>
```
![animated gif demonstrating how to add the Tomcat plugin](/gifs/03-adicioando-o-plugin-do-tomcat.gif)

3 - Add a second plugin inside the *plugins* block that was created in the previous step, as shown in the following code:

```xml
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.2.2</version>
    <configuration>
        <webXml>src\main\webapp\WEB-INF\web.xml</webXml>
        <warSourceDirectory>src/main/webapp</warSourceDirectory>
    </configuration>
</plugin>
```
![animated gif demonstrating how to add the maven-war plugin](/gifs/04-adicioando-o-plugin-do-maven.gif)

4 - The resulting code should be equal to the following:

```xml
<plugins>
  <plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.1</version>
    <configuration>
      <path>/</path>
    </configuration>
    <executions>
      <execution>
        <id>tomcat-run</id>
        <goals>
          <goal>exec-war</goal>
        </goals>
        <phase>package</phase>
        <configuration>
            <enableNaming>false</enableNaming>
        </configuration>
      </execution>
    </executions>
  </plugin>
  <plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.2.2</version>
    <configuration>
        <webXml>src\main\webapp\WEB-INF\web.xml</webXml>
        <warSourceDirectory>src/main/webapp</warSourceDirectory>
    </configuration>
  </plugin>
</plugins>
```

5 - Save all changes **(CTRL + S)** and then click the *Load Maven Changes* button. Maven will detect that new *plugins* have been added to the project and will automatically download them. Wait for the loading to finish.

6 - Once done, you can now run the application and render your first web page in the browser. To do this, navigate to the Maven menu inside IntelliJ, expand the *carstore* project, then click on *plugins*, then click on *tomcat7*, and finally double-click on the *tomcat7:run* option. This will start the web server (Servlet Container).

7 - After the loading process, open a tab in your browser and type the following address: http://localhost:8080

8 - A web page should be rendered containing the message **Hello, world!**

![animated gif demonstrating how to run the created project](/gifs/05-executando-o-servidor.gif)

9 - Review everything that has been done so far!

---

## Task 3: Creating your first Servlet and making an *http* request

Now that you have your application properly created and your web server is running, it's time to create your first Servlet and make your first **http** request.

1 - Open the *pom.xml* file again

2 - Locate the *</dependencies>* block. You should add two dependencies inside the *</dependencies>* block in your application's **pom.xml**, as shown in the following code.

NOTE: No code should be removed at this stage. Just add the following code inside the *<dependencies>* block:

```xml
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
```
![animated gif demonstrating how to add dependencies to the project's pom.xml](/gifs/06-adicionando-dependencias.gif)

3 - Save all changes **(CTRL + S)** and then click the *Load Maven Changes* button. Maven will detect that two new *dependencies* have been added to your project and will automatically download them for you. Wait for the loading to finish.

4 - After the loading/download of the new *dependencies* is complete, go back to the *Project* tab and navigate in your project by clicking on the *carstore* folder, *src*, and then *main*. Right-click on the *main* folder and choose the *New* option, then *Directory*. The new directory creation wizard will open; select the **Java** option.

5 - After the **Java** directory has been created, let's create a package so we can create our first **Java** class. To do this, right-click on the Java directory, choose the *New* option, then the *Package* option, and in the creation wizard type: *br.com.carstore.servlet*. This will be the default package for our application.

![animated gif demonstrating how to create the Java directory and the br.com.carstore package](/gifs/07-criando-diretorio-java-e-package.gif)

6 - Now let's create our first Java class (Servlet). Right-click on the package we just created (*br.com.carstore.servlet*), select the *New* option, then the *Java Class* option. In the creation wizard, type the class name: **CreateCarServlet**

7 - Now with your first *servlet* properly created, you need to add the *@WebServlet* annotation. This annotation should be placed one line above where the class name **CreateCarServlet** is declared, then add the extension (extends) for the HttpServlet class. The resulting code should look like the following:

```java
package br.com.carstore.servlet;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;

@WebServlet("/create-car")
public class CreateCarServlet extends HttpServlet {

}
```
![animated gif demonstrating how to create the CreateCarServlet class and add the @WebServlet annotation](/gifs/08-criando-servlet.gif)

8 - Now that the *Servlet* class has been properly created, let's override the **doPost()** method. This method is responsible for receiving HTTP POST requests made to our application.

To do this, with the **CreateCarServlet** class open, position the cursor inside the class (between the opening and closing braces) and use the keyboard shortcut **(CTRL + O)**. The IntelliJ method override wizard will open. In the list of available methods, search for the **doPost()** method and double-click on it.

![animated gif demonstrating how to override the doPost method](/gifs/09-override-dopost.gif)

9 - Now with the **doPost()** method properly created, let's implement the logic to capture the value sent via the HTML form.

For this, we need to use the **req** (request) object. It is through this object that we can capture values sent via our HTML forms.

The resulting code should be equal to the following:

```java
package br.com.carstore.servlet;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/create-car")
public class CreateCarServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        String carName = req.getParameter("car-name");

        System.out.println(carName);

        resp.sendRedirect("index.html");

    }

}
```

![animated gif demonstrating how to implement the getParameter method](/gifs/10-implementando-get-parameter.gif)

10 - Now let's change the *index.html* page of our project so that we have an HTML form (form) that submits data to our **CreateCarServlet** via *HTTP POST*.

To do this, navigate to the *index.html* file that is inside the *webapp* directory: carstore/src/main/webapp. Open the *index.html* file and replace all the existing code with the following code:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Car Store</title>
</head>
<body>

    <h1>Car Store</h1>

    <form action="/create-car" method="post">

        <label>Car Name</label>
        <input type="text" name="car-name" id="car-name">

        <button type="submit">Register</button>

    </form>

</body>
</html>
```

11 - Save all changes **(CTRL + S)** and run your application again *(tomcat7:run)*.

After the loading process, open a tab in your browser and type the following address: http://localhost:8080

A web page should be rendered containing a form with the **Car Store** label, a text field labeled **Car Name** and a button labeled **Register**.

Type a car name in the text field and click the register button.

![animated gif demonstrating the project working](/gifs/12-mostrando-o-projeto-funcionando.gif)

12 - Review everything that has been done so far!

---

Congratulations! :+1:

You created your first Java application. Created a Web project using IntelliJ. Installed the Tomcat Embed (Servlet Container). Created your first Java Servlet class and received your first HTTP request.
