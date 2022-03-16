---
date: 2021-06-28
linktitle: Build it with Spring Boot - CRUD Operations - Part 5
next: /2020/08/unit-testing-in-maven-surefire-configurations/
prev: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
title: Build it with Spring Boot - CRUD Operations - Part 5
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





## Recap from Part 4

We are pretty much done with our application. In [Part 4][6], we added all the features to list, add, edit, view and delete student information.

From our [Model View Controller][8] diagram, everything is ready.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/model-view-controller-part5.png" alt="Model View Controller - Part 5" />
</div>
{{< /rawhtml >}}

From the application structure, we completed our Service layer, [JPA][9] layer, Database access via [Hibernate][10], Controller that accepts requests and [Thymeleaf][11] views that show the information. We used [Bootstrap][12] to style our view pages.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/spring-boot-crud-operation-application-structure-part5.png" alt="Build it with Spring Boot - CRUD Application Structure - Part 5" />
</div>
{{< /rawhtml >}}

In this video, we will secure our application using [Spring Security][12].

Below is the video for the same that guides you for this part.

{{< yt "https://www.youtube.com/embed/ZIZ4AVkVL0g" >}}





{{< articlead >}}

## Staring with Spring Security

Spring Security provides authentication, authorization and protection against common attackes for your application. The simplest way to add a default security for your application is via the [Spring Security Auto Configuration][13]. It provides a lot of features out of the box. Here are some features(check [latest documentation][13] for updated list)

- Enables Spring Security’s default configuration, which creates a servlet `Filter` as a bean named `springSecurityFilterChain` and registers it with the Servlet container for every request. This bean is responsible for all the security (protecting the application URLs, validating submitted username and passwords, redirecting to the log in form, and so on) within your application.
- Creates a `UserDetailsService` bean with a username of `user` and a randomly generated password that is logged to the console.
- Registers the `Filter` with a bean named `springSecurityFilterChain` with the Servlet container for every request.
- Generate a default login form for you
- Let the user with a username of user and a password that is logged to the console to authenticate with form-based authentication
- Protects the password storage with BCrypt
- Lets the user log out
- [CSRF attack][14] prevention
- [Session Fixation][15] protection
- Security Header integration
- Integrate with the following Servlet API methods:
  - `HttpServletRequest#getRemoteUser()`
  - `HttpServletRequest.html#getUserPrincipal()`
  - `HttpServletRequest.html#isUserInRole(String)`
  - `HttpServletRequest.html#login(String, String)`
  - `HttpServletRequest.html#logout()`

