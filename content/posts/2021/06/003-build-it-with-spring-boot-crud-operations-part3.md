---
date: 2021-06-11
linktitle: Build it with Spring Boot - CRUD Operations - Part 3
next: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
prev: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
title: Build it with Spring Boot - CRUD Operations - Part 3
weight: 10
tags: [ "spring-boot", "java", "crud", "applications" ]
categories: [ "Spring Boot", "Java" ]
---


This series builds Java applications using [Spring Boot 2][1], which has become de-facto standard for building applications in Java world. It makes it easy to create stand-alone, production-grade Spring based Applications. It cuts down on a lot of boilerplate code, and its sensible defaults for configurations allows to build applications in super quick time.

The project we are building here is a Spring Boot Web application that demonstrates [CRUD operations][2]. The application allows to manage student information, by a paginated student list, adding a new student, viewing student information, editing them, and deleting them. The application also has Login/Logout feature.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/build-it-with-spring-boot-crud.gif" alt="Build it with Spring Boot - CRUD Operation Application" />
</div>
{{< /rawhtml >}}

Here are all the articles of this series.

- [**Part 1**][3] - Introduction to the Application and Bootstrapping the application, and configure the Web Server.
- [**Part 2**][4] - Configure the Database, create JPA Entities and JPA Repositories, Service classes and Junit tests for these
- [**Part 3 (this article)**][5] - Building the UI pages using Thymeleaf and Bootstrap
- [**Part 4**][6] - Making the application work with UI pages, with data coming from Database
- [**Part 5**][7] - Adding Login/Logout using Spring Security and closure





## Recap from Part 2

A quick look at the [MVC diagram][8]. And in [Part 2][4] of the video, we pretty much completed our **Model** part. We created the service classes that contains methods to fetch, add, update and delete student data.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/model-view-controller-part2.png" alt="Model View Controller - Part 3" />
</div>
{{< /rawhtml >}}

From the application structure, we completed our Service layer, [JPA][9] layer, and setup the connection to the database via [Hibernate][10].

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-application-structure-part2.png" alt="Build it with Spring Boot - CRUD Application Structure - Part 2" />
</div>
{{< /rawhtml >}}

In this video, we will create the UI using [Thymeleaf][11] and [Bootstrap][12].

Below is the video for the same that guides you for this part.

{{< yt "https://www.youtube.com/embed/ZIZ4AVkVL0g" >}}





{{< articlead >}}

## UI Layouts

Our UI will consist of 5 pages.

1. List Students
2. Add Student
3. View Student
4. Edit Student
5. Delete Student

Let's take quick look at them one by one.

### UI Layout - List Students

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-ui-layout-list-students.png" alt="Build it with Spring Boot - CRUD Application - UI Layout - List Students" />
</div>
{{< /rawhtml >}}

`List Students` page which will have list of students in tabular format. There is also a `Add New` button on the page to add new student and there are `Previous` and `Next` buttons to paginate the student list.

Each row in the table corresponds to a single Student and also contains the individual student actions with the use of Buttons - `View, Edit` and `Delete`.


### UI Layout - Add Student

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-ui-layout-add-student.png" alt="Build it with Spring Boot - CRUD Application - UI Layout - Add Student" />
</div>
{{< /rawhtml >}}

`Add Student` screen allows to add a new student. It has Text fields for `First Name` and `Last Name`. The `Add` button saves the new student information and `Cancel` to discard the add operation and go back to `List Students` page.


### UI Layout - View Student

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-ui-layout-view-student.png" alt="Build it with Spring Boot - CRUD Application - UI Layout - View Student" />
</div>
{{< /rawhtml >}}

`View Student` screen is similar to `Add Student`, allows to view information of an existing student. It displayes the `First Name` and `Last Name` of the selected user on the page, and there is a `Go Back` button which takes the user to the `List Students` page.

### UI Layout - Edit Student

`Edit Student` screen, yet again similar to `Add Student` and `View Student`,  and allows to edit information of an existing student. It has Text fields for `First Name` and `Last Name`. The `Update` button saves the updated student information and `Cancel` to discard the operation and go back to `List Students` page.


### UI Layout - Delete Student

