---
date: 2020-08-31
linktitle: Maven Dependency Scopes
next: /2020/09/maven-project-structure/
prev: /2020/08/maven-dependency-types-direct-and-transitive/
title: Maven Dependency Scopes
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

Maven Dependency Scopes determine when a dependency is included in a classpath, during compilation, test and when you run your application.

Here is the video explaining these.

{{< yt "https://www.youtube.com/embed/wOLS7iyvbTo" >}}





## Maven Classpaths

Maven has following classpaths available during build

- `compile-classpath` - This classpath is used when the application code is being compiled
- `test-classpath` - This classpath is used when the Application code is being tested
- `run-classpath` - This classpath is used when the Application code is run/packaged

In a typical Maven project, you have two kinds of source folders.

1. `src-main-java` - which contains your application code which is intended for production running
2. `src-test-java` - where you have your test classes which essentially tests your application code.

When maven compiles your application code in `src-main-java`, it uses `compile-classpath` and when maven compiles and runs your test classes in `src-test-java`, it uses the `test-classpath`.

And finally, when maven packages your application or runs your application, it uses the `run-classpath`.

Keeping these classpaths in our mind, let’s now talk about maven dependency scopes.





{{< articlead >}}

## Maven Scopes
Maven scopes apply to your dependencies and they define where your dependency land up in various classpaths - `compile, test, run`. So for example, if we want a specific dependency to be available on `test-classpath`, we can configure its scope appropriately.

How do we define the scope of a dependency? We use the xml tag - `scope` in the dependency configuration like this.

```xml
...
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.11</version>
  <scope>test</scope>
</dependency>
...
```

