---
date: 2021-06-02
linktitle: Build it with Spring Boot - CRUD Operations - Part 2
next: /2020/08/install-apache-maven-on-linux/
prev: /2021/05/build-it-with-spring-boot-crud-operations-part-1/
title: Build it with Spring Boot - CRUD Operations - Part 2
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

- [**Part 1**][3] - Introduction to the Application and Bootstrapping the application, and configure the Web Server.
- [**Part 2 (this article)**][33] - Configure the Database, create JPA Entities and JPA Repositories, Service classes and Junit tests for these
- [**Part 3**][34] - Building the UI pages using Thymeleaf and Bootstrap
- [**Part 4**][20] - Making the application work with UI pages, with data coming from Database
- [**Part 5**][35] - Adding Login/Logout using Spring Security and closure





## Recap from Part 1

In [Part 1][3] of the article, we had a introduction to the application that we are building and how it is structured. We bootstrapped our application which ran with Undertow server.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operations-part2.png" alt="Build it with Spring Boot - CRUD Operation Application - Part 2" />
</div>
{{< /rawhtml >}}

In this part, we will configure the database, hibernate and JPA layers(highlighted block in pink). We will create the Entity/Model classes that will be used in the application.

Below is the video for the same that guides you for this part.

{{< yt "https://www.youtube.com/embed/PeSei1DvRss" >}}





## Model Classes and Usecases

A quick look at the [class diagram][4] of the [entities][5] of our application.

{{< rawhtml >}}
<div class="image">
    <img src="https://yuml.me/diagram/scruffy/class/%5BUser%7C-username:%20string;-password:string%7C+getters();+setters()%5D,%20%5BStudent%7C-id:%20UUID;-firstName:%20string;-lastName:%20string%7C+getters();+setters()%5D" alt="Build it with Spring Boot - CRUD Operation - Class Diagram" />
</div>
{{< /rawhtml >}}

Our primary `Entity` in the application is `Student`, which has `id, firstname` and `lastname` properties. Another supporting `Entity` is `User` which will be used to login to the application. It has `username` and `password` as property.

Taking a look at the [Use Cases][6] for our application, we have following use cases.

{{< rawhtml >}}
<div class="image">
    <img src="https://yuml.me/diagram/scruffy/usecase/%5BUser%5D-(Login),%20%5BUser%5D-(List%20Students),%20%5BUser%5D-(View%20a%20Student),%20%5BUser%5D-(Add%20new%20student),%20%5BUser%5D-(Edit%20existing%20student),%20%5BUser%5D-(Delete%20student),%20%5BUser%5D-(Logout)" alt="Build it with Spring Boot - CRUD Operation - Use Case Diagram" />
</div>
{{< /rawhtml >}}

1. Login
2. List Students
3. View a Student
4. Add new Student
5. Edit existing Student
6. Delete Student
7. Logout





{{< articlead >}}

## Configure Database(H2)

Let's configure our database - [H2][7]. Head over to [Maven Repository][8] and search for artifact for `h2`. We are looking for dependency from `com.h2database.h2` and pick the latest version(`1.4.200` as of writing this article). Copy the dependency and add it to our project `pom.xml`.

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
You will notice that, we have removed the `<scope>` and `<version>` tags from above. We removed `<scope>` because [H2][7] is mostly used for in-memory testing during test phase. Since, we are using it as our primary database when we run the application, we need it to be available in [compile scope][9].

We also removed the `<version>` because our project has `spring-boot-starter-parent` as parent and it already manages the version, so we don't need to explicitly provide the version. The correct compatible version will be used from `spring-boot-starter-parent`.

{{< rawhtml >}}
<div class="notification">
    So, database is available in our application, but how do we setup the Datasource and Entity Manager?
</div>
{{< /rawhtml >}}

Spring Boot comes to the rescue. Instead of we configuring all of this, we can make use of [Spring Boot Autoconfiguration][10] to create these objects. It takes the burden off our shoulders by creating dependencies in consistent and ideal way, and we can continue to focus on building our application.

For our purpose, we need the autoconfiguration, along with other dependencies from JPA - `spring-boot-data-jpa`.

Let's add the below dependency to our project. You can, also get it also from [Maven Repository][8].

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

Again, we removed the `<version>` tag, since the managed version will be provided from the parent pom - `spring-boot-starter-parent`. It also fetches `hibernate-core` along with it which it uses internally to make calls to the database using the datasource.

And that single dependency will take care of everything...well almost. 

The database connection details still need to be configured, and these need to be configured in `application.properties` or `application.yaml`. Spring Boot supports both, but I believe `yaml` provides a much better structure to the configurations, and hence we will use the same.

We need to first rename our `application.properties` to `application.yaml`, so lets do that.

```bash
mv src/main/resources/application.properties src/main/resources/application.yaml
```

And then let's add the below configuration in `application.yaml`

