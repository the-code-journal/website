---
date: 2021-05-30
linktitle: Build it with Spring Boot - CRUD Operations - Part 1
next: /2021/05/build-it-with-spring-boot-crud-operations-part-2/
prev: /2021/05/curious-case-of-modulo-operator/
title: Build it with Spring Boot - CRUD Operations - Part 1
weight: 10
tags: [ "spring-boot", "java", "crud", "applications" ]
categories: [ "Spring Boot", "Java" ]
---


This series builds Java applications using [Spring Boot 2][1], which has become de-facto standard for building applications in Java world. It makes it easy to create stand-alone, production-grade Spring based Applications. It cuts down on a lot of boilerplate code, and its sensible defaults for configurations allows to build applications in super quick time.

The project we are building here is a Spring Boot Web application that demonstrates [CRUD operations][2]. The application allows to manage student information, by a paginated student list, adding a new student, viewing student information, editing them, and deleting them. The application also has Login/Logout feature.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/05/build-it-with-spring-boot-crud.gif" alt="Build it with Spring Boot - CRUD Operation Application" />
</div>
{{< /rawhtml >}}

Here are all the articles of this series.

- [**Part 1 (this article)**][19] - Introduction to the Application and Bootstrapping the application, and configure the Web Server.
- [**Part 2**][20] - Configure the Database, create JPA Entities and JPA Repositories, Service classes and Junit tests for these
- [**Part 3**][21] - Building the UI pages using Thymeleaf and Bootstrap
- [**Part 4**][22] - Making the application work with UI pages, with data coming from Database
- [**Part 5**][23] - Adding Login/Logout using Spring Security and closure

In this article, we will boostrap our application from scratch and configure the Web Server. Below is the video for the same that guides you for this part.

{{< yt "https://www.youtube.com/embed/SSqIvBrNMkY" >}}





## CRUD Operations

What are [**"CRUD"** operations][2]? It is the short form of basic operations that a system dealing with data can have - **`CREATE, READ, UPDATE`** and **`DELETE`**. Any application that deals with data, will surely have some of these operations, if not all.

And these are any application you can think of - Education systems, e-commerce, retail, medical and healthcare, social platforms with blogs, articles and posts, financial applications like banking and insurance, gaming, etc. These kind of applications are everywhere.



## Technology Stack

We will be building this application with these technologies.

- **`Web Server (Undertow)`** - We will be using [Undertow][3] as our web server. Spring Boot by default comes with [Apache Tomcat][17], but I personally have found [Undertow][3] to be a lot more performant. You can ofcourse still go with Tomcat or even Jetty, and the application should still work fine.
- **`Spring MVC`** - We will use [Spring MVC][4] for our request handling and facilitating fetching data from the database and passing it onto the UI for user to see.
- **`Database (H2)`** - We will be using in-memory [database H2][5]. You can of course configure any other Relational Databases like MySQL or Postgres or MariaDB, and they would work just fine. I chose [H2][5] just to keep things simple.
- **`Java Persistence API`** - For a consistent access to the database, we will be using [Java Persistence API][6] as the abstraction layer.
- **`Hibernate`** - [Hibernate][7] will do the heavy lifting of taking the JPA metadata and making the actual calls to the database.
- **`Thymeleaf`** - For our UI, we are going to use [Thymeleaf][8]. You can use other View technologies like JSP, but [Thymeleaf][8] is simple and it allows to do things in parallel. The UI can be built separately without backend stuff being ready. The biggest advantage that I see is that [Thymeleaf][8] templates work without any server. You can open them in regular browser and the layout would just work fine.
- **`Bootstrap`** - And to make our life easier with styling, we will use [Bootstrap][9]. You can build complicated layouts with [Bootstrap][9], but we are using it here to make our UI pretty.
- **`Spring Security`** - To add security, we will be using the [Spring Security][10] module, and configure Login and Logout functionality from there.
- **`Junit 5`** - Last but not the least, not to forget about testing, we will write our Unit test classes with [Junit Jupiter or Junit 5][11].
- **`Spring Boot 2`** - And finally, all these technologies, will be stitched together with [Spring Boot 2][12] development workflow.





