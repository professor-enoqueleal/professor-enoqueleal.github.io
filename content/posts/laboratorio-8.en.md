+++
title = "LAB 8 - Creating the image upload feature (WIP)"
description = "This lab aims to present a basic approach on how to implement the ability to upload images!"
date = 2024-05-08
draft = true
author = "Enoque Leal"
tags = [ "java", "html", "web", "servlet" ]
+++


## Lab overview and objectives

Este laboratório visa apresentar uma forma básica sobre como implementar a capacidade de fazer upload de imagens.

After completing this lab, you should be able to:

- Fazer requisições http através de um formulário HTML e capturar os dados dessa requisição em uma Servlet;
- Alterar uma tabela no banco de dados, criando uma nova coluna para armazenar o caminho (path) da imagem;
- Alterar a DAO (Data Access Object), inserindo a nova variável para trafegar o caminho da imagem;
- Alterar o formulário HTML para capturar o arquivo / imagem no momento do cadastro;

## This lab is also available as a video lesson on YouTube

{{< youtube XwzLNVX5yWE >}}
{{< youtube pGGYm7UekpA >}}
{{< youtube 0gx1M4nFmG0 >}}

## Task 1: Alterando a tabela CAR

Para podermos implementar a funcionalidade de *upload de imagens*, precisamos alterar a estrutura da tabela **CAR** adicionando uma nova coluna que irá armazenar o caminho (path) onde a imagem foi salva.