```yaml
spring:
  application:
    name: build-it-with-spring-boot-mvc-jpa-thymeleaf
  datasource:
    url: jdbc:h2:mem:student_db
    driver-class-name: org.h2.Driver
    username: sa
    password: password
```

A full list of **Spring Boot Application Properties** is [available here][12] for reference. Let's discuss the things that we added above.

- `spring.application.name` - Name of our Application as identified by Spring Boot
- `spring.datasource` - Properties that allows to configure a DataSource
  - `spring.datasource.url` - JDBC Url for the Database. We are using in-memory mode(database gets created when application runs and then destroyed with the application shuts down). This is put as per [H2 Cheatsheet documentation][11]
  - `spring.datasource.driver-class-name` - Driver class as per [H2 Cheatsheet documentation][11]
  - `spring.datasource.username` - Username to connect to the Database
  - `spring.datasource.password` - Password to connect to the Database

And with that, we are connected to the database(**in theory at least**).

{{< rawhtml >}}
<div class="notification">
    There are ways to verify the connection to the database using points at the end of this article, but since this process is so seemless, this verification is not required in most cases. But, if you feel this connection verification is important, go ahead and use the points to do that.
</div>
{{< /rawhtml >}}







{{< articlead >}}

## JPA Entities

Let's create our JPA Entities, which essentially marks the Java class to be managed by the Entity Manager. Entity Manager performs the interaction with the database for the Entities that it is managing. To configure a JPA entity, we get annotations from `javax.persistence.*` package.

- `@Entity` - Marks the Java class as an Entity
- `@Table` - Allows to map the Entity to its Database table.
- `@Id` - Defines the entity identifier. All entities must have an `@Id` field, and they map to Primary Key columns of the table in most scenarios.
- `@GeneratedValue` - For fields like Primary Key, an unique value needs to be provided while inserting new records. This annotation allows to generate this value automatically
- `@Column` - To specify the mapping for Entity attribute and database table column

All JPA Entities must have their `equals` and `hashCode` methods to be overriden from the `Object` class so that when the objects are read/written to the database, it can be done in consistent and non-clashing manner.

To help you with these `equals` and `hashCode` methods, you can use the IDE support to generate these for you. However, I always feel these are redundant and adds to class code and cognitive load. Luckily, I am not the only one who has ever felt it this way and [Project Lombok][13] was created for this reason. Let's add the dependency for lombok in our `pom.xml`

```xml
<!-- Miscellaneous -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

{{< rawhtml >}}
<div class="notification">
    The <code>&lt;scope>provided&lt;/scope></code> is added since we don't want this dependency to be packaged along with our final application jar.
</div>
{{< /rawhtml >}}

 [Lombok][13] gives a lot of annotations, and below are some that are particular to our application that we can add to our classes.

- `@Getter` - Adds the getter methods for all fields
- `@Setter` - Adds the setter methods for all fields
- `@EqualsAndHashCode` - Adds the `equals` and `hashCode` methods for the class
- `@ToString` - Adds a `toString()` method which prints the values of the fields
- `@Data` - Combines the functionality of `@Getter`, `@Setter`, `@EqualsAndHashCode` and `@ToString`
- `@NoArgsConstructor` - Adds the public no-argument constructor of the Class
- `@AllArgsConstructor` - Adds the public all-arguments constructor of the Class

There are a huge range of annotations and configuration options from [lombok][13], and I would suggest, you take a look at the [documetation here][14] which explains the uses for quite a lot of them.

{{< rawhtml >}}
<div class="notification">
    While adding the dependency in <code>pom.xml</code> will work while compiling the application through Maven, on your IDE, it will still show compilation errors. You need to add the support for lombok for your IDE by installing lombok for your IDE. You can follow the installation <a href="https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok#installation">tutorial here</a> to add lombok support from within your IDE.
</div>
{{< /rawhtml >}}

Let's create two classes - `Student.java` and `User.java`, add our fields from our [class diagram][15] and configure them as JPA entities.

```java
package io.codejournal.springboot.mvcjpathymeleaf.entity;

