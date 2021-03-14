---
date: 2020-08-22
linktitle: Maven Co-ordinates and Depedendency Management
next: /2020/08/maven-dependency-types-direct-and-transitive/
prev: /2020/08/first-build-with-apache-maven-command-prompt/
title: Maven Co-ordinates and Depedendency Management
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

Maven's Dependency Management is at the heart of the power that Maven brings to manage project's dependency. This articles introduces you to the concept. You can also watch the full vide below.

{{< yt "https://www.youtube.com/embed/9GKh-Qxlk7c" >}}





## What is a Dependency?

In a layman’s term, a dependency is an external code that your application uses to run. Because the application code relies on this external code for correct execution, the application code is **dependent** on it and that is why the name **Dependency**.

For example, below is the code for Java's Hello World program.

```java
public class App {

    public static void main(String[] args) {

        System.out.println("Hello World");

    }
}
```

Now, you might feel that there are no dependencies here, but if you see closely, we are using `System` class here. Within the `System` class, there is a static variable, `out`, which handles the output to the Standard console. We use `out` to print output to console, by calling `println` method and passing `"Hello World"` string as argument.

We don’t know how the printing on the console happens. We have not written that, but Java runtime environment or JRE provides this basic set of language dependencies out of the box. To run, Java programs, you need the JRE. Wherever this code runs, the JRE will have the `System` class and hence, this program will run without any issues.

So, in a general sense and keeping Maven in context, this is not a dependency, but if you take the definition of dependency pretty strictly, `System` class is a dependency for your program.



{{< articlead >}}

## Dependency in Maven World

When we talk about dependency in the context of Maven,

{{< rawhtml >}}
<div class="notification">
    a dependency is code or classes that are external to your application and excluding the JRE as well.
</div>
{{< /rawhtml >}}

Take a typical Java Web Application with Database as backend.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2020/08/004-maven-dependency-management/maven-web-application.png" alt="Maven Dependency Java Web Application" />
</div>
{{< /rawhtml >}}

So, in this example, Web Server and Database Driver are the dependencies because these libraries deal with their specific implementations. Web Server has code to deal with requests that are coming and Database Driver knows how to interact with the database. And all we get from these dependencies are a certain set of classes that we need to use in the application to make everything work.



{{< articlead >}}

## Dependency Management

{{< rawhtml >}}
<div class="notification">
    In a nutshell, Dependency Management in Maven allows us to manage compatible versions of the related dependencies automatically.
</div>
{{< /rawhtml >}}

What that means is that, when one of your dependency in turn is dependent on any other dependency, both dependencies are included in your application, and all you need to do is to define the main dependency. Also, when you update the main dependency, the related dependency is also updated.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2020/08/004-maven-dependency-management/maven-web-application-dependency.png" alt="Maven Java Web Application Dependencies" />
</div>
{{< /rawhtml >}}

In our example here, Web Server 1.0 in turn depends on DB Driver 1.0. When we configure these dependencies in Maven, all we need is to configure Web Server 1.0 in our maven configuration and DB Driver 1.0 will be automatically available in our application.

Similarly, when we migrate from Web Server 1.0 to Web Server 2.0, Maven is going to automatically update the DB Driver from 1.0 to 2.0 because Web Server 2.0 depends on DB Driver 2.0. These version compatibility is provided by people managing Web Server, and maven uses it for the correct compatible dependency resolution.



{{< articlead >}}

## Maven Co-ordinates

Configuring a Maven dependency requires you to know what the co-ordinates of your dependency are. Just like GPS coordinates are Latitude and Longitude values that allow us to mark a specific point on our globe, Maven coordinates allow you to precisely specify which dependency and its version you need in your application.

Maven uses the combination of `groupId, artifactId`, and `version` as coordinates. Each combination of `groupId, artifactId, version` uniquely defines a depedency that you want to include in your application.

- `groupId` - represents an organization or a project group. Most dependencies use a dot notation, but there is no hard and fast rule(eg. junit).
- `artifactId` - eepresents the product or module within the project group.
- `version` - The version of the artifact or the project module.

Let's take example of [junit](https://mvnrepository.com/artifact/junit/junit) dependency with coordinates - `groupId(javax.servlet), artifactId(servlet-api), version(2.5)`, here is how they are available on the local maven repository.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/.m2/repository]
-> tree javax
javax
└── servlet
    └── servlet-api
        └── 2.5
            ├── _remote.repositories
            ├── servlet-api-2.5.jar
            ├── servlet-api-2.5.jar.sha1
            ├── servlet-api-2.5.pom
            └── servlet-api-2.5.pom.sha1
</pre>
{{< /rawhtml >}}

The `groupId` creates the team/organization's directory structure and dots(.) are used to separate them into different folders. In our case, these are the folders - `javax` and `servlet`.

`artifactId` folder marks the dependency and all versions of this dependency will reside inside this. So, for our example above, `servlet-api` is the `artifactId` folder `2.5` is the `version` of the dependency. Within the `version` folder, there is the dependency jar file, checksums and this dependencies `pom.xml`.

That is how, Maven uses the coordinates to locate the jar file for the dependency and make it available for your application.



{{< articlead >}}

## Adding a Maven Dependency to POM

To include a dependency in your application, you need to configure the Maven coordinate of your dependency in the `<dependencies>` section of your `pom.xml`.

Here is an example with full POM for adding [Google Guava](https://github.com/google/guava) dependency in your project.

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>
```

To create a simple Maven project, watch our earlier tutorial - [First Build with Maven - Command Prompt](/2020/08/first-build-with-apache-maven-command-prompt).

Here is the full `pom.xml`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.codejournal.maven</groupId>
    <artifactId>dependency-management-simple</artifactId>
    <version>1.0</version>

    <name>dependency-management-simple</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>

        <!-- Adds Google Guava Library -->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>29.0-jre</version>
        </dependency>

    </dependencies>
</project>
```

And with that, Google Guava is available in your project and you can use classes from it.


If you run `mvn dependency:tree`, the output confirms the inclusion.

{{< rawhtml >}}
<pre class="code-output">
-> mvn dependency:tree
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------< io.codejournal.maven:dependency-management-simple >----------
[INFO] Building dependency-management-simple 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ dependency-management-simple ---
[INFO] io.codejournal.maven:dependency-management-simple:jar:1.0
[INFO] +- com.google.guava:guava:jar:29.0-jre:compile
[INFO]    +- com.google.guava:failureaccess:jar:1.0.1:compile
[INFO]    +- com.google.guava:listenablefuture:jar:9999.0-empty-to-avoid-conflict-with-guava:compile
[INFO]    +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO]    +- org.checkerframework:checker-qual:jar:2.11.1:compile
[INFO]    +- com.google.errorprone:error_prone_annotations:jar:2.3.4:compile
[INFO]    \- com.google.j2objc:j2objc-annotations:jar:1.3:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.889 s
[INFO] Finished at: 2020-09-27T22:49:14+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}