Keeping things simple yet again and `Delete Student` screen, is similar to `Add Student, View Student` and `Edit Student` screens, and it allows to delete an existing student. It shows the student information that is about to be deleted. `Delete` button to confirm the delete operation and `Cancel` to discard the operation and go back to `List Students` page.

So, those are all the pages that we need. And let’s get building.





{{< articlead >}}

## Thymeleaf

If you working with [Spring MVC][13] with your views being HTML pages, then [Thymeleaf][11] is the best View technology that works seemlessly. Thymeleaf emphasizes on natural HTML templates that just work. You don't need any rendering engine to view them. Simply open the templates in the Browser and they work perfectly fine, because the code is pure HTML. For example, rendering a text value in variable - `message` in `<p>` tag would be like this.

```html
<p th:text="${message}">This is a template value</p>
```

The part `th:text="${message}"` is Thymeleaf specific syntax and when passed through a rendering engine, the value of `message` property will replace `This is a template value`. Otherwise, if you open this in regular browser directly, `th:text="${message}"` is treated as unknown HTML tag attribute and browsers simply ignore it, and render the tag as - `<p>This is a template value</p>`.

If you are trying to move away from JSPs, [Thymeleaf][11] is the perfect alternative and it will make the migration that much more simpler compared any other view technologies that [Spring MVC][13] supports.

Let's add the support for [Thymeleaf][11] by adding the dependency for `spring-boot-starter-thymeleaf` in `pom.xml`.

{{< rawhtml >}}
<div class="notification">
    You will find the Maven Project completed till <a href="https://github.com/the-code-journal/build-it-with-spring-boot/tree/main/01-mvc-jpa-thymeleaf/part02">Part 2 here</a>, if you are following this article directly.
</div>
{{< /rawhtml >}}

```xml
<!-- View -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

And with that, Spring Boot's autoconfiguration for [Thymeleaf][11] are available. A lot of thymeleaf configurations are available under `spring.thymeleaf.*` configuration group, but if you follow the conventions, you don't need any additional configuration. However, you can still take a look at the [configurations here][14].

One such convention that is configured by Spring Boot by default is `spring.thymeleaf.prefix`. This is the prefix that is automatically added to view names when building the URL. The default value is below.

```yaml
spring:
  thymeleaf:
    prefix: "classpath:/templates/"
    suffix: "html"
```

What this indicates is that we need to place all our view files under directory `/templates/` which is available on classpath. Since, `src/main/resources` gets added to our classpath, we can create a folder in here as `templates` and we can store all our files in there. So, let's create this `templates` directory. Also, all view names would automatically be suffixed as `.html` from the configuration - `spring.thymeleaf.suffix`.

Also, it is a good practice to store all the views for a specific Entity in a dedicated folder. So, we will create another directory as `/templates/students` where we will store all the pages that we will create for students in this folder.

```bash
-> pwd
/home/codejournal/sts-workspace/mvc-jpa-thymeleaf

-> mkdir -p src/main/resources/templates/students

-> tree src/main/resources/
src/main/resources/
├── application.yaml
└── templates
    └── students
```





## Controller

Till now, everything we wanted to see, was from Application Logs or h2-console. We didn't have a endpoint which we could call on our server. With [Spring MVC][13], you can define a [`Controller`][15] which will receive requests sent by client(browser in our case) and return the view which should be shown to the user.

A [`Controller`][15] can simply be configured by adding a [`@Controller`][15] annotation to a class. To mark which method should be called when an endpoint is called, the method needs to be annotated using [`@RequestMapping`][16] which provides properties to configure how the mapping should be done. To make our lives even more simpler, [Spring MVC][13] also provides convinient methods for frequently used HTTP Method(`GET, POST, PUT, DELETE, PATCH`). The corresponding annotations for these are `@GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping`.

All these annotations, take a String property which signifies which endpoint url they should map to. For example,

```java
@GetMapping("/students/")
```

this would map to `http://localhost:8080/students/`

{{< rawhtml >}}
<div class="notification">
    While with <code>@Controller</code>, the request handling is simplified to a great extent, if you want to understand more in detail how Spring handles this internally using <code>DispatcherServlet</code>, I strongly advice reading about this in the <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet">documentation here</a>.
</div>
{{< /rawhtml >}}