import java.util.UUID;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "student")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {

    @Id
    @GeneratedValue(generator = "UUID")
    @Column(name = "s_id")
    private UUID id;

    @Column(name = "s_first_name")
    private String firstName;

    @Column(name = "s_last_name")
    private String lastName;
}
```

```java
package io.codejournal.springboot.mvcjpathymeleaf.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "user")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @Id
    @Column(name = "u_username")
    private String username;

    @Column(name = "u_password")
    private String password;
}
```

There is a lot to get your head around, so let's take it one by one.

- `@Entity` annotation on both the classes marks each of the classes as a JPA entities
- `@Table` annotation links `Student` class to `student` table, while `User` class links the entity to `user` table
- `@Data` annotation adds auto-generated methods for **getters, setters, equals, hashCode** and **toString**
- `@NoArgsConstructor` annotations adds a public no-argument constructor for each of the Entity classes
- `@AllArgsConstructor` annotations adds a public all-argument constructor for each of the Entity classes
- `@Column` annotation links each field to their corresponding table columns in the database
- `@Id` annotation denotes the primary key field for each table(`Student.id` and `User.username`)
- `@GeneratedValue(generator = "UUID")` annotation specifies that `Student.id` field has to be generated automatically as `UUID` while inserting new rows in the database, if not present already.

And there we have our JPA entities for our projects.





{{< articlead >}}

## Spring Data Repositories

Let's use some more of Spring magic. [Spring Data Repositories][16] allows us to make queries to the database, with almost zero boilerplate code. The central interface is the [`Repository`][36] interface which takes the Domain class and the Id class as generic arguments, and spring automatically makes query for you. To make life even more simpler, Spring automatically provides you with interface [`CrudRepository`][37] that has methods to perform basic CRUD operations in various scenarios. There is another interface [`PagingAndSortingRepository`][38], which apart from CRUD operations, provides you with Pagination and Soring support for the select queries.

And finally, to manage JPA entities via Spring Data Repositories, we get [`JpaRepository`][39] which provides the same support as [`PagingAndSortingRepository`][38], except that it also works for JPA entities in specific, which is what we need.

Let's create a package - `io.codejournal.springboot.mvcjpathymeleaf.repository` add our `StudentRepository` and `UserRepository` interfaces.

```java
package io.codejournal.springboot.mvcjpathymeleaf.repository;

import java.util.UUID;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import io.codejournal.springboot.mvcjpathymeleaf.entity.Student;

@Repository
public interface StudentRepository extends JpaRepository<Student, UUID> {
}
```

```java
package io.codejournal.springboot.mvcjpathymeleaf.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import io.codejournal.springboot.mvcjpathymeleaf.entity.User;

@Repository
public interface UserRepository extends JpaRepository<User, String> {
}
```

Nothing unusual here. We just created our interfaces which extend the `JpaRepository`. Since, the `Repository` interface needs the class to manage, `JpaRepository` too needs the argument Entity class to manage and the Id class for the Entity. `Student` has `UUID` as its id(`student.id`), hence `JpaRepository<Student, UUID>`, while `User` has `String` as its id(`user.username`), hence `JpaRepository<User, String>`.

We also added the marker annotation [`@Repository`][17] to both our interfaces, so that Spring recognized these interfaces as Repositories and treats them the same, while building objects for these.





{{< articlead >}}

## Service Classes

Moving on to our Service layer which represents our business logic layer. Taking a look at our [Use Case][15] diagram, we can split our use cases for each of our entities.

**Student** entity comprises of:

1. List students
2. View a student
3. Add new student
4. Edit/Update existing student
5. Delete student

While **User** entity has use cases:

1. Login
2. Logout

For each of our use cases for Student, let's have a method in our `StudentService.java`. Also, add a new student and edit/update a student is pretty much the same thing in our case. If the student id is missing, then we will add the student information, while if the student id is present we will update the student information. So, we can combime these operations as `save` in our service class.

To perform these operations with the database, we will need the `StudentRepository` that we created in the previous section. Since, this is a dependency for our Service class, we will inject it in our class. You can use any form of [dependency injection][18] provided by Spring, but I find **Constructor Injection** to be the cleanest approach. It marks the dependency clearly and upfront, and there is no chance of missing the dependency.

Finally, to keep it simple, we won't have any logic as such in our methods, and they will merely be delegates to our repository calling appropriate methods.

Below is the `StudentService.java`.

```java
package io.codejournal.springboot.mvcjpathymeleaf.service;

import java.util.Optional;
import java.util.UUID;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Service;

import io.codejournal.springboot.mvcjpathymeleaf.entity.Student;
import io.codejournal.springboot.mvcjpathymeleaf.repository.StudentRepository;

@Service
public class StudentService {

    private final StudentRepository repository;

    @Autowired
    public StudentService(final StudentRepository repository) {
        this.repository = repository;
    }

    public Page<Student> getStudents(final int pageNumber, final int size) {
        return repository.findAll(PageRequest.of(pageNumber, size));
    }

    public Optional<Student> getStudent(final UUID id) {
        return repository.findById(id);
    }

    public Student save(final Student student) {
        return repository.save(student);
    }