{{< articlead >}}

## Model-View-Controller

A quick refresher on [MVC or Model View Controller][13] design pattern. It is commonly used for developing user interfaces that divides the related program logic into three interconnected elements - **Model**, **View** and **Controller**. This is done to separate internal representations of information from the ways information is presented to and accepted from the user.

**`Model`** is the central component of the pattern. It is the application's dynamic data structure, independent of the user interface. It directly manages the data, logic and rules of the application.

**`View`** comprises of any representation of information such as a text, chart, diagram or table. Multiple views of the same information are possible, such as a bar chart for management and a tabular view for accountants.

**`Controller`** acts as the driver here and accepts input and converts it to commands for the model or view.

A typical request flow in MVC pattern looks like this.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/05/model-view-controller-normal.png" alt="Model View Controller (Normal)" />
</div>
{{< /rawhtml >}}

User issues a request to see a view. This request is received by the controller, and it fetches/manipulates the model to fulfill the user request. Controller then forwards the request to view which uses the updated model to generate the view that user requested and show it to the user.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/05/model-view-controller-redirect.png" alt="Model View Controller (Redirect)" />
</div>
{{< /rawhtml >}}

For some user requests, controller doesn't need any model and in these cases, model simply redirects the user request to the appropriate view, which shows user the desired information.



## Application Structure

Let’s look at how this application is structured with these technologies.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/05/spring-boot-crud-operation-application-structure.png" alt="Spring Boot CRUD Operation Application Structure" />
</div>
{{< /rawhtml >}}

We have our client which will be a **Browser** here, and we have our **Undertow** web server, which hosts our application. We have our **H2 database** that stores all the information. All interactions with the database are handled by **Hibernate** layer. These data interactions in Hibernate are requested by the **JPA layer** and **Service layer** in front it.

The **Controller** will handle the requests from the **Client/Browser** and after fetching data with the help of the service layer, pass it onto the **Thymeleaf layer** which renders the final response for client.

In front, facing the client, we have our **Spring Security layer** which secures and controls the request flow to the internal layers of the application.

Here is how the request will flow within these layers.

The **Client/Browser** will initiate a request which will be validated by the **Spring Security layer**. After successful validation, it passes the request to the **Controller**, which makes the call to **Service layer** and fetches the data from the database. All the subsequent layers(**JPA** and **Hibernate**) make appropriate requests to finally fetch the data from **H2 database** and pass them back, all they way back to the **Controller**. **Controller** then selects the **Thymeleaf** view as per the request and passes the model data. **Thymeleaf** response is rendered to generate the final web page with the model data passed to it and finally, the response is sent back to the **Client/Browser**, through the **Spring Security layer**.

Now, that we have a good idea on the application we are building, and how it is structured, let’s Build it with Spring Boot.





{{< articlead >}}

## Creating the Scratch Project

Let's head over to [Spring Initializer][14] to create the scratch project for the application. You will see a lot of options here, and it can be overwhelming, but don't worry, we will keep it simple.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/05/spring-starter.png" alt="Spring Starter" />
</div>
{{< /rawhtml >}}

Here are things to chose.

- **Project** - We will chose Maven project since that is what we will be using and I am more comfortable with Maven.
- **Language** - We will chose `Java` here.
- **Spring Boot** - We can keep the default selected value. At the time of writing, latest Spring Boot version is **2.5.0**.
- **Project Metadata** - This is the section that we will provide for application co-ordinates
  - **Group** - `io.codejournal.springboot`
  - **Artifact** - `mvc-jpa-thymeleaf`
  - **Name** - Gets pre-filled as **Artifact**. Keep it as it is
  - **Description** - Keep it as it is
  - **Package name** - Keep it as it is
  - **Packaging** - Default selection is `Jar`. Keep it as it is
  - **Java** - Default selection is `11` for Java 11. Keep it as it is.