Let's create a class `StudentController` and add two methods `index()` and `getStudents()` that have `@GetMapping` annotations on them. The return values for these methods needs to be `String` since it indicates the view name that should be rendered as response.

```java
package io.codejournal.springboot.mvcjpathymeleaf.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class StudentController {

    @GetMapping("/students/")
    public String index() {
        return "redirect:list";
    }

    @GetMapping("/students/list")
    public String getStudents() {
        return "students/list";
    }
}
```

Two points to mention here.

- `redirect:list` allows a redirection from one endpoint to another. That way when somebody calls `/students/` we automatically redirect the user to `/students/list`.
- Returning the value as `"students/list"` indicates that the view that should be rendered by the Thymeleaf engine is `/templates/students/list.html`. `/templates/` comes from `spring.thymeleaf.prefix` configuration while `.html` comes from `spring.thymeleaf.suffix`. That way, the [View Resolver][17] locates the view that needs to be rendered.

Let's create this `list.html` in `src/main/resources/templates/students/`.

```html
<!DOCTYPE html SYSTEM "http://www.thymeleaf.org/dtd/xhtml1-strict-thymeleaf-4.dtd">

<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Student List</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>

  <body>
    <h1>Welcome to Thymeleaf(Placeholder)!</h1>
  </body>
</html>
```

Run your server, and try hitting the URL - `http://localhost:8080/students/` or use `-> curl -L http://localhost:8080/students/` and you should get the page.





{{< articlead >}}

## Model object

To pass the data to views from controller, [`Model`][18] object is provided by Spring and you can add values in this model object and it will be passed to the view, where the view can use the values added to the [`Model`][18] and generate the view as intended.

For example, let's modify the `Welcome to Thymeleaf(Placeholder)!` text with the value passed from the `StudentController`. To get access to this [`Model`][18] object, we simply need to add this as our method argument and Spring will create and pass it along with calling the method.

We can set values using the `add*` method provides by the [`Model`][18] class.

```java
@GetMapping("/students/list")
public String getStudents(final Model model) {

    model.addAttribute("message", "Hello World Thymeleaf(Rendered)");

    return "students/list";
}
```

Also, the `list.html` would be modified as below.

```html
<body>
    <h1 th:text="${message}">Welcome to Thymeleaf(Placeholder)!</h1>
</body>
```

And when you access the page now, you will see the rendered version from the Server. You can also open the `list.html` directly to compare the results.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/thymeleaf-with-without-server.png" alt="Thymeleaf - With and Without Server Rendering" />
</div>
{{< /rawhtml >}}





## Reloading View automatically

While working with HTML pages, it is very tiring to everytime refresh the page whenever you make changes. A lot of new web frameworks provide support for this via [hot swapping/reloading()][19] of web pages as soon as they are saved.

Spring Boot too, provides the same functionality with `spring-boot-devtools` which we added in our [Part 2][4].

If you take a close look at the Application logs, you will see a log line like this.

```
2021-06-11 02:17:08.006  INFO 18404 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
```

This starts the [LiveReload][20] server which publishes any new file changes on the Port - 35729. On the browser, you need a way to capture this. [LiveReload][20] provides plugins for Safari and Chrome(sadly Firefox plugin is discontinued). Check their [Installation page][21] for installing these.

You can also add a small script which does the same thing, and `spring-boot-devtools` provides this. Add this in your `<head>` section, and you should be good to go. Everytime, you change your thymeleaf html files, the browser will automatically reload and it will show the new changes.

```html
<script src="http://localhost:35729/livereload.js"></script>
```





## Adding Google Font (Optional)

This is completely optional, but use [Google Fonts][22] on your web pages allows to have consistent text format across different devices.

#### Step 1

Visit [Google Font][22] and chose the font that suits your linking. There are various options available to chose the fonts from - `Categories`, `Language`, and `Font properties`. You can add your own text for Font preview and also increase/decrease the Font Size.

#### Step 2

Once you have the Font of your liking, click on the font tile to go to the Font page. Here you can click on `+ Select this style` link on the multiple font variations. As soon as you select this, you will get the code snippet as example to include in your web pages on the Right.

You can chose between `<link>` and `@import` options, and also how to use the font in CSS property. Below is the screenshot for reference.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/google-font.png" alt="Selecting a Google Font" />
</div>
{{< /rawhtml >}}