    public void delete(final UUID id) {
        repository.deleteById(id);
    }
}
```

- We indicate the **Constructor Dependency Injection** by marking the constructor as `@Autowired`. When spring calls the constructor, it locates the Bean for `StudentRepository` and passes that, which we save in our field `repository` in the constructor.
- `getStudents` method returns a `Page` of student records. This page is determined by the `pageNumber` and `size` arguments of the method. The call for `repository.findAll` is provided by [`PagingAndSortingRepository`][38] and is used to make the call to the database and get the page requested. The `PageRequest` implements `Pageable` interface which is what is used to make appropriate SQL query to get the requested records.
- `getStudent` fetches a single student record by its id. The call made is `repository.findById(id)` provided by the [`CrudRepository`][37]. It returns and `Optional` object, i.e. if the record is found, `Optional.get()` would return the student object otherwise it will return null.
- `save` takes a Student record and saves it in the database. If the `student.id` already exists in the database, it will be updated, otherwise a new record is created. Again, `repository.save(student)` is coming from [`CrudRepository`][37].
- `delete` deletes the record in the databse if it exists. The actual call - `repository.deleteById(id)`, is provided by [`CrudRepository`][37], and it deletes the record if it exists and the corresponding object is returned. Otherwise, it returns null. However, we don't intend to use the returned value, that is why our service method returns a `void`.
- Finally, the class is annotated with [`@Service`][19] annotation, so that Spring can treat it that way.

And that's pretty much it for `StudentService`. It's simple and clean.

For `UserService`, we will be needing that in [Part 4][20], and we will add details then. If you want, you can create an empty `UserService` class with `UserRepository` injected via the constructor. Here is how it will look like.

```java
package io.codejournal.springboot.mvcjpathymeleaf.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import io.codejournal.springboot.mvcjpathymeleaf.repository.UserRepository;

@Service
public class UserService {

    private final UserRepository repository;

    @Autowired
    public UserService(final UserRepository repository) {
        this.repository = repository;
    }

    // Methods to be added in Part 4
}
```





{{< articlead >}}

## Test Setup

We already have `spring-boot-starter-test` when we initially created the project from [Spring Initalizer][22]. This also brings [JUnit Jupiter(Junit 5)][21] dependency along with it, so we don't need to specify it explicitly(unless you want to use a newer version of Junit 5). We will add two additional dependencies before we start writing our Junit 5 tests.

### AssertJ

While [Junit 5][21] provides assertions, I feel [assertj's][23] assertions are much more readable and it allows chaining of assertions one after another on the same object. For example, take a look at an example assertion chain on a list. It is much more intuitive, readable and concise.

```java
assertThat(fellowshipOfTheRing).hasSize(9)
                               .contains(frodo, sam)
                               .doesNotContain(sauron);
```

### Mockito

We will use [Mockito][24] as our mocking framework to mock out our dependencies. Its simple, straight-forward and easy to get up and running.

Let's add the corresponding dependencies in our `pom.xml`

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.10.19</version>
    <scope>test</scope>
</dependency>
```

Yet again, we didn't have to specify `<version>` for `assertj-core` since the version is managed from `spring-boot-starter-parent`. However, `mockiito-all` is not managed, and hence we need to specify the `<version>`.



## JPA Repository Tests

Let's start with unit tests for JPA Repositories. To facilitates testing for JPA Repositories, Spring provides a dedicated annotation - [`@DataJpaTest`][25]. This allows to inject [`TestEntityManager`][26] which can be used to setup and/or verify data for the operations done via the JPA Repositories.

Below is the `StudentRepositoryTest` that tests the `save` operation of the `StudentRepository` which essentially comes from the [`CrudRepository`][37].

```java
package io.codejournal.springboot.mvcjpathymeleaf.repository;

import static java.util.UUID.randomUUID;
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import io.codejournal.springboot.mvcjpathymeleaf.entity.Student;

@ExtendWith(SpringExtension.class)
@DataJpaTest
public class StudentRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private StudentRepository repository;

    @Test
    public void save_StoresRecord_WhenRecordIsValid() {

        final Student expected = new Student();
        expected.setFirstName(randomUUID().toString());
        expected.setLastName(randomUUID().toString());

        final Student saved = repository.save(expected);

        final Student actual = entityManager.find(Student.class, saved.getId());

        assertThat(actual).isEqualTo(expected);
    }
}
```

Let's understand the things that we have here.

1. `@ExtendWith(SpringExtension.class)` - With this, we tell Junit that the current test needs additional functionality extension from `SpringExtension.class`, which creates the bean objects that are used in the test class.
2. We `@Autowire` the `TestEntityManager` that we will use to verify that the `save` operation actually saves a record that can be queried with this.
3. We `@Autowire` our bean for interface, `StudentRepository` provided by Spring as it would provide when the application runs. We will use `repository.save` to save a record that we will verify with `TestEntityManager`.
4. And then we have our test method - `save_StoresRecord_WhenRecordIsValid`, which uses the `method-then-when` format for naming. In this method, we do the following.
   * We create a `Student` record with `firstName` and `lastName`, and then save it using `repository.save`.
   * The `save` method returns the saved student record which has `id` field populated.
   * We use the returned student id, and query our `TestEntityManager` to locate the `Student` entity with this id. We use `entityManager.find` which returns the actual student record after it was saved.
   * To ensure that everything happened as expected, we compare the actual student object fetched using `entityManager.find` and the original student object that we created, to verify that both of them are same.