Once done, click on `GENERATE CTL + ⏎` to generate the Maven Project in a zip file and it will be downloaded.

{{< rawhtml >}}
<div class="notification">
    We didn't add any dependencies here since we will be adding them when we will need them in our project. That will give you more context as to why the dependencies were added and which part of code needs them.
</div>
{{< /rawhtml >}}

Extract the contents of the Project and import the Maven Project in the IDE of your choice.



## Project

Taking a look at the project structure, it will look like this.

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── io
    │   │       └── codejournal
    │   │           └── springboot
    │   │               └── mvcjpathymeleaf
    │   │                   └── MvcJpaThymeleafApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── io
                └── codejournal
                    └── springboot
                        └── mvcjpathymeleaf
                            └── MvcJpaThymeleafApplicationTests.java
</pre>
{{< /rawhtml >}}

We have our main class - `MvcJpaThymeleafApplication.java` which runs the application, and there is a corresponding Spring Boot Test class - `MvcJpaThymeleafApplicationTests.java`. An empty `application.properties` to add our configurations for the application. There is Maven Wrapper `mvnw`(for linux and mac) and `mvnw.cmd`(for Windows). A `HELP.md` that has documentation links for Maven and Spring Boot.

And then we have our `pom.xml`. Lets look at the contents of that.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <groupId>io.codejournal.springboot</groupId>
  <artifactId>mvc-jpa-thymeleaf</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>mvc-jpa-thymeleaf</name>
  <description>Demo project for Spring Boot</description>

  <properties>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

Nothing too complicated here as well. Our project has parent pointing to `spring-boot-starter-parent` so that we import Spring Boot related configurations and properties into our project. We have our project co-ordinates as they were put in the [Creating the Scratch Project][15] section. Properties section defines the `Java version(11)` for the project. We have two dependencies - `spring-boot-starter` which starts a Spring Boot application, and `spring-boot-starter-test` to run our tests with Spring awareness. And then finally, `spring-boot-maven-plugin` is there to facilitate the packaging and running of the application when run via the Maven command line.

Ok. Let's run the application now - `mvn spring-boot:run`

{{< rawhtml >}}
<pre class="code-output">
-> mvn spring-boot:run
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------< io.codejournal.springboot:mvc-jpa-thymeleaf >-------------
[INFO] Building mvc-jpa-thymeleaf 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[logs removed to reduce verbosity]
...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)

2021-05-30 21:45:10.696  INFO 22765 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : Starting MvcJpaThymeleafApplication using Java 11.0.11 on codejournal-pc with PID 22765 (/home/codejournal/workspace/mvc-jpa-thymeleaf/target/classes started by codejournal in /home/codejournal/workspace/mvc-jpa-thymeleaf)
2021-05-30 21:45:10.698  INFO 22765 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : No active profile set, falling back to default profiles: default
2021-05-30 21:45:11.123  INFO 22765 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : Started MvcJpaThymeleafApplication in 0.759 seconds (JVM running for 1.035)
2021-05-30 21:45:11.125  INFO 22765 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT
2021-05-30 21:45:11.126  INFO 22765 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  17.892 s
[INFO] Finished at: 2021-05-30T21:45:11+05:30
[INFO] ------------------------------------------------------------------------

</pre>
{{< /rawhtml >}}

So, the application runs and then shut down.





{{< articlead >}}

## Adding the Web Server

To add Web capabilities to your application, Spring Boot provides a auto-configuration in the form of [`spring-boot-starter-web`][16]. Adding this dependency, adds `spring-web`, `spring-webmvc` spring modulus, along with some others. It also adds the Web Server to the application which is [`Apache Tomcat`][17] by default. However, as mentioned earlier, we will instead be using [`Undertow`][3] as our Web Server, so we need to exclude [`Apache Tomcat`][17], and add the starter for undertow. Here is how we will configure this.

Let's add this dependency.