#### Step 3

Add the code snippet in your Web Pages and use the fonts. For our project, I am using [Rubik - Light 300][23] as the Font.

To include this font, we will add the `<style>` tag in our `<head>` section, and use `@import` format to include this font.

```html
<style>
    @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

    body {
        font-family: 'Rubik', sans-serif;
    }
</style>
```




{{< articlead >}}

## Bootstrap

Anybody who has worked with front-end for long, would definately know about [Bootstrap][24]. It's a free and open-source CSS framework that allows to build responsive and mobile-first front-end web pages. The contains CSS and Javascript based design templates for Typhography, Forms, Buttons, Navigation, and many other interface components.

With [Bootstrap][24], you can build beautiful and consistent web page components without too much effort. All you have to do is to add styles appropriately to your HTML structure. You can head over to the [Docs section][25] and find examples of all the various components and layouts that are provided by [Bootstrap][24].

To include Bootstrap in our pages, visit the [Docs section][25] for latest steps. Essentially, you will need to include the CSS and Javascript files, in your `<head>` section as below.

```html
<!-- Bootstrap 5.0.2 at the time of writing this -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
    integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
    integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM"
    crossorigin="anonymous"></script>
```





## Bootstrap Components for Thymeleaf Views

As already mentioned, there are several Bootstrap components available that can be customized to tailor-fit your needs. I won't go into all of these and I would recommend you [check this out][25] yourself while working on any of your projects. Instead, below are the various components that I have used to build the UI for each of the pages.

### Students List Page

On the Students List page, we are going to use Bootstrap components - [navbar][26], [button][27], [table][28], [pagination][29]. Below is how they are laid out on the UI.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-bootstrap-student-list.png" alt="Bootstrap Components - Student List Page" />
</div>
{{< /rawhtml >}}

### View Students Page

On the View Student page, we are going to use Bootstrap components - [navbar][26], [button][27], [alert][30], [form(input)][31]. Below is how they are laid out on the UI.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-bootstrap-student-view.png" alt="Bootstrap Components - View Student Page" />
</div>
{{< /rawhtml >}}

### Add Students Page

On the Add Student page, we are going to use Bootstrap components - [navbar][26], [button][27], [form(input)][31]. Below is how they are laid out on the UI.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-bootstrap-student-add.png" alt="Bootstrap Components - Add Student Page" />
</div>
{{< /rawhtml >}}

### Edit Students Page

On the Edit Student page, we are going to use Bootstrap components - [navbar][26], [button][27], [alert][30], [form(input)][31]. Below is how they are laid out on the UI.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-bootstrap-student-edit.png" alt="Bootstrap Components - Edit Student Page" />
</div>
{{< /rawhtml >}}

### Delete Students Page

On the Delete Student page, we are going to use Bootstrap components - [navbar][26], [button][27], [alert][30], [form(input)][31]. Below is how they are laid out on the UI.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-bootstrap-student-delete.png" alt="Bootstrap Components - Delete Student Page" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Thymeleaf Views

Now, I won't go into detail what is the HTML structure of the Thymeleaf views. Its pretty self explanatory. Plus, I would encourage you to look at the [Bootstrap documentation][25] and adjust the UI as per your liking. It will give you much better confidence on [Bootstrap][24].

Below is the HTML for all the views.

### Student List Page

{{< rawhtml >}}
<pre class="code-output">
&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.thymeleaf.org">
&lt;head>
    &lt;title>Student List&lt;/title>
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    &lt;script src="http://localhost:35729/livereload.js">&lt;/script>
    &lt;style>
        @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

        body {
            font-family: 'Rubik', sans-serif;
        }
    &lt;/style>
    &lt;link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
    &lt;script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
&lt;/head>

