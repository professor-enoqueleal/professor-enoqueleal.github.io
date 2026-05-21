+++
title = "LAB 4 - Creating the DELETE method"
description = "This lab aims to present a basic approach on how to delete data from a database table!"
date = 2025-04-04
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

This lab aims to present a basic approach on how to delete data from a database table!

After completing this lab, you should be able to:

- Make HTTP requests through an HTML form and capture the request data in a Servlet;
- Delete/remove data that was persisted in the database (delete car where id = ?).

## This lab is also available as a video lesson on YouTube

{{< youtube b3LSVup5AMc >}}

## Task 1: Creating the deleteCarById() method

Now that you already have your application properly created, your web server is running, and you can save and query data from the database, it's time to implement the operation to delete/remove data that was persisted in your application's database.

1 - To do this, let's create a new method called **deleteCarById()**. This method should be created inside the **CarDao** class.

NOTE: The **CarDao** class already exists and has two methods: **createCar()** and **findAllCars**. In this section, we will only create/add a new method inside this class. The method name should be **deleteCarById()**. This method will receive a String as a parameter and its return type will be *void*.

To do this, open the **CarDao** class and implement the **deleteCarById()** method. The resulting code should be equal to the following:


```java
public void deleteCarById(String carId) {

}
```

2 - Now let's implement the logic that removes/deletes data from the database. The first step is to create a string variable that we will call SQL. This variable receives as its value a string containing the SQL command to perform the delete (DELETE CAR WHERE ID = ?).

```java
public void deleteCarById(String carId) {

    String SQL = "DELETE CAR WHERE ID = ?";

}
```

NOTE: This command will remove the data stored in our table according to the provided ID.


3 - The rest of the method implementation will be very similar to the implementation we did previously in the **createCar()** method. We will have the *try / catch* block and also feedback messages so we can know whether the operation was successful or not.

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

NOTE: Don't worry about code duplication at this point. In the next labs, we will refactor to remove duplicated code *(Boilerplate)*.

4 - Now that we have implemented the **deleteCarById()** method that receives an ID as a parameter and executes the logic for removing a car record from our table, we can proceed with creating the *Servlet* that will be responsible for receiving *delete* requests.

In the br.com.carstore.servlet package, select the *New* option and then the *Java Class* option. In the creation wizard, type the class name: **DeleteCarServlet**.

The structure of this servlet will be similar to the **CreateCarServlet** servlet structure, but the endpoint registered in the **@WebServlet** annotation should be **/delete-car** and the method to be overridden (Override) should be the **doPost()** method.

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

Done, our *DAO* and *Servlet* classes are ready to receive the request and execute the delete command in our database.

After executing the call to the **carDao** class and the **deleteCarById** method, a request is made to the **/find-all-cars** endpoint which in turn executes a new query on the database and displays the updated data on the **dashboard.jsp** page.

---

## Task 2: Refactoring the code to return the ID

For the *delete* command to work correctly, we need the car's ID to be returned at the time of the query. For this, we will need to refactor our code.

1 - The first step is to refactor the *model* class **Car**, adding a new variable of type *String* called *id* and a new overloaded constructor that receives *ID* and *Name* as parameters. Open the Car class located inside the *br.com.carstore.model* package, create a new private variable of type String called ID and create its respective *(getter)* method getId(). Then, create a constructor that receives *id* and *name* as parameters.

NOTE: No code should be removed at this stage. Just add a new variable and the overloaded constructor.

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

2 - Now it will be necessary to refactor the **findAllCars()** method that was created in Lab 3, so that when iterating *(while)* through the *resultSet*, it retrieves two properties: *id* and *name*.

The existing code is the following:

```java

while (resultSet.next()) {

    String carName = resultSet.getString("name");

    Car car = new Car(carName);

    cars.add(car);

}

```

After the refactoring, the resulting code should be equal to the following:


```java

while (resultSet.next()) {

    String carId = resultSet.getString("id");
    String carName = resultSet.getString("name");

    Car car = new Car(carId, carName);

    cars.add(car);

}

```

3 - Now it will be necessary to refactor the HTML form (dashboard.jsp) to display the ID at the time of the query. Open the **dashboard.jsp** page and add the variable *${car.id}*

The existing code is the following:

```html
<c:forEach var="car" items="${cars}">
    <tr>
        <td></td>
        <td>${car.name}</td>
    </tr>
</c:forEach>

```

After the refactoring, the resulting code should be equal to the following:

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

5 - Make sure everything up to this point is working properly. Save everything (CTRL + S) and test your application *tomcat7:run*, register a car and check if the ID is now displayed when the *dashboard.jsp* page is rendered as shown in the following image:

![animated gif demonstrating the car ID being displayed in the table](/gifs/13-exibindo-o-campo-id.gif)

## Task 3: Refactoring the dashboard.jsp page

Now that the car ID is displayed on the page, it's time to implement the logic for removing the car in the *dashboard.jsp* HTML form.

1 - To do this, open the *dashboard.jsp* page again. Inside the table, add one more column (th). The label will be *Actions*.

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

2 - The value to be filled in will be a form that, when submitted, will make an HTTP request to our new **DeleteCarServlet** servlet passing the vehicle ID as a parameter.

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

![image showing the delete button being rendered in the listing form](/images/02-formulario-com-o-botao.png)

3 - Review everything that has been done so far!

![animated gif demonstrating the delete feature working](/gifs/14-delete-funcionando.gif)

---

Congratulations! :+1:

You created the third part of the CRUD (delete). You implemented the *deleteCarById()* method and now data can be deleted from the database.