A similar test can also be written for `UserRepositoryTest`.

```java
package io.codejournal.springboot.mvcjpathymeleaf.repository;

import static java.util.UUID.randomUUID;
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import io.codejournal.springboot.mvcjpathymeleaf.entity.User;

@ExtendWith(SpringExtension.class)
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository repository;

    @Test
    public void save_StoresRecord_WhenRecordIsValid() {

        final User expected = new User();
        expected.setUsername(randomUUID().toString());
        expected.setPassword(randomUUID().toString());

        final User saved = repository.save(expected);

        final User actual = entityManager.find(User.class, saved.getUsername());

        assertThat(actual).isEqualTo(expected);
    }
}
```

{{< rawhtml >}}
<div class="notification">
    In practice, we should write the tests if we add additional methods to our Repository interfaces, that do not come from <code>CrudRepository, PagingAndSortingRepository</code> and <code>JpaRepository</code>. The example above gives a general idea how to go about writing test for the custom methods, if and when you have them.
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Service Tests

Let's configure the tests for Service classes, in particular - `StudentService`. We will add tests for `UserService` in [Part 4][20], when we configure Spring Security because that is when we will add methods in `UserService`.

Below is the quick setup of the Junit test for `StudentServiceTest`.

```java
import org.springframework.boot.test.mock.mockito.MockBean;

@ExtendWith(SpringExtension.class)
public class StudentServiceTest {

    @MockBean
    private StudentRepository repository;

    private StudentService fixture;

    @BeforeEach
    public void setUp() {
        fixture = new StudentService(repository);
    }

    // Test methods follow here
}
```

1. Again, we add the `@ExtendWith(SpringExtension.class)` to signify that Spring should add additional functionality.
2. [`@MockBean`][27] annotation is provided by Spring to automatically create a mocked implementation using [`Mockito`][24] and is injected on the specified field. Here our dependency is `StudentRepository` for which we request a mocked bean version using [`@MockBean`][27] annotation.
3. `@BeforeEach` annotation is the setup method from Junit 5, that executed before every test method. Within this `setUp` method, we create a new instance `StudentService` class, and set it in our `fixture` field. As you can see, using the [Constructor Depndency Injection][10] mandates us that we provide the instance of `StudentRepository` for the constructor, thereby the chances of missing it during setting it up, are minimal.

The approach to test the methods of `StudentService` is simple.

1. We make calls on the `fixture` object with appropriate parameters, and the value returned is compared against a pre-created expected value.
2. Any calls to `repository` have pre-configured stubs, to allow capturing of values and performing verifications on these.
3. We will use [BDDMockito][28] methods which follow `//given //when //then` approach of tests

Let's take an example for `StudentService.getStudent` test method.

```java
import static org.mockito.BDDMockito.given;
import static org.mockito.BDDMockito.then;

@Test
public void getStudent_ReturnsStudent_WhenStudentExist() {

    final UUID id = randomUUID();
    final String firstName = randomUUID().toString();
    final String lastName = randomUUID().toString();

    final Student student = new Student(id, firstName, lastName);

    final Optional<Student> expected = Optional.of(student);

    given(repository.findById(id)).willReturn(expected);

    final Optional<Student> actual = fixture.getStudent(id);

    assertThat(actual).isEqualTo(expected);

    then(repository).should().findById(id);
    then(repository).shouldHaveNoMoreInteractions();
}
```

- We create a Student object with random values. This student object is what we expect to be returned back when we call `getStudent` method.
- The method `getStudent` internally makes a call - `repository.findById(id)` to find the student record in the database. It returns `Optional` with student record when the record is found, other it return `Optional.empty()`.
- We can add expecation on the call - `repository.findById(id)` by using `BDDMockito.given` method. Since, this tests assumes that record exists for the id passed, we set the expectation as - `given(repository.findById(id)).willReturn(Optional.of(student))`
- Then we make the actual call on our fixture object - `fixture.getStudent(id)`, which returns `Optional<Student>` as actual value.
- We then proceed to compare that both our expected value and actual value and verify that both are equal.
- Also, we need to verify that the expectation we set on our mocked repository, are the ones that got used, and that no other additional unnecessay methods are called on the repository. To do this, we use `BDDMockito.then` to verify that `repository.findById(id)` is called once, and then no other method is called. These verifications are done with statements - `then(repository).should().findById(id)` and `then(repository).shouldHaveNoMoreInteractions()`.

Below is the full `StudentServiceTest` which tests all the methods of `StudentService` with all scenarios.

