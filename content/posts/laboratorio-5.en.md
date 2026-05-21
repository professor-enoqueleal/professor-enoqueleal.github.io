+++
title = "LAB 5 - Creating the UPDATE method"
description = "This lab aims to present a basic approach on how to update data in a database table!"
date = 2025-04-11
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++

## Lab overview and objectives

This lab aims to present a basic approach on how to update data in a database table!

After completing this lab, you should be able to:

- Make HTTP requests through an HTML form and capture the request data in a Servlet;
- Update data that was persisted in the database (update ... where id = ?).

## This lab is also available as a video lesson on YouTube

{{< youtube nZwDype_1p8 >}}

## Task 1: Creating the updateCar() method

Now that you already have your application properly created, your web server is running, and you can save, query, and delete data from the database, it's time to implement the operation to update data that was persisted in your application's database.

1 - To do this, let's create a new method called **updateCar()** that receives a Car object as a parameter. This method should be created inside the **CarDao** class.

NOTE: The **CarDao** class already exists and has three methods: **createCar()**, **findAllCars**, and **deleteCarById()**. In this section, we will only add a new method inside this class. The method name should be **updateCar()**. This method will receive a *Car* object as a parameter and its return type will be *void*.

To do this, open the **CarDao** class and implement the **updateCar()** method. The resulting code should be equal to the following:


```java
public void updateCar(Car car) {

}
```

2 - Now let's implement the logic that updates the data in our table. The first step is to create a *string* variable. We will call this variable SQL. This variable receives as its value a string containing the SQL command to perform the update (UPDATE ... WHERE ID = ?).

The resulting code should be equal to the following:

```java
public void updateCar(Car car) {

    String SQL = "UPDATE CAR SET NAME = ? WHERE ID = ?";

}
```

NOTE: This command will update the data stored in our table according to the provided ID. It will update the data contained in the fields specified through the **SET** command. This example is extremely simple. Pay close attention when replicating this function in your project.


3 - The rest of the method implementation will be very similar to the implementation we did previously in the **createCar()** method. We will have the *try / catch* block and also feedback messages so we can know whether the operation was successful or not.

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

NOTE: Don't worry about code duplication at this point. In the next labs, we will refactor to remove duplicated code *(Boilerplate)*.

4 - Now that we have implemented the **updateCar()** method that receives a *Car* object as a parameter and executes the logic for updating a car record in our table, we can proceed with refactoring the *Servlet* that will be responsible for receiving *update* requests.

In the *br.com.carstore.servlet* package, find the *Servlet* called **CreateCarServlet**.

This Servlet already exists and will receive some changes. Now when receiving a request in the *doPost()* method, in addition to collecting the *car-name* parameter, we will also collect the *ID* and store it in a String variable called *carId*.

After this implementation, the resulting code should be equal to the following:

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

5 - Now we need to validate whether the value of the *carId* variable is empty. If it is empty (does not contain an ID), it means it is a create request. And if the value of the *carId* variable is not empty (contains an ID), it means it is an update request.

For this logic, we will use the *isBlank()* method from the **String** class that was introduced in Java version 11.

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

NOTE: Notice that if it is a create *(insert)* request, we will continue calling the **createCar()** method; otherwise, we will now call the new **updateCar()** method. The rest of the logic remains the same.

Done, our *DAO* and *Servlet* classes are ready to receive requests and execute the update or create command in our database.

After executing the call to the **carDao** class and the **updateCar** method, a request is made to the **/find-all-cars** endpoint which in turn executes a new query on the database and displays the updated data on the **dashboard.jsp** page.

## Task 2: Refactoring the dashboard.jsp page

Now that we have built the logic to update a car's data, we need to refactor the *dashboard.jsp* page and add the *anchor / hyperlink* that will redirect the request to the *index.jsp* page, sending the data properly filled in the input fields.

1 - To do this, open the *dashboard.jsp* page. Locate the *form* we created in the previous lab inside the *table* and add an *anchor* inside the Actions column, next to the delete *button*, before the form closing tag. We will also add a label so that visually the *button* and the *anchor* are separated.

After the implementation, the resulting code should be equal to the following:

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

NOTE: Notice that the *anchor* will create a *hyperlink* to the *index.jsp* page. This operation will send some parameters such as the *car.id* and *car.name* variables. This is necessary so we can access these values on the *index.jsp* page as will be demonstrated below.

2 - Changing the **index** page extension

The next step is to change the index page extension from **.html** to **.jsp**. This is necessary so we can access the values that were sent as parameters in the request made through the *hyperlink* from the *dashboard.jsp* page to the *index.jsp* page.

In IntelliJ, navigate to the **index.html** page located in the directory: car-store-guide/app/src/main/webapp. Right-click on the index.html file, select the *refactor / rename* option, and change the extension from *.html* to *.jsp*.

![animated gif demonstrating how to change the page extension](/gifs/15-trocando-a-extensao.gif)

3 - Refactoring the *index.jsp* page

Now that the index page has the *.jsp* extension, we can make the necessary refactoring on the page so that our *update* operation works correctly.

With the *index.jsp* page open, we will add a new *input* field of type *hidden* to store the ID variable ${param.id} and add the value property to the input field to store the variable ${param.name}.

After the refactoring, the resulting code should be equal to the following:

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

Save all changes **(CTRL + S)** and run your application *(tomcat7:run)*. Access your application through the link http://localhost:8080 and register a car. Notice that now after the registration screen, in the *dashboard.jsp* page table, a *hyperlink* with the *Update* label is displayed. When you click this hyperlink, you will be redirected back to *index.jsp*, but the fields will be properly filled with the values displayed in the table. Simply change the value and click the register button. The user will be redirected to *dashboard.jsp* and the values will be updated in the database as demonstrated in the animated gif below.

![animated gif demonstrating how to change the page extension](/gifs/16-testando-update.gif)

5 - Changing the button text *(Optional)*

For organizational purposes, let's change the button text on the *index.jsp* page from *Register* to *Save*.

In IntelliJ, navigate to the *index.jsp* page located in the directory: car-store-guide/app/src/main/webapp. Open the file and locate the **register** button.

Change the word *Register* to *Save*.

---

Congratulations! :+1:

You created the fourth part of the CRUD (update). You implemented the *updateCar()* method and now data can be updated in the database.