&lt;body>
    &lt;nav class="navbar navbar-expand-lg navbar-light bg-light">
        &lt;div class="container-fluid">
            &lt;h1>Student CRUD Operations&lt;/h1>
            &lt;div class="d-flex">
                &lt;button class="btn btn-primary" type="submit">Logout&lt;/button>
            &lt;/div>
        &lt;/div>
    &lt;/nav>

    &lt;br />
    &lt;br />

    &lt;div class="container-sm">
        &lt;table class="table table-bordered table-striped table-hover">
            &lt;thead>
                &lt;tr>
                    &lt;th style="width: 10%; text-align: center">#&lt;/th>
                    &lt;th style="width: 30%;">First Name&lt;/th>
                    &lt;th style="width: 30%;">Last Name&lt;/th>
                    &lt;th style="width: 30%;">&nbsp;&lt;/th>
                &lt;/tr>
            &lt;/thead>
            &lt;tbody>
                &lt;tr>
                    &lt;td>1&lt;/td>
                    &lt;td>John&lt;/td>
                    &lt;td>Doe&lt;/td>
                    &lt;td>
                        &lt;a class="btn btn-info" href="#" role="button">View&lt;/a>
                        &lt;a class="btn btn-warning" href="#" role="button">Edit&lt;/a>
                        &lt;a class="btn btn-danger" href="#" role="button">Delete&lt;/a>
                    &lt;/td>
                &lt;/tr>
                &lt;tr>
                    &lt;td>2&lt;/td>
                    &lt;td>Rafael&lt;/td>
                    &lt;td>Nadal&lt;/td>
                    &lt;td>
                        &lt;a class="btn btn-info" href="#" role="button">View&lt;/a>
                        &lt;a class="btn btn-warning" href="#" role="button">Edit&lt;/a>
                        &lt;a class="btn btn-danger" href="#" role="button">Delete&lt;/a>
                    &lt;/td>
                &lt;/tr>
            &lt;/tbody>
        &lt;/table>

        &lt;nav class="navbar navbar-expand-lg navbar-light">
            &lt;div class="container-fluid">
                &lt;a href="#" class="btn btn-primary">Add new&lt;/a>
                &lt;div class="d-flex">
                    &lt;nav aria-label="Page navigation example">
                        &lt;ul class="pagination">
                            &lt;li class="page-item">&lt;a class="page-link" href="#">Previous&lt;/a>&lt;/li>
                            &lt;li class="page-item">&lt;a class="page-link" href="#">Next&lt;/a>&lt;/li>
                        &lt;/ul>
                    &lt;/nav>
                &lt;/div>
            &lt;/div>
        &lt;/nav>
    &lt;/div>
&lt;/body>
&lt;/html>
</pre>
{{< /rawhtml >}}


### View Student Page

{{< rawhtml >}}
<pre class="code-output">
&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.thymeleaf.org">
&lt;head>
    &lt;title>Student List&lt;/title>
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    &lt;script src="http://localhost:35729/livereload.js">&lt;/script>
    &lt;style>
        @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

        body {
            font-family: 'Rubik', sans-serif;
        }
    &lt;/style>
    &lt;link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
    &lt;script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
&lt;/head>

&lt;body>
    &lt;nav class="navbar navbar-expand-lg navbar-light bg-light">
        &lt;div class="container-fluid">
            &lt;h1>Student CRUD Operations&lt;/h1>
            &lt;div class="d-flex">
                &lt;button class="btn btn-primary" type="submit">Logout&lt;/button>
            &lt;/div>
        &lt;/div>
    &lt;/nav>

    &lt;br />
    &lt;br />

    &lt;div class="container-sm">

        &lt;h2 class="navbar-brand">Student Details&lt;/h2>

        &lt;div class="alert alert-danger" role="alert">
            Student not found
        &lt;/div>

        &lt;form action="#" method="post">
            &lt;div class="mb-3">
                &lt;label for="firstName" class="form-label">First Name&lt;/label>
                &lt;input type="text" class="form-control" id="firstName" disabled="true" />
            &lt;/div>
            &lt;div class="mb-3">
                &lt;label for="lastName" class="form-label">Last Name&lt;/label>
                &lt;input type="text" class="form-control" id="lastName" disabled="true" />
            &lt;/div>
            &lt;a href="#" class="btn btn-outline-secondary">Cancel&lt;/a>
        &lt;/form>
    &lt;/div>
&lt;/body>
&lt;/html>
</pre>
{{< /rawhtml >}}


### Add Student Page