```java
package io.codejournal.springboot.mvcjpathymeleaf.service;

import static java.util.UUID.randomUUID;
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;
import static org.mockito.BDDMockito.then;
import static org.mockito.BDDMockito.willDoNothing;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.PageRequest;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import io.codejournal.springboot.mvcjpathymeleaf.entity.Student;
import io.codejournal.springboot.mvcjpathymeleaf.repository.StudentRepository;

@ExtendWith(SpringExtension.class)
public class StudentServiceTest {

    @MockBean
    private StudentRepository repository;

    private StudentService fixture;

    @BeforeEach
    public void setUp() {
        fixture = new StudentService(repository);
    }

    @Test
    public void getStudents_ReturnsStudents_WhenStudentsExists() {

        final int pageNumber = (int) (Math.random() * 100);
        final int pageSize = (int) (Math.random() * 100);

        final int totalRecords = (int) (Math.random() * 100);

        final Student student1 = new Student(randomUUID(), randomUUID().toString(), randomUUID().toString());
        final Student student2 = new Student(randomUUID(), randomUUID().toString(), randomUUID().toString());
        final Student student3 = new Student(randomUUID(), randomUUID().toString(), randomUUID().toString());

        final List<Student> students = Arrays.asList(student1, student2, student3);

        final PageRequest page = PageRequest.of(pageNumber, pageSize);

        final Page<Student> expected = new PageImpl<>(students, page, totalRecords);

        given(repository.findAll(page)).willReturn(expected);

        final Page<Student> actual = fixture.getStudents(pageNumber, pageSize);

        assertThat(actual).isEqualTo(expected);
        assertThat(actual.getContent()).hasSameElementsAs(students);
        assertThat(actual.getPageable()).isEqualTo(page);

        then(repository).should().findAll(page);
        then(repository).shouldHaveNoMoreInteractions();
    }

    @Test
    public void getStudent_ReturnsStudent_WhenStudentExist() {

        final UUID id = randomUUID();

        final Student student = new Student(randomUUID(), randomUUID().toString(), randomUUID().toString());

        final Optional<Student> expected = Optional.of(student);

        given(repository.findById(id)).willReturn(expected);

        final Optional<Student> actual = fixture.getStudent(id);

        assertThat(actual).isEqualTo(expected);

        then(repository).should().findById(id);
        then(repository).shouldHaveNoMoreInteractions();
    }

    @Test
    public void getStudent_ReturnsStudent_WhenStudentDoesNotExist() {

        final UUID id = randomUUID();

        final Optional<Student> expected = Optional.empty();

        given(repository.findById(id)).willReturn(expected);

        final Optional<Student> actual = fixture.getStudent(id);

        assertThat(actual).isEqualTo(expected);

        then(repository).should().findById(id);
        then(repository).shouldHaveNoMoreInteractions();
    }

    @Test
    public void save_ReturnSaved_WhenStudentRecordIsCreated() {

        final UUID id = randomUUID();

        final Student expected = new Student();
        expected.setFirstName(randomUUID().toString());
        expected.setLastName(randomUUID().toString());

        given(repository.save(expected)).willAnswer(invocation -> {

            final Student toSave = invocation.getArgument(0);

            toSave.setId(id);

            return toSave;
        });

        final Student actual = fixture.save(expected);

        assertThat(actual).isEqualTo(expected);

        then(repository).should().save(expected);
        then(repository).shouldHaveNoMoreInteractions();
    }

    @Test
    public void delete_DeletesStudent_WhenStudentExists() {

        final UUID id = randomUUID();

        willDoNothing().given(repository).deleteById(id);

        fixture.delete(id);

        then(repository).should().deleteById(id);
        then(repository).shouldHaveNoMoreInteractions();
    }
}
```





{{< articlead >}}

## Running the Application

The application was pretty much ready when we completed our Service classes, but without further ado, let's run the application - `mvn spring-boot:run`