To enable Spring Security, simple add the dependency in your `pom.xml` as below.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Along with this, we need to enable the auto configuration by using annotation [`@EnableWebSecurity`][16]. Simply, add this in your `MvcJpaThymeleafApplication.java`. (P.S. It doesn't matter on which class you put this annotation.)

```java
@SpringBootApplication
@EnableWebSecurity  // <---------------------- Enable Web Security
public class MvcJpaThymeleafApplication {
    // Rest of the class body
}
```

If you run your application now, you will see that the application is now doing a lot of more things at startup and you will see the default password for the application like below.

{{< rawhtml >}}
<pre class="code-output">
2022-03-16 21:51:34.228  INFO 32353 --- [  restartedMain] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: c346bcd2-b1c2-41a2-b17d-1c7ce413cc1c

2022-03-16 21:51:34.273  INFO 32353 --- [  restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will not secure any request
</pre>
{{< /rawhtml >}}





## Configuration - Protecting endpoints

The easiest way to start configuring the Spring Security for a Web Application is by using [`WebSecurityConfigurerAdapter`][17]. It provides various methods that you can use to configure beans that Spring creates. Create a class `SecurityConfig`, add `@Configuration` annotation and extend the [`WebSecurityConfigurerAdapter`][17] class.

```java
package io.codejournal.springboot.mvcjpathymeleaf.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // Security Config here
}
```

Let's start with the first two methods that we can use to configure the endpoints where Spring Security can take control.

```java
public void configure(WebSecurity web) throws Exception;
public void configure(HttpSecurity http) throws Exception;
```

- `configure(WebSecurity web)` Endpoint used can be used to ignore the spring security filters, security features (secure headers, csrf protection etc) are also ignored and no security context will be set and can not protect endpoints for Cross-Site Scripting, XSS attacks, content-sniffing.
- `configure(HttpSecurity http)` Endpoint can be used to ignore the authentication for endpoints used in antMatchers and other security features will be in effect such as secure headers, CSRF protection, etc.

For instance, our `/h2-console` endpoint does not need any Spring Security, so we can completely ignore that in `configure(WebSecurity web)`. Here is how we will do that.

```java
public void configure(final WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/h2-console/**");
}
```

For the `HttpSecurity`, we need following things.
1. We need the Login form
2. Once the Login is successful, we need to redirect the user to the Student List page
3. All Student pages(list, add, view, edit and delete) should be protected

The above configuration can be done as part of `configure(HttpSecurity http)` as below

```java
public void configure(final HttpSecurity http) throws Exception {
    http
        .formLogin()                          // Enables Login Form
            .defaultSuccessUrl("/students/")  // Redirects user to /students/ after successful login
        .and()
            .logout().permitAll()             // Allow everybody to access the Logout endpoint(/logout)
        .and()
            .authorizeRequests()              // Validate request to have role(USER) unless otherwise
            .antMatchers("/**").hasRole("USER")
    ;
}
```





{{< articlead >}}

## Configuration - Authentication Provider

As the name suggests, [`AuthenticationProvider`][24] configures the provider who will perform the Authentication. For example, [`DaoAuthenticationProvider`][18] supports username/password based authentication while [`JwtAuthenticationProvider`][19] supports authenticating a JWT token. For our case, we will be using [`DaoAuthenticationProvider`][18]. It needs 2 beans in context to be able to perform authentication.

1. Instance of [`PasswordEncoder`][20] - to encode the password before comparing the passwordsfrom the database
2. Instance of [`UserDetailsService`][21] - to load the Users 

For [`PasswordEncoder`][20], we can use [`BCryptPasswordEncoder`][22] provided by Spring Security. It uses [BCrypt strong hashing][23] function to encode the passwords.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

For the [`UserDetailsService`][21], we can have our `UserService` class implement this interface and we can load the user from the database from there. It provides a method `loadUserByUsername​(String username)` to load the User. Below is the code added to `UserService`.

```java
package io.codejournal.springboot.mvcjpathymeleaf.service;

import static java.util.Collections.singletonList;

import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import io.codejournal.springboot.mvcjpathymeleaf.entity.User;
import io.codejournal.springboot.mvcjpathymeleaf.repository.UserRepository;

@Service
public class UserService implements UserDetailsService {

    private final UserRepository repository;

    @Autowired
    public UserService(final UserRepository repository) {
        this.repository = repository;
    }

    @Override
    public UserDetails loadUserByUsername(final String username) throws UsernameNotFoundException {

        final Optional<User> record = repository.findByUsername(username);

        if (record.isEmpty()) {
            throw new UsernameNotFoundException("User not found - " + username);
        }

        final User user = record.get();

        // @formatter:off
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(),
                singletonList(new SimpleGrantedAuthority("ROLE_USER"))
        );
        // @formatter:on
    }

    public User save(final User user) {
        return repository.save(user);
    }
}
```

{{< rawhtml >}}
<div class="notification">
    I also added the <code>save(User user)</code> method in <code>UserService</code> above since it is used later and I was getting lazy to mention that separately 😀
</div>
{{< /rawhtml >}}


Now, that we have our [`PasswordEncoder`][20] and [`UserDetailsService`][21] beans available, we can configure our [`AuthenticationProvider`][24] as a Spring Bean.

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService service;

    //...

    @Bean
    public AuthenticationProvider authenticationProvider() {

        final DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();

        authenticationProvider.setPasswordEncoder(passwordEncoder());
        authenticationProvider.setUserDetailsService(service);

        return authenticationProvider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```





{{< articlead >}}

## Configuration - Authentication Manager

Finally, we need to let the [`AuthenticationManager`][25] know about our [`AuthenticationProvider`][24] mechanism(username/password verification will be done using a Database) and [`WebSecurityConfigurerAdapter`][17] provides a method to just do that.

```java
@Override
public void configure(final AuthenticationManagerBuilder auth) throws Exception {
  auth.authenticationProvider(authenticationProvider());
}
```

And instead of using the default Spring Security credentials, we need to create a User in our database. We can simply create a test user when the application starts-up. Again, we can use [`ApplicationRunner`][26] class to insert a test User when the application starts.

```java
@Bean
public ApplicationRunner initializeUsers(final PasswordEncoder passwordEncoder) {

    final User defaultUser = new User("code-journal", passwordEncoder.encode("password"));

    return args -> service.save(defaultUser);
}
```

And with that our `SecurityConfig` class is complete. Below is the full class for your reference.

{{< rawhtml >}}
<pre class="code-output">
package io.codejournal.springboot.mvcjpathymeleaf.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

import io.codejournal.springboot.mvcjpathymeleaf.entity.User;
import io.codejournal.springboot.mvcjpathymeleaf.service.UserService;

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService service;

    @Override
    public void configure(final WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/h2-console/**");
    }

    @Override
    public void configure(final HttpSecurity http) throws Exception {
        // @formatter:off
        http.formLogin()
                .defaultSuccessUrl("/students/")
            .and()
                .logout()
                .permitAll()
            .and()
                .authorizeRequests()
                .antMatchers("/**").hasRole("USER")
        ;
        // @formatter:on
    }

    @Override
    public void configure(final AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(authenticationProvider());
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {

        final DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();

        authenticationProvider.setPasswordEncoder(passwordEncoder());
        authenticationProvider.setUserDetailsService(service);

        return authenticationProvider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public ApplicationRunner initializeUsers(final PasswordEncoder passwordEncoder) {

        final User defaultUser = new User("code-journal", passwordEncoder.encode("password"));

        return args -> service.save(defaultUser);
    }
}
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Configuration - Adding CSRF Token

By default, Spring Security enables the [CSRF protection][27] for your application and endpoints. Since, we haven't disabled this, we will need to include the CSRF Token in our [Thymeleaf][28].

Including the CSRF Token can be done via multiple ways and you can [lookup the documentation][29] that fits your use case. For our use case, we will use the CSRF Request Attribute which are exposed as an `HttpServletRequest` attribute named `_csrf`. For example,

```xml
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" />
```

We will need to include this all of `<form>` elements of our [Thymeleaf][28] views. Here are the list of changes to our views.



### add.html

To allow logout to work correctly, we need to update the Navigation Header to below.

```xml
<div class="navbar-header">
    <h1 class="navbar-brand">Student CRUD Operations</h1>
</div>
<div class="d-flex">
    <form action="/logout" method="POST">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        <input type="submit" class="btn btn-primary" value="Logout" />
    </form>
</div>
```

Also, the form which adds the Student information will also include the CSRF token.

```xml
<form th:action="@{save}" method="post" th:object="${student}">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <!-- Rest of the Form -->
</form>
```



### edit.html

To allow logout to work correctly, we need to update the Navigation Header to below.

```xml
<div class="navbar-header">
    <h1 class="navbar-brand">Student CRUD Operations</h1>
</div>
<div class="d-flex">
    <form action="/logout" method="POST">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        <input type="submit" class="btn btn-primary" value="Logout" />
    </form>
</div>
```

Also, the form which edits the Student information will also include the CSRF token.

```xml
<form th:action="@{save}" method="post" th:object="${student}" th:if="${student.id != null}">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <!-- Rest of the Form -->
</form>
```



### delete.html

To allow logout to work correctly, we need to update the Navigation Header to below.

```xml
<div class="navbar-header">
    <h1 class="navbar-brand">Student CRUD Operations</h1>
</div>
<div class="d-flex">
    <form action="/logout" method="POST">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        <input type="submit" class="btn btn-primary" value="Logout" />
    </form>
</div>
```

Also, the form which edits the Student information will also include the CSRF token.

```xml
<form th:action="@{delete}" method="post" th:object="${student}" th:if="${student.id != null}">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <!-- Rest of the Form -->
</form>
```



### list.html and view.html

To allow logout to work correctly, we need to update the Navigation Header to below.

```xml
<div class="navbar-header">
    <h1 class="navbar-brand">Student CRUD Operations</h1>
</div>
<div class="d-flex">
    <form action="/logout" method="POST">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        <input type="submit" class="btn btn-primary" value="Logout" />
    </form>
</div>
```

{{< rawhtml >}}
<div class="notification">
    The header section in all the views can be extracted to a common file and then it can be included in any of the files. That way, there is just a single location where changes related to header exists. Refer to <a href="https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#template-layout" target="_blank">Thymeleaf documentation here</a> for more details.
</div>
{{< /rawhtml >}}

And finally, the application is ready. Run the application and navigate to `http://localhost:8080/` and you will be presented with login page. Login with credentials(`code-journal/password`), navigate the application and perform Logout. Everything should be working.

Here is the [Github project for Part 5][30] for your reference.


  [1]: https://spring.io/projects/spring-boot
  [2]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
  [3]: /2021/06/build-it-with-spring-boot-crud-operations-part-1/
  [4]: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
  [5]: /2021/06/build-it-with-spring-boot-crud-operations-part-3/
  [6]: /2021/06/build-it-with-spring-boot-crud-operations-part-4/
  [7]: /2021/06/build-it-with-spring-boot-crud-operations-part-5/
  [8]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
  [9]: https://www.oracle.com/java/technologies/persistence-jsp.html
 [10]: https://hibernate.org/orm/
 [11]: https://www.thymeleaf.org
 [12]: https://getbootstrap.com/
 [13]: https://docs.spring.io/spring-security/reference/servlet/getting-started.html#servlet-hello-auto-configuration
 [14]: https://en.wikipedia.org/wiki/Cross-site_request_forgery
 [15]: https://en.wikipedia.org/wiki/Session_fixation
 [16]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/EnableWebSecurity.html
 [17]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html
 [18]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html#servlet-authentication-daoauthenticationprovider
 [19]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/server/resource/authentication/JwtAuthenticationProvider.html
 [20]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/password/PasswordEncoder.html
 [21]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetailsService.html
 [22]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html
 [23]: https://en.wikipedia.org/wiki/Bcrypt
 [24]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationProvider.html
 [25]: https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationManager.html
 [26]: https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html
 [27]: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html
 [28]: https://www.thymeleaf.org
 [29]: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html#servlet-csrf-include
 [30]: https://github.com/the-code-journal/build-it-with-spring-boot/tree/main/01-mvc-jpa-thymeleaf/part05