{{< rawhtml >}}
<pre class="code-output">
&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.thymeleaf.org">
&lt;head>
    &lt;title>Student List&lt;/title>
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    &lt;script src="http://localhost:35729/livereload.js">&lt;/script>
    &lt;style>
        @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

        body {
            font-family: 'Rubik', sans-serif;
        }
    &lt;/style>
    &lt;link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
    &lt;script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
&lt;/head>

&lt;body>
    &lt;nav class="navbar navbar-expand-lg navbar-light bg-light">
        &lt;div class="container-fluid">
            &lt;h1>Student CRUD Operations&lt;/h1>
            &lt;div class="d-flex">
                &lt;button class="btn btn-primary" type="submit">Logout&lt;/button>
            &lt;/div>
        &lt;/div>
    &lt;/nav>

    &lt;br />
    &lt;br />

    &lt;div class="container-sm">
        &lt;h2 class="navbar-brand">Add Student&lt;/h2>
        &lt;form action="#" method="post">
            &lt;div class="mb-3">
                &lt;label for="firstName" class="form-label">First Name&lt;/label>
                &lt;input type="text" class="form-control" id="firstName"
                    placeholder="First Name of the Student" />
            &lt;/div>
            &lt;div class="mb-3">
                &lt;label for="lastName" class="form-label">Last Name&lt;/label>
                &lt;input type="text" class="form-control" id="lastName"
                    placeholder="Last Name of the Student" />
            &lt;/div>
            &lt;input type="submit" class="btn btn-success" value="Add" />
            &lt;a href="#" class="btn btn-outline-secondary">Cancel&lt;/a>
        &lt;/form>
    &lt;/div>
&lt;/body>
&lt;/html>
</pre>
{{< /rawhtml >}}


### Edit Student Page

{{< rawhtml >}}
<pre class="code-output">
&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.thymeleaf.org">
&lt;head>
    &lt;title>Student List&lt;/title>
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    &lt;script src="http://localhost:35729/livereload.js">&lt;/script>
    &lt;style>
        @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

        body {
            font-family: 'Rubik', sans-serif;
        }
    &lt;/style>
    &lt;link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
    &lt;script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
&lt;/head>

&lt;body>
    &lt;nav class="navbar navbar-expand-lg navbar-light bg-light">
        &lt;div class="container-fluid">
            &lt;h1>Student CRUD Operations&lt;/h1>
            &lt;div class="d-flex">
                &lt;button class="btn btn-primary" type="submit">Logout&lt;/button>
            &lt;/div>
        &lt;/div>
    &lt;/nav>

    &lt;br />
    &lt;br />

    &lt;div class="container-sm">
        &lt;h2 class="navbar-brand">Edit Student&lt;/h2>

        &lt;div class="alert alert-danger" role="alert">
            Student not found
        &lt;/div>

        &lt;form action="#" method="post">
            &lt;div class="mb-3">
                &lt;label for="firstName" class="form-label">First Name&lt;/label>
                &lt;input type="text" class="form-control" id="firstName"
                    placeholder="First Name of the Student" />
            &lt;/div>
            &lt;div class="mb-3">
                &lt;label for="lastName" class="form-label">Last Name&lt;/label>
                &lt;input type="text" class="form-control" id="lastName"
                    placeholder="Last Name of the Student" />
            &lt;/div>
            &lt;input type="submit" class="btn btn-success" value="Update" />
            &lt;a href="#" class="btn btn-outline-secondary">Cancel&lt;/a>
        &lt;/form>
    &lt;/div>
&lt;/body>
&lt;/html>
</pre>
{{< /rawhtml >}}


### Delete Student Page

{{< rawhtml >}}
<pre class="code-output">
&lt;!DOCTYPE html>
&lt;html lang="en" xmlns:th="http://www.thymeleaf.org">
&lt;head>
    &lt;title>Student List&lt;/title>
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    &lt;script src="http://localhost:35729/livereload.js">&lt;/script>
    &lt;style>
        @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@300&display=swap');

        body {
            font-family: 'Rubik', sans-serif;
        }
    &lt;/style>
    &lt;link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous" />
    &lt;script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
&lt;/head>