Maven provides us with six dependency scopes. These are
- [compile](#maven-scope---compile)
- [test](#maven-scope---test)
- [runtime](#maven-scope---runtime)
- [provided](#maven-scope---provided)
- [system](#maven-scope---system)
- [import](#maven-scope---import)

Let's talk about these one by one.





{{< articlead >}}

## Maven Scope - compile

This is the default scope for the dependencies. So, if you don’t specify any scope for a dependency, `compile` scope is inferred.

The dependency with `compile` scope is available in all classpaths - `compile, test` and `run`. So, any class of the dependency, can be used in your application code, your test classes, and it will also be available when your application is run by maven.





{{< articlead >}}

## Maven Scope - test

When a dependency is only needed in your test classes, you define the scope as `test`. And with that, the dependency is only available to your test classes, and not anywhere else.





{{< articlead >}}

## Maven Scope - runtime

`runtime` scope is used when you want the dependency only when you test and run your application. This is useful for dynamically linked dependencies like JDBC drivers, which aren’t directly referenced in your code.

Hence, these dependencies are only available on `test-classpath` and `run-classpath`, and NOT on `compile-classpath`.





{{< articlead >}}

## Maven Scope - provided

`provided` scope is very similar to `compile`, except that it expects the environment where you run the application, already contains these dependencies.

For example, almost all the Java Application servers or Java web servers, have `servlet-api` which is the core API that defines the contracts between a servlet class and the runtime environment provided for an instance by a conforming servlet container.

As `provided` scope expects the dependency to be available in the runtime environment, hence, the dependency is only available on `compile-classpath` and `test-classpth`, and NOT on `run-classpath`(because runtime environment already provides it).

Below is the snapshot of the `lib` directory of [Tomcat](https://tomcat.apache.org) and [Jetty](https://www.eclipse.org/jetty), and as you can see, both of these have `servlet-api` dependency.

{{< rawhtml >}}
<pre class="code-output">
[apache-tomcat-9.0.39]         |        [jetty-distribution-9.4.33.v20201020]
-> tree .                      |        -> tree .
.                              |        .
├── bin                        |        ├── bin
├── conf                       |        ├── demo-base
├── lib                        |        ├── etc
│   ├── annotations-api.jar    |        ├── lib
│   ├── ...                    |        │   ├── annotations
                               │        │   ├── ...
│   ├── servlet-api.jar        |        │   ├── servlet-api-3.1.jar
│   ├── ...                    |        │   ├── ...
│   └── websocket-api.jar      |        │   └── websocket
├── logs                       |        ├── logs
├── temp                       |        ├── modules
├── webapps                    |        ├── resources
└── work                       |        ├── start.jar
                               |        └── webapps
</pre>
{{< /rawhtml >}}

Similarly, if there is another dependency that is available in your application server or target runtime environment, you should mark them as `provided` in your maven configuration.





{{< articlead >}}

### Maven Scope - system

`system` scope is similar to `provided`, except that this dependency is not managed by maven.

You need to specify where this dependency exist in your local filesystem using additional tag - `systemPath`, where you give the path to the dependency.

```xml
...
<dependency>
  <groupId>com.company</groupId>
  <artifactId>very-secret</artifactId>
  <version>1.0</version>
  <scope>system</scope>
  <systemPath>/usr/lib/very-secret-1.0.jar</systemPath>
</dependency>
...
```

Here in the example, the path to the dependency is `/usr/lib` and inside this directory, we have our dependency `very-secret 1.0.jar`.

This scope, as already said, works similar to `provided` and hence, it is only available on `compile-classpath` and `test-classpath`.





{{< articlead >}}

### Maven Scope - import

Last one is - `import`. And this scope is defined a little differently. This is part of the `dependencyManagement` section of the pom and it allows you to import all the dependencies configured in the maven coordinates, as is. Here is an example.

```xml
...
<dependencyManagement>
 <dependencies>
  <dependency>
    <groupId>com.company</groupId>
    <artifactId>project-deps</artifactId>
    <version>1.0</version>
    <scope>import</scope>
    <type>pom</type>
  </dependency>
 </dependencies>
</dependencyManagement>
...
```

So, in this example, `project-deps` is essentially a pom in itself which defines one or more dependencies. When it is imported like above, it brings those dependencies in the project where it is imported.

The scopes are kept as it is when they are imported - so, a `compile` scoped dependency defined in `project-deps` is imported as `compile` scoped dependency in the project and similarly for other scopes.

This is very useful when you want a consistent set of dependencies that are used in a team or organization.

For example, we have a team’s pom defining a list of dependency as part of team’s pom. And then each project of the team imports these base set of dependencies.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-import.png" alt="Maven Scope Import Example" />
</div>
{{< /rawhtml >}}

So, when tomorrow you update a dependency in the team’s pom, it will automatically be reflected in all importing projects.





{{< articlead >}}

Here is the reference table for quick reference.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scopes.png" alt="Maven Scopes" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Maven Scopes & Transitive Dependencies

When we talk about scopes, we also need to understand how these scopes affect the transitive dependencies, and if they are imported along with the direct dependency or not.

Now, this only works for `compile, test, runtime` and `provided` scope dependencies, since `system` is not applicable for transitive dependencies. Also `import`, brings the dependencies as they are, so it also does not apply to transitive dependencies

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-transitivity.png" alt="Maven Scopes & Transitive Dependencies" />
</div>
{{< /rawhtml >}}

For example, let’s assume `junit` dependency has all kinds of scope transitive dependency - `compile, test, runtime` and `provided` dependency.

{{< rawhtml >}}
<div class="notification">

    When we define a scope for junit in our maven configuration, which one of these transitive dependency is going to be available in our project and what will be their scope?<br/><br/>
    
    Will they preserve their scope or maven will add them with a different scope in our application?
</div>
{{< /rawhtml >}}

Let’s understand this.





{{< articlead >}}

## Transitive Dependency and Maven Scope - compile

Let’s talk about the `compile` scope. We already know that `compile` scoped dependencies are available on all classpaths - `compile-classpath`, `test-classpath` and `run-classpath`.

When a dependency is marked as `compile` scope, it brings along transitive dependencies that have scope as `compile` or `runtime`.

Also, their scope remains the same. So, a transitive dependency, that is marked as `compile`, will stay as `compile`, and a transitive dependency with scope as `runtime`, will stay as `runtime`.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-transitivity-compile.png" alt="Transitive Dependency for Maven Scope - Compile" />
</div>
{{< /rawhtml >}}






{{< articlead >}}

## Transitive Dependency and Maven Scope - test

We already know that `test` scoped dependencies are available only on `test-classpath`.

When a dependency is marked as `test` scope, it brings along transitive dependencies that have scope as `compile` or `runtime`.

Also, these transitive dependencies are marked as `test` scoped when they are added to the project.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-transitivity-test.png" alt="Transitive Dependency for Maven Scope - Test" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Transitive Dependency and Maven Scope - runtime

We already know that `runtime` scoped dependencies are available on `test-classpath` and `run-classpath`.

When a dependency is marked as `runtime` scope, it brings along transitive dependencies that have scope as `compile` or `runtime`.

Also, these transitive dependencies are marked as `runtime` scope when they are added to the project.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-transitivity-runtime.png" alt="Transitive Dependency for Maven Scope - Runtime" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Transitive Dependency and Maven Scope - provided

And finally, transitivity of `provided` scoped dependencies. We already know that `provided` scoped dependencies are available on `test-classpath` and `run-classpath`.

When a dependency is marked as `provided` scope, it brings along transitive dependencies that have scope as `compile` or `runtime`.

Also, these transitive dependencies are marked as `provided` scoped when they are added to the project.

{{< rawhtml >}}
<div class="image">
    <img src="/images/006-maven-scopes/maven-scope-transitivity-provided.png" alt="Transitive Dependency for Maven Scope - Provided" />
</div>
{{< /rawhtml >}}