```xml
<!-- Add the Spring Boot Start Web -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <!-- Exclude the Tomcat since we will add Undertow -->
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<!-- Add Undertow -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

And now, if you run the application(`mvn spring-boot:run`), you will see that now the application runs, and since it starts the [`Undertow`][3] server, the application doesn't shut down right away. You will need to press `Ctl + C` to signal stop to the application which stops the server, and application terminates.

{{< rawhtml >}}
<pre class="code-output">
-> mvn spring-boot:run
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------< io.codejournal.springboot:mvc-jpa-thymeleaf >-------------
[INFO] Building mvc-jpa-thymeleaf 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[logs removed to reduce verbosity]
...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)

2021-05-30 22:05:33.552  INFO 23953 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : Starting MvcJpaThymeleafApplication using Java 11.0.11 on codejournal-pc with PID 23953 (/home/codejournal/workspace/mvc-jpa-thymeleaf/target/classes started by codejournal in /home/codejournal/workspace/mvc-jpa-thymeleaf)
2021-05-30 22:05:33.555  INFO 23953 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : No active profile set, falling back to default profiles: default
2021-05-30 22:05:34.488  WARN 23953 --- [           main] io.undertow.websockets.jsr               : UT026010: Buffer pool was not set on WebSocketDeploymentInfo, the default pool will be used
2021-05-30 22:05:34.506  INFO 23953 --- [           main] io.undertow.servlet                      : Initializing Spring embedded WebApplicationContext
2021-05-30 22:05:34.506  INFO 23953 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 883 ms
2021-05-30 22:05:34.839  INFO 23953 --- [           main] io.undertow                              : starting server: Undertow - 2.2.7.Final
2021-05-30 22:05:34.844  INFO 23953 --- [           main] org.xnio                                 : XNIO version 3.8.0.Final
2021-05-30 22:05:34.850  INFO 23953 --- [           main] org.xnio.nio                             : XNIO NIO Implementation Version 3.8.0.Final
2021-05-30 22:05:34.946  INFO 23953 --- [           main] org.jboss.threads                        : JBoss Threads version 3.1.0.Final
2021-05-30 22:05:34.984  INFO 23953 --- [           main] o.s.b.w.e.undertow.UndertowWebServer     : Undertow started on port(s) 8080 (http)
2021-05-30 22:05:34.996  INFO 23953 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : Started MvcJpaThymeleafApplication in 1.958 seconds (JVM running for 2.454)
2021-05-30 22:05:34.997  INFO 23953 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT
2021-05-30 22:05:34.998  INFO 23953 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
^C[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  21.519 s
[INFO] Finished at: 2021-05-30T22:05:51+05:30
[INFO] ------------------------------------------------------------------------

</pre>
{{< /rawhtml >}}

So, our Web Server is set and our application is running. And that concludes our [Part 1][19] of this series.

In [Part 2][20], we will setup the database and configure access to database via Java Persistence API(JPA).

Here is the [Github project for Part 1][18] for your reference.






  [1]: https://spring.io/projects/spring-boot
  [2]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
  [3]: https://undertow.io
  [4]: https://spring.io/projects/spring-framework
  [5]: https://h2database.com/html/main.html
  [6]: https://www.oracle.com/java/technologies/persistence-jsp.html
  [7]: https://hibernate.org/orm/
  [8]: https://www.thymeleaf.org
  [9]: https://getbootstrap.com/
 [10]: https://spring.io/projects/spring-security
 [11]: https://junit.org/junit5/
 [12]: https://spring.io/projects/spring-boot
 [13]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
 [14]: https://start.spring.io/
 [15]: #creating-the-scratch-project
 [16]: https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-starters/spring-boot-starter-web/build.gradle
 [17]: https://tomcat.apache.org/
 [18]: https://github.com/the-code-journal/build-it-with-spring-boot/tree/main/01-mvc-jpa-thymeleaf/part01
 [19]: /2021/05/build-it-with-spring-boot-crud-operations-part-1/
 [20]: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
 [21]: /2021/06/build-it-with-spring-boot-crud-operations-part-3/
 [22]: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
 [23]: /2021/06/build-it-with-spring-boot-crud-operations-part-5/
