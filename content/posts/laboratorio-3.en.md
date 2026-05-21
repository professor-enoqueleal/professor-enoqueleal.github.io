+++
title = "LAB 3 - Creating the SELECT method"
description = "This lab presents the basic steps to create a query layer in the data persistence layer!"
date = 2025-03-07
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

This lab presents the basic steps to create a query layer in the data persistence layer!

After completing this lab, you should be able to:

- Make HTTP requests through an HTML form and capture the request data in a Servlet;
- Query data that was persisted in the database (select * from car) and display the data in an HTML form.

## This lab is also available as a video lesson on YouTube

{{< youtube r9LFSecJlFE >}}

## Task 1: Creating the findAllCar() method

Now that you already have your application properly created, your web server is running, and you can save data to the database, it's time to implement a query layer for the data persisted in your application's database.

1 - To do this, let's refactor the **Car** model class by removing the setter method and creating an overloaded constructor. Open the Car class, delete the getName() method and create a constructor that receives name as a parameter.

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

2 - Due to this refactoring in the **Car** class, it is necessary to make an adjustment in the **CreateCarServlet.java** class, removing the line **car.setCarName(carName)** and passing the carName parameter via constructor **Car car = new Car(carName)**.

After the refactoring, the **CreateCarServlet.java** class code should be equal to the following:

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

3 - Now let's create a new method called **findAllCars** inside the CarDao class. To do this, open the CarDao class.

Create the **findAllCars** method that returns a list of type **Car**. The resulting code should be equal to the following:

```java
public List<Car> findAllCars() {

}
```
At this point your IDE will be alerting an error because the return statement has not been implemented yet. Don't worry because this will be resolved in the next step.

4 - Now let's implement the logic that queries data from the database. The first step is to create our SQL string containing the SQL command for the query (SELECT * FROM).

The resulting code should be equal to the following:

```java
public List<Car> findAllCars() {

     String SQL = "SELECT * FROM CAR";

}
```

This command will query our database, in the CAR table, and return all existing records in this table.

5 - The rest of the implementation will be very similar to the **createCar()** method implementation. We will have the *try / catch* block and also feedback messages so we can know whether the operation was successful or not.

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

6 - Now that we have implemented the findAllCar method that returns a list with all cars registered in our Car table, we can proceed with creating our Servlet responsible for receiving query requests.

In the servlet package, select the *New* option and then the *Java Class* option. In the creation wizard, type the class name: **ListCarServlet**.

The structure of this servlet will be similar to the **CreateCarServlet** servlet structure, but the endpoint registered in the **@WebServlet** annotation should be **find-all-cars** and the method to be overridden (Override) is the **doGet()** method.

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

7 - Now with the **ListCarServlet** class created and the doGet method overridden, we can implement the logic that will call the CarDao class and the **findAllCars()** method.

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

Notice that we are now redirecting the user to a new page called dashboard.jsp. This page does not yet exist and we will create it in the next step.

8 - Now let's create the page to display the result of our query in a table on our HTML page.

Create a new page called **dashboard.jsp** inside the **webapp** folder. Inside this page, create a table with two columns, one column for the *ID* property and the other for the *Name* property.

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

9 - Now let's add the **JSTL** (The JavaServer Pages Standard Tag Library) taglib import. It is through this tag library that we can use IF or FOR inside our page.

After adding the tag library import and the for loop, the resulting code should be equal to the following:

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

10 - The last step is to redirect the user to the new page after creating (insert) a new car in our database. To do this, in the **CreateCarServlet** class, replace the line containing the code:

```java
 req.getRequestDispatcher("index.html").forward(req, resp);
```
With the following line:

```java
resp.sendRedirect("/find-all-cars");
```

After that, upon creating a new car, the request will be redirected to our new **ListCarServlet** servlet which will execute the **doPost()** method, query the database, and finally redirect the user to the dashboard.jsp page where the data registered in the database will be rendered in the browser.

11 - Review everything that has been done so far!

---

Congratulations! :+1:

You created the second part of the CRUD (read). You implemented the findAllCar method and now the data is queried from the database and displayed in our new HTML form.
