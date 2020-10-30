---
date: 2020-09-03
linktitle: Maven Dependency Scopes
next: /2020/09/maven-project-object-model/
prev: /2020/08/maven-dependency-scopes/
title: Maven Project Structure
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

Maven uses convention based approach, and hence, it defines a standard directory structure for the project to organize the files.

Familiarity with this structure is important for understanding how the project files are used by Maven to build and assemble the project.

Here is the video explaining these.

{{< yt "https://www.youtube.com/embed/NkeoI3qTDrU" >}}





## But why?

But, why limit ourselves to one project structure?

Because, this discipline allows you to be more productive. It might take a bit of time for you to get used to this structure, but the effort is worth it.

Prior to Maven, projects had very varied directory structure and if you worked on multiple projects, keeping track of files across these projects, would have been really difficult.

Maven made all of this simpler, by having a consistent project structure across projects. So, even if you moved to a different organization or worked on an open source project, you will feel completely at home and navigating the project files would be a walk in the park.

With this consistent project structure, Maven plugins can easily assume the location of certain files that they need for execution. That means you can just include the plugin and they will just work, out of the box. You don’t need to configure anything specific for most cases. It really, really reduces the time to get your builds running.

Let’s now look at a standard directory structure for a Maven Project.





{{< articlead >}}

## Maven Configuration File - pom.xml

The first and foremost, maven configuration file - `pom.xml`. This should be located in your project’s root directory or project's home directory.

In this directory, you also will have `src` folder that will contain all the files that, Maven plugins will use.

Below, is the example of the project's home directory.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree . -L 1
.
├── pom.xml
└── src
</pre>
{{< /rawhtml >}}

Let’s see how `src` folder is laid out, to store the files.





{{< articlead >}}

## Source Code Files

For all of your Application code files, they need to be placed inside the `main` directory of `src`. Within the `main` directory, you need to put the files inside the language specific folder.

So, your java files will be stored in `src/main/java` and your `scala` or `kotlin` files will be stored at `src/main/scala` or `src/main/kotlin` respectively.

Within each of these directories, you have your regular package structure, that is unique to your application.

Here is an example.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    |  +-----------------------------------+
    └──|── main                            |
       |  ├── java                         |
       |  │   └── io                       |
       |  │       └── codejournal          |
       |  │           └── maven            |
       |  │               └── App.java     |
       |  ├── kotlin                       |
       |  └── scala                        |
       +-----------------------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Application Config Files

For storing the application configuration files, like property configs, yaml configs, xmls configs, or any other file that your application needs to use from classpath when its run, you need to store those in `resources` folder inside `main` directory.

So, full path is `src/main/resources`.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    └───── main
          ├── java
          ├── kotlin
          ├── scala
       +--|--------------------------------+ 
       |  └── resources                    |
       |    └── application.properties     |
       +-----------------------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Web Application Files

If you working on a web application, you need to store your htmls, css, javascript, jsp, etc. They are stored in `src/main/webapp`. The directory structure inside `webapp` should confirm to Java Servlet Specification so that these can be packaged correctly and gets deployed correctly on Application servers.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    └───── main
          ├── java
          ├── kotlin
          ├── scala
          └── resources
       +--|--------------------------------+
       |  └── webapp                       |
       |      ├── META-INF                 |
       |      ├── resources                |
       |      │   ├── css                  |
       |      │   ├── images               |
       |      │   └── js                   |
       |      └── WEB-INF                  |
       |          ├── jsp                  |
       |          └── web.xml              |
       +-----------------------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Application Test Code Files

When we talk about files for testing, Maven expects these to be stored in a separate directory - `test` inside `src` folder.

And for Test class files for your source classes, the structure is similar to files under `src/main`. So, if you have application Java class files under `src/main/java`, then you need to place your test classes for these at `src/test/java`.

Similarly, for other language class files for scala, kotlin and groovy.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    ├───── main
    |     ├── java
    |     ├── kotlin
    |     ├── scala
    |     ├── resources
    |     └── webapp
    |  +--------------------------------------+
    └──|─ test                                |
       |  ├── java                            |
       |  |   └── io                          |
       |  |       └── codejournal             |
       |  |           └── maven               |
       |  |               └── AppTest.java    |
       |  ├── kotlin                          |
       |  └── scala                           |
       +--------------------------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Application Test Config Files

And if you require any test specific configuration files, then you need to store those in `resources` folder in your `src/test` folder. Again, similar to what the structure is for `src/main`.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    ├───── main
    |     ├── java
    |     ├── kotlin
    |     ├── scala
    |     ├── resources
    |     └── webapp
    |
    └──── test
          ├── java
          ├── kotlin
          ├── scala
       +--|-------------------------------------+ 
       |  └── resources                         |
       |    └── application-test.properties     |
       +----------------------------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Integration Test Code Files

If you have some integration tests that maven needs to run, Maven expects you to store these in a separate directory inside `src`. It is `src/it`, short form for integration test.

The directory structure of integration test folder is exactly similar to that of `src/main` or `src/test`. You get language specific folders to store your integration test class files and an additional resources folder to store any configuration files that you might need during the integration tests.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    ├───── main
    ├───── test
    |  +-----------------------+ 
    └──|─ it                   |
       |  ├── java             |
       |  ├── kotlin           |
       |  ├── scala            |
       |  |                    |
       |  └── resources        |
       +-----------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Generating a Site

As you know, Maven can generate a project site for your application. To facilitate this, maven expects you to store these files in a separate folder called `site` inside the `src` folder.

The `site` directory contains a `site.xml` that defines how Maven should build the site. To store the site pages, maven provides different folders for each supported markup language.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    ├───── main
    ├───── test
    ├───── it
    |  +----------------------+ 
    └──|─ site                |
       |  ├── apt             |
       |  ├── fml             |
       |  ├── markdown        |
       |  ├── xdoc            |
       |  |                   |
       |  └── site.xml        |
       +----------------------+
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Assembly Files

And finally, `assembly` directory that tells Maven how to assemble your project. Without specifying any assembly configuration, maven assembles the project based on the default packaging from `pom.xml`.

But if you want to assemble your project differently, you can define the configuration in the `assembly` directory, and provide the configuration file to the maven-assembly plugin.

{{< rawhtml >}}
<pre class="code-output">
[codejournal@codejournal-box ~/workspace/maven-simple-project]
-> tree .
.
├── pom.xml
└── src
    ├───── main
    ├───── test
    ├───── it
    ├───── site
    |  +-------------------------+ 
    └──|─ assemby                |
       |  └── distribution.xml   |
       +-------------------------+
</pre>
{{< /rawhtml >}}