{{< rawhtml >}}
<pre class="code-output">
INFO 18901 --- [ main] i.c.s.m.MvcJpaThymeleafApplication       : Starting MvcJpaThymeleafApplication using Java 11.0.11 on codejournal-pc with PID 18901 (/home/codejournal/sts-workspace/mvc-jpa-thymeleaf/target/classes started by codejournal in /home/codejournal/sts-workspace/mvc-jpa-thymeleaf)
INFO 18901 --- [ main] i.c.s.m.MvcJpaThymeleafApplication       : No active profile set, falling back to default profiles: default
INFO 18901 --- [ main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
INFO 18901 --- [ main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 46 ms. Found 2 JPA repository interfaces.
WARN 18901 --- [ main] io.undertow.websockets.jsr               : UT026010: Buffer pool was not set on WebSocketDeploymentInfo, the default pool will be used
INFO 18901 --- [ main] io.undertow.servlet                      : Initializing Spring embedded WebApplicationContext
INFO 18901 --- [ main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 898 ms
INFO 18901 --- [ main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
INFO 18901 --- [ main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
INFO 18901 --- [ main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
INFO 18901 --- [ main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.31.Final
INFO 18901 --- [ main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
INFO 18901 --- [ main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
INFO 18901 --- [ main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
INFO 18901 --- [ main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
WARN 18901 --- [ main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
INFO 18901 --- [ main] io.undertow                              : starting server: Undertow - 2.2.7.Final
INFO 18901 --- [ main] org.xnio                                 : XNIO version 3.8.0.Final
INFO 18901 --- [ main] org.xnio.nio                             : XNIO NIO Implementation Version 3.8.0.Final
INFO 18901 --- [ main] org.jboss.threads                        : JBoss Threads version 3.1.0.Final
INFO 18901 --- [ main] o.s.b.w.e.undertow.UndertowWebServer     : Undertow started on port(s) 8080 (http)
INFO 18901 --- [ main] i.c.s.m.MvcJpaThymeleafApplication       : Started MvcJpaThymeleafApplication in 2.596 seconds (JVM running for 2.872)
INFO 18901 --- [ main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT
INFO 18901 --- [ main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
</pre>
{{< /rawhtml >}}

The application runs and there are no errors, and we see some logs which indidate that the database interaction is working. But, we don't have a clear way to know if our integration from JPA Repositories to the Database is working correctly. Also, there is no endpoint as such at the moment, which we configured that we can call to see some results.

{{< rawhtml >}}
<div class="notification">
    So, how can we verify that things are in order?
</div>
{{< /rawhtml >}}

### Show SQL Logs

One of the quick things that we can do is to enable the SQL logs from Spring itself. That way, we can confirm that database operations happening via the Jpa Entity Manager.

This can be enabled using the [`spring.jpa.show-sql`][29] configuration. We will configure this in our `application.yaml` as below.

```yaml
spring:
  ...
  datasource:
    ...
  jpa:
    show-sql: true
```

And when you run the application now, we have the usual logs along with statements like this.

{{< rawhtml >}}
<pre class="code-output">
...
INFO 19729 --- [ main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect

Hibernate: drop table if exists student CASCADE 
Hibernate: drop table if exists user CASCADE 
Hibernate: create table student (s_id binary not null, s_first_name varchar(255), s_last_name varchar(255), primary key (s_id))
Hibernate: create table user (u_username varchar(255) not null, u_password varchar(255), primary key (u_username))

INFO 19729 --- [ main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
...
</pre>
{{< /rawhtml >}}

Aha. So, Hibernate is dropping and creating these tables on startup. Nice. That verifies that our database connection is working. But, what about our JPA Repositories. There is still no clear indication.

{{< rawhtml >}}
<div class="notification">
    While, its good to see SQL activity, it might not be a good option for Production systems. So, use this property judiciously.
</div>
{{< /rawhtml >}}



### ApplicationRunner Spring class

To be able to verify if our JPA Repositories can communication to the database correctly, we need to make the call method of the repositories that we created. There is no way getting around that. And Spring Boot provides [`ApplicationRunner`][30] exactly for this purpose.

As soon as the application starts, we can run some code(insert some student rows) and verify that the corresponding activity is visible in logs. `ApplicationRunner` interface has just a single `run` method, so we can use Java 8 lambda here.

We will add this piece of code in our main application class, `MvcJpaThymeleafApplication`. We will create a couple of student records, and insert them using the `StudentRepository`. Now, we don't have that yet in our `MvcJpaThymeleafApplication`, so we can `@Autowire` it. Here is the final code.

```java
package io.codejournal.springboot.mvcjpathymeleaf;

import static java.util.UUID.randomUUID;

import java.util.Arrays;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import io.codejournal.springboot.mvcjpathymeleaf.entity.Student;
import io.codejournal.springboot.mvcjpathymeleaf.repository.StudentRepository;

@SpringBootApplication
public class MvcJpaThymeleafApplication {

    @Autowired
    private StudentRepository studentRepository;

    public static void main(final String[] args) {
        SpringApplication.run(MvcJpaThymeleafApplication.class, args);
    }

    @Bean
    public ApplicationRunner initializeStudents() {

        final Student defaultStudent1 = new Student(randomUUID(), "John", "Doe");
        final Student defaultStudent2 = new Student(randomUUID(), "Linda", "Rostam");

        return args -> studentRepository.saveAll(Arrays.asList(defaultStudent1, defaultStudent2));
    }
}
```

And let's run our application now, and we will see more SQL activity as below.

{{< rawhtml >}}
<pre class="code-output">
...
INFO 20741 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect

Hibernate: drop table if exists student CASCADE 
Hibernate: drop table if exists user CASCADE 
Hibernate: create table student (s_id binary not null, s_first_name varchar(255), s_last_name varchar(255), primary key (s_id))
Hibernate: create table user (u_username varchar(255) not null, u_password varchar(255), primary key (u_username))

INFO 20741 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
...
...
INFO 20741 --- [           main] i.c.s.m.MvcJpaThymeleafApplication       : Started MvcJpaThymeleafApplication in 2.648 seconds (JVM running for 2.964)
INFO 20741 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT

Hibernate: select student0_.s_id as s_id1_0_0_, student0_.s_first_name as s_first_2_0_0_, student0_.s_last_name as s_last_n3_0_0_ from student student0_ where student0_.s_id=?
Hibernate: select student0_.s_id as s_id1_0_0_, student0_.s_first_name as s_first_2_0_0_, student0_.s_last_name as s_last_n3_0_0_ from student student0_ where student0_.s_id=?
Hibernate: insert into student (s_first_name, s_last_name, s_id) values (?, ?, ?)
Hibernate: insert into student (s_first_name, s_last_name, s_id) values (?, ?, ?)

INFO 20741 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
</pre>
{{< /rawhtml >}}

We see that There are two `SELECT` queries to verify that the records that are about to be inserted, do not exist already, and then there are two `INSERT` queries that inserts the two student records that we requested to be saved via the `studentRepository.saveAll`.

And that verifies our integration of JPA Repositories with the database.

{{< rawhtml >}}
<div class="notification">
    But, what if you want something more visual and want to query the database yourself?
</div>
{{< /rawhtml >}}




### Spring Boot Devtools

For external databases, like Postgres, or MySQL, there are clients available(cli or gui) to run the queries outside of the application to verify the operations. But, with in-memory database, the database is created when the application starts and gets destroyed once it exits(unless you save the data in a file and reload it next time when application starts).

Spring Boot helps you here as well, and they build a whole dependency for tasks similar to these with `spring-boot-devtools`. It provides various features, but right now we will discuss only couple of them.

1. When you add this dependency, it allows accessing the H2 database via the [`/h2-console`][31] path on the server.
2. Right now, everytime, when you make changes to any of your code files, you need to stop the server and then start again. With `spring-boot-devtools` on your classpath, it automatically detects the changes done to the application files, and then restarts the application on its own. That way, you don't need to worry about restarting it everytime you make code changes. You can continue making code changes, and the Spring Boot will continue to update the application with new changes.

Let's add this dependency in our `pom.xml`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>provided</scope>
</dependency>
```

Yet again, `<scope>provided</scope>` since we don't want this dependency to end up in our final application jar.

And now, when you run your application and then navigate to `http://localhost:8080/h2-console`, you will see the H2 console login window as below.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/h2-console.png" alt="Build it with Spring Boot - H2 Console" />
</div>
{{< /rawhtml >}}

Take a note of the `JDBC Url` and `Password`, as they should match the same as we configured in our `application.yaml`, and once you have provided that, click on `Connect` to connect to the In-Memory Database and see the tables of the database. Once, you are connected, you can view the Tables in the databases as well as run ad-hoc queries against the tables to add/update/verify application operations.

Below is the screenshot that shows the H2 console after logging in.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/h2-console-query.png" alt="Build it with Spring Boot - H2 Console Query" />
</div>
{{< /rawhtml >}}

On the left, you have the database and the tables listed, while on the right side, you can write SQL queries. When you run the SQL Queries, the results are returned in the bottom section.

And that concludes our [Part 2][33] of this series. It was a long one and we did quite a lot of things here.

In [Part 3][34], we will create the UI using Thymeleaf template engine.

Here is the [Github project for Part 2][32] for your reference.





  [1]: https://spring.io/projects/spring-boot
  [2]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
  [3]: /2021/05/build-it-with-spring-boot-crud-operations-part-1/
  [4]: https://en.wikipedia.org/wiki/Class_diagram
  [5]: https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model
  [6]: https://en.wikipedia.org/wiki/Use_case
  [7]: https://h2database.com/html/main.html
  [8]: https://mvnrepository.com/
  [9]: /2020/08/maven-dependency-scopes/
 [10]: https://docs.spring.io/spring-boot/docs/current/reference/html/auto-configuration-classes.html#auto-configuration-classes.core
 [11]: https://h2database.com/html/cheatSheet.html
 [12]: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html
 [13]: https://projectlombok.org
 [14]: https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok
 [15]: #model-classes-and-usecases
 [16]: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories
 [17]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Repository.html
 [18]: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators
 [19]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html
 [20]: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
 [21]: https://junit.org/junit5/
 [22]: https://start.spring.io/
 [23]: https://assertj.github.io/doc/
 [24]: https://site.mockito.org
 [25]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-spring-data-jpa
 [26]: https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure/orm/jpa/TestEntityManager.java
 [27]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.mocking-beans
 [28]: https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/BDDMockito.html
 [29]: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.data.spring.jpa.show-sql
 [30]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.command-line-runner
 [31]: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.data.spring.h2.console.path
 [32]: https://github.com/the-code-journal/build-it-with-spring-boot/tree/main/01-mvc-jpa-thymeleaf/part02
 [33]: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
 [34]: /2021/06/build-it-with-spring-boot-crud-operations-part-3/
 [35]: /2021/06/build-it-with-spring-boot-crud-operations-part-5/
 [36]: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html
 [37]: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html
 [38]: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html
 [39]: https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html