&lt;body>
    &lt;nav class="navbar navbar-expand-lg navbar-light bg-light">
        &lt;div class="container-fluid">
            &lt;h1>Student CRUD Operations&lt;/h1>
            &lt;div class="d-flex">
                &lt;button class="btn btn-primary" type="submit">Logout&lt;/button>
            &lt;/div>
        &lt;/div>
    &lt;/nav>

    &lt;br />
    &lt;br />

    &lt;div class="container-sm">
        &lt;h2 class="navbar-brand">Delete Student&lt;/h2>

        &lt;div class="alert alert-danger" role="alert">
            Student not found
        &lt;/div>

        &lt;form action="#" method="post">
            &lt;div class="mb-3">
                &lt;label for="firstName" class="form-label">First Name&lt;/label>
                &lt;input type="text" class="form-control" id="firstName" disabled="true" />
            &lt;/div>
            &lt;div class="mb-3">
                &lt;label for="lastName" class="form-label">Last Name&lt;/label>
                &lt;input type="text" class="form-control" id="lastName" disabled="true" />
            &lt;/div>
            &lt;input type="submit" class="btn btn-danger" value="Delete" />
            &lt;a href="#" class="btn btn-outline-secondary">Cancel&lt;/a>
        &lt;/form>
    &lt;/div>
&lt;/body>
&lt;/html>
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Controller

To allow all these views to be returned when we make request for them, we need to add methods in our `StudentController`. Below are the paths that return these pages.

- Returns `list.html` when HTTP request is `/students/` or `/students/list`
- Returns `view.html` when HTTP request is `/students/view`
- Returns `add.html` when HTTP request is `/students/add`
- Returns `edit.html` when HTTP request is `/students/edit`
- Returns `delete.html` when HTTP request is `/students/delete`

Below is the full `StudentController` which has methods to handle all the above requests.

```java
package io.codejournal.springboot.mvcjpathymeleaf.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class StudentController {

    @GetMapping("/students/")
    public String index() {
        return "redirect:list";
    }

    @GetMapping("/students/list")
    public String getStudents(final Model model) {
        return "students/list";
    }

    @GetMapping("/students/add")
    public String add() {
        return "students/add";
    }

    @GetMapping("/students/view")
    public String view() {
        return "students/view";
    }

    @GetMapping("/students/edit")
    public String edit() {
        return "students/edit";
    }

    @GetMapping("/students/delete")
    public String delete() {
        return "students/delete";
    }
}
```

And with this, [Part 3][5] concludes here. Again, a lot of things and we completed our [Thymeleaf][11] web pages

In [Part 4][6], we will fetch data from Database and populate our Thymeleaf views.

Here is the [Github project for Part 3][32] for your reference.


  [1]: https://spring.io/projects/spring-boot
  [2]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
  [3]: /2021/06/build-it-with-spring-boot-crud-operations-part-1/
  [4]: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
  [5]: /2021/06/build-it-with-spring-boot-crud-operations-part-3/
  [6]: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
  [7]: /2021/06/build-it-with-spring-boot-crud-operations-part-3/
  [8]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
  [9]: https://www.oracle.com/java/technologies/persistence-jsp.html
 [10]: https://hibernate.org/orm/
 [11]: https://www.thymeleaf.org
 [12]: https://getbootstrap.com/
 [13]: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html
 [14]: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.templating
 [15]: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-controller
 [16]: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping
 [17]: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver
 [18]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/ui/Model.html
 [19]: https://en.wikipedia.org/wiki/Hot_swapping
 [20]: http://livereload.com/
 [21]: http://livereload.com/extensions/
 [22]: https://fonts.google.com
 [23]: https://fonts.google.com/specimen/Rubik?query=Rubik
 [24]: https://getbootstrap.com/
 [25]: https://getbootstrap.com/docs/5.0/
 [26]: https://getbootstrap.com/docs/5.0/components/navbar/
 [27]: https://getbootstrap.com/docs/5.0/components/buttons/
 [28]: https://getbootstrap.com/docs/5.0/content/tables/
 [29]: https://getbootstrap.com/docs/5.0/components/pagination/
 [30]: https://getbootstrap.com/docs/5.0/components/alerts/
 [31]: https://getbootstrap.com/docs/5.0/forms/overview/
 [32]: https://github.com/the-code-journal/build-it-with-spring-boot/tree/main/01-mvc-jpa-thymeleaf/part03