1 - Execute a aplicação (**tomcat7:run**) e aguarde toda a etapa de inicialização. Quando a inicialização finalizar, com a aplicação em execução, acesse a console de gerenciamento do H2DB através do link: [http://localhost:8080/console](http://localhost:8080/console)

2 - After accessing the console of the **H2 DB**, log in using the following information:

* **Driver Class:** org.h2.Driver
* **JDBC URL:** jdbc:h2:~/test
* **Username:** sa
* **Password:** sa

3 - After logging in, let's change a estrutura da tabela CAR adicionando uma nova coluna chamada **IMAGE** do tipo **VARCHAR** com o tamanho de **255**. O tamanho é importante porque dependendo do diretório onde a imagem for armazenada, pode ocupar muitos caracteres. 

To do this utilize o comando **SQL** a seguir:

```sql
ALTER TABLE CAR ADD IMAGE VARCHAR (255);
```

---

## Task 2: Alterando as classes Car.java e CarDao.java

Com a estrutura da tabela properly prepared para receber o caminho da imagem, precisamos alterar também as classes **Car.java** e **CarDao.java** adicionando uma nova variável do tipo String que iremos nomear também de image.

1 - In IntelliJ navigate to the package *(package)* model que fica no diretório: car-store-guide/app/src/main/java/br.com.carstore.model e open the class **Car.java**

2 - Com a classe properly opened, let's add uma nova variável chamada **image**, do tipo **String** e com o modificador **private**.

3 - Após criar a variável **image**, precisamos criar o método getter (getImage()) e também um novo construtor que vai receber os três parâmetros, sendo eles: id, name e image. Dessa forma, agora podemos criar uma instância de Car passando os três parâmetros via construtor.

The resulting code of the implementation should be equal to the following:

```java
package br.com.carstore.model;

public class Car {

    private String id;
    private String name;
    private String image;

    public Car(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public Car(String id, String name, String image) {
        this.id = id;
        this.name = name;
        this.image = image;
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getImage() {
        return image;
    }

}

```

4 - Agora let's refactor a classe **CarDao.java**. To do this, no IntelliJ navigate to the package *(package)* **dao** que fica no diretório: car-store-guide/app/src/main/java/br.com.carstore.dao e open the class **CarDao.java**

Com a classe properly opened, let's refactor os métodos **createCar()**, **findAllCars()** e **updateCar()**. Precisamos refatorar esses métodos para que eles contemplem o novo campo **IMAGE** nas suas operações.

5 - Refatorando o método **createCar()**. O Primeiro paço é alterar nossa String **SQL**, porque atualmente o comando está preparado apenas para **INSERT** de 1 campo, no caso o **NAME**. Precisamos alterar o comando para que ele contemple o **INSERT** em 2 campos, sendo eles **NAME** e **IMAGE**.

Sendo assim, let's change a String SQL:

**DE**
```sql
INSERT INTO CAR (NAME) VALUES (?)
```

**PARA**
```sql
INSERT INTO CAR (NAME, IMAGE) VALUES (?, ?)
```
Agora a nossa String **SQL** esta preparada para fazer o **INSERT** nas colunas **NAME** e **IMAGE**.

Na sequência, precisamos preparar o preparedStatement para inserir o segundo parâmetro que foi adicionado a nossa string SQL. To do this, basta adicionar mais uma linha:

```java
preparedStatement.setString(2, car.getImage())
```

The resulting code of the implementation should be equal to the following:

```java
public void createCar(Car car) {

    String SQL = "INSERT INTO CAR (NAME, IMAGE) VALUES (?, ?)";

    try {

        Connection connection = DriverManager.getConnection("jdbc:h2:~/test", "sa","sa");

        System.out.println("success in database connection");

        PreparedStatement preparedStatement = connection.prepareStatement(SQL);

        preparedStatement.setString(1, car.getName());
        preparedStatement.setString(2, car.getImage());
        preparedStatement.execute();

        System.out.println("success in insert car");

        connection.close();

    } catch (Exception e) {

        System.out.println("fail in database connection");

    }

}
```

Com isso a refatoração do método **createCar()** está pronta. 

6 - Refatorando o método **findAllCar()**. Agora que já refatoramos o método **createCar()** e garantimos que o caminho da imagem esta sendo persistido no nosso banco de dados, it's time to garantir que essas informações estão sendo retornadas nas nossas consultas. To do this, precisamos refatorar o método **findAllCars()** inserindo o novo campo **(IMAGE)**. 

A String SQL nesse caso não precisa ser alterada porque ela esta retornando todos os campos (select * ...). Portando, só precisamos adicionar o **getString("image")** no momento da interação do nosso **resultSet**. To do this, faça a seguinte alteração:

```java
// adicione essa linha dentro do bloco while
String image = resultSet.getString("image");

// Inseria o parâmetro image no momento de criar uma nova instância de Car
Car car = new Car(carId, carName, image);

```

The resulting code of the implementation should be equal to the following:
```java

public List<Car> findAllCars(){

    // código acima omitodo ...

    while(resultSet.next()){

        String carId=resultSet.getString("id");
        String carName=resultSet.getString("name");
        String image=resultSet.getString("image");

        Car car=new Car(carId,carName,image);

        cars.add(car);

    }

    // código abaixo omitodo ...

}
```

O restante do código foi omitido por questões de legibilidade.

Com isso a refatoração do método **findAllCars()** está pronta.

7 - Refatorando o método **updateCar()**. Agora que já refatoramos o método **findAllCars()** e garantimos que o caminho da imagem esta sendo retornado do nosso banco de dados, it's time to garantir que essas informações estão sendo atualizadas corretamente. To do this, precisamos refatorar o método **updateCar()** inserindo o novo campo **(IMAGE)**.

A refatoração desse método será similar a do método **createCar()**. Precisamos alterar a String SQL e inserir os parâmetros (setString) corretamente no preparedStatement. 

Sendo assim, let's change a String SQL:

**DE**
```sql
UPDATE CAR SET NAME = ? WHERE ID = ?
```

**PARA**
```sql
UPDATE CAR SET NAME = ?, IMAGE = ? WHERE ID = ?
```
Agora a nossa String **SQL** esta preparada para fazer o **UPDATE** nas colunas **NAME** e **IMAGE** conforme o ID especificado.

NOTE: Atenção ao próximo passo, pois numero do index do ID vai mudar de 2 para 3, porque agora a imagem é o segundo parâmetro e o ID vai ser o terceiro.

Insira o código a seguir:

```java
preparedStatement.setString(2, car.getImage())
```

The resulting code of the implementation should be equal to the following:

```java
public void updateCar(Car car) {

    String SQL = "UPDATE CAR SET NAME = ? WHERE ID = ?";

    try {

        Connection connection = ConnectionPoolConfig.getConnection();

        PreparedStatement preparedStatement = connection.prepareStatement(SQL);

        preparedStatement.setString(1, car.getName());
        preparedStatement.setString(2, car.getImage());
        preparedStatement.setString(3, car.getId());
        preparedStatement.execute();

        System.out.println("success in update car");

        connection.close();

    } catch (Exception e) {

        System.out.println("fail in database connection");
        System.out.println("Error: " + e.getMessage());

    }

}
```
Com isso a refatoração do método **updateCar()** está pronta.

8 - Save all changes **(CTRL + S)** e faça uma revisão tudo que foi feito até aqui!


## Task 3: Incluindo a biblioteca Apache Commons File Upload

Agora que a classe CarDao já foi refatorada e esta pronta para receber o caminho do arquivo e persisti-lo no banco de dados, precisamos adicionar no nosso projeto, duas novas dependências que irão nos auxiliar no processo de upload do arquivo através de uma requisição http.

A biblioteca (library) [Commons FileUpload](https://commons.apache.org/proper/commons-fileupload/) facilita a adição de recursos robustos e de alto desempenho para upload de arquivos a seus servlets e aplicativos da web.

1 - Adicionando as dependências

In IntelliJ IDEA, open the file de configuração do projeto chamado "pom.xml" (usually located in the project root).

Locate the section <dependencies> e adicione as seguintes dependências:

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.5</version>
</dependency>

<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.12.0</version>
</dependency>
```

NOTE: No code should be removed at this stage. Just add the new dependency no *pom.xml*.

2 - Save all changes **(CTRL + S)**

After saving the changes, o IntelliJ IDEA should automatically synchronize the changes do arquivo de configuração e baixar as bibliotecas **commons-fileupload** e **commons-io**.



## Task 4: Alterando a servlet CreateCarServlet e implementando o método uploadImage()

Após adicionar a dependência no nosso projeto, podemos seguir com a implementação necessária. Três novos métodos serão criados agora, sendo eles, **uploadImage()**, **checkFieldType()** e **processUploadedFile()**. Cada um desses métodos tem sua responsabilidade bem definida e vamos falar sobre elas a seguir.

1 - Vamos abrir nossa classe CreateCarServlet e refatora-la. To do this, no IntelliJ navigate to the package *(package)* **servlet** que fica no diretório: car-store-guide/app/src/main/java/br.com.carstore.servlet e open the class **CreateCarServlet.java**.

Com a classe properly opened, let's create o método **uploadImage** que recebe por parâmetro uma instância de HttpServletRequest e devolve um Map de String. This method será responsável varrer toda a lista de parâmetros que recebemos do formulário HTML, chamar o método **checkFieldType**. This method será responsável por verificar se o conteúdo recebido do formulário HTML é um arquivo (**File**) ou apenas um texto simples (FormField). Se for um texto simples, pegaremos o nome do campo e o valor e adicionaremos no mapa de parâmetros. Se for um arquivo, o terceiro método **processUploadedFile** é chamado para realizar o upload do arquivo e armazenar o arquivo no diretório especificado.

The resulting code of the implementation should be equal to the following:

```java
private Map<String, String> uploadImage(HttpServletRequest httpServletRequest) {

    Map<String, String> requestParameters = new HashMap<>();

    if (isMultipartContent(httpServletRequest)) {

        try {

            DiskFileItemFactory diskFileItemFactory = new DiskFileItemFactory();

            List<FileItem> fileItems = new ServletFileUpload(diskFileItemFactory).parseRequest(httpServletRequest);

            for (FileItem fileItem : fileItems) {

                checkFieldType(fileItem, requestParameters);

            }

        } catch (Exception ex) {

            requestParameters.put("image", "img/default-car.jpg");

        }

    }

    return requestParameters;

}

```

2 - Creating the method **checkFieldType**. A implementação desse método consiste em uma validação condicional responsável por verificar se o campo é do tipo file ou não. Se for file o método **processUploadedFile** é chamado. Se não for, apenas pegamos o valor do campo e adicionamos no map parameters.

The resulting code of the implementation should be equal to the following:

```java

private void checkFieldType(FileItem item, Map requestParameters) throws Exception {

    if (item.isFormField()) {

        requestParameters.put(item.getFieldName(), item.getString());

    } else {

        String fileName = processUploadedFile(item);
        requestParameters.put("image", "img/".concat(fileName));

    }

}

```

3 - Creating the method **processUploadedFile()**. This method receberá por parâmetro uma instância de um objeto do tipo FileItem que contém um campo do tipo file. Com isso, criamos o nome do arquivo com base em um *timestemp*, removemos qualquer espaço em branco que exista no nome do arquivo e por último gravamos o arquivo em uma pasta chamada IMG dentro do nosso servidor.

The resulting code of the implementation should be equal to the following:

```java

private String processUploadedFile(FileItem fileItem) throws Exception {
    Long currentTime = new Date().getTime();
    String fileName = currentTime.toString().concat("-").concat(fileItem.getName().replace(" ", ""));
    String filePath = this.getServletContext().getRealPath("img").concat(File.separator).concat(fileName);
    fileItem.write(new File(filePath));
    return fileName;
}

```

4 - Agora que toda a implementação de upload foi feita, nós precisamos refatorar o método **doPost** para pegar os parametros do Mao que é devolvido pelo méotodo uploadImage ao invés de pegar do HttpServletRequest.

The resulting code of the implementation should be equal to the following:

```java
    @Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {

    Map<String, String> parameters = uploadImage(req);
    
    String carImagePath = parameters.get("image");
    String carName = parameters.get("car-name");
    
    Car car = new Car("0", carName, carImagePath);
    
    new CarDao().createCar(car);
    
    resp.sendRedirect("/find-all-cars");

}
```

5 - Save all changes **(CTRL + S)** e faça uma revisão de tudo que foi feito até aqui!


## Task 5: Alterando a página index.jsp

Agora precisamos alterar o nosso formulário index.jsp adicionando um novo input do tipo file. Esse **input** será responsável por possibilitar que de forma visual o utilizador do sistema consiga selecionar um arquivo para ser carregado.

1 - Localize o formulário **index.jsp**. To do this, no IntelliJ navegue até a pasta **webapp** que fica no diretório: car-store-guide/app/src/main/java/webapp e open the file index.jsp.

2 - Com o arquivo devidamente aberto, let's add um novo campo *input* do tipo *file* dentro do nosso formulário:

```html
<label for="image">Choose file</label>
<input type="file" name="image" id="image">
```
Na sequência precisamos adicionar a propriedade **enctype="multipart/form-data"** ao nosso formulário:
```html
<form action="/create-car" method="post" enctype="multipart/form-data">
```

The resulting code of the implementation should be equal to the following:

```html
<html>
<body>
<h2>Create Car</h2>

<form action="/create-car" method="post" enctype="multipart/form-data">

    <div>
        <label for="car-name" >Car Name</label>
        <input type="text" name="car-name" id="car-name">
    </div>
    <div>
        <label for="image">Choose file</label>
        <input type="file" name="image" id="image">
    </div>

    <button type="submit">Save</button>

</form>

</body>
</html>
```

3 - Save all changes **(CTRL + S)** e faça uma revisão tudo que foi feito até aqui!


## Task 5: Alterando a página dashboard.jsp (WIP)


Congratulations! :+1:

Você implementou a funcionalidade de upload de imagem. Agora no momento do cadastro de um carro, o usuário pode escolher uma imagem para cadastrar em conjunto com os dados do carro.

Voltar para [LABORATÓRIO 7](./LABORATORIO-7.md) *ou* ir para LABORATÓRIO 9
