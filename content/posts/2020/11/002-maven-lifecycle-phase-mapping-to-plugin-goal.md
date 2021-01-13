---
date: 2020-11-21
linktitle: Maven Lifecycle Phase Mapping to Plugin Goals
next: /2020/12/generating-code-using-maven-java-to-xsd/
prev: /2020/11/maven-lifecycles/
title: Maven Lifecycle Phase Mapping to Plugin Goals
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In the [previous article](/2020/11/maven-lifecycles/), we discussed about the Maven Lifecycles and the phases in them. We also got to know that Maven provides default plugin mappings to make builds faster without too much configuration. However, many a times, the build steps need to be customized to fulfill the needs of the project. That is when you need to map plugins to lifecycle phases to make your builds work for your requirements.

Here is the video showing how you can configure a plugin to a Lifecycle Phase.

{{< yt "https://www.youtube.com/embed/3FfWf50HYZU" >}}





## The Project

The initial project used in this article is [available here][1].

Here are the contents of the Project

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── io
    │           └── codejournal
    │               └── maven
    │                   └── App.java
    └── test
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── AppTest.java
</pre>
{{< /rawhtml >}}

Nothing too fancy, just `App.java` which is a Hello World project in Java, and `AppTest.java` is a very simple Junit test class.

The main file that you will be working in the `pom.xml`. Here is how it looks.

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.codejournal.maven</groupId>
    <artifactId>custom-lifecycle-mapping</artifactId>
    <version>1.0</version>
    <name>custom-lifecycle-mapping</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.7.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

It has project coordinates, some properties and Junit test dependency.





{{< articlead >}}

## Status Quo

To keep things simple, let's customize the lifecycle - [`clean`][2].

It has just three phases - `pre-clean, clean` and `post-clean`. Maven provides the default plugin mapping of the `clean` phase via the [maven-clean-plugin][3].

Before we start customizing the build, let's run the clean lifecycle to see what output we get. To run all the phases of the clean lifecycle, we need to run.

```
mvn post-clean
```

And here is the output.

{{< rawhtml >}}
<pre class="code-output">
-> mvn post-clean
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------< io.codejournal.maven:custom-lifecycle-mapping >------------
[INFO] Building custom-lifecycle-mapping 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ custom-lifecycle-mapping ---
[INFO] Deleting /home/codejournal/workspace/maven-for-beginners-custom-lifecycle-mappings/target
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.190 s
[INFO] Finished at: 2020-11-21T21:21:19+01:00
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

As expected, since only the `clean` phase is mapped, we get output from the [maven-clean-plugin][3].





{{< articlead >}}

## Customizing Build

We first need the plugin that we need to run. Let's go with a simple command that prints something when the phase is executed. Back in the old days, Apache Ant was used to run the builds. There are huge set of [ant tasks][4] which you can use to run all sorts of commands for your build purposes. Its a little old way of doing things, but it works well; very well in fact. One of these tasks is the [`echo`][5] tasks that prints on the console.

[maven-antrun-plugin][6] was created for Maven to run the Ant tasks. It has just one goal `run` that executes whatever ants tasks are configured.

The plugin needs to be configured as [`<plugin>`][7] in the [`<build>`][8] section.

The [`<execution>`][9] section is where all the things come together. It has `<phase>` which tells which phase you need the plugin to bind to do and `<goals>` configures the goal which needs to be executed.

Now, we need to configure our [`echo`][5] task. That is part of `<configuration>` section. Having put all the values, the `pom.xml` will now look like this.

Here is how the configuration looks.

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>echo-pre-clean</id>
                    <phase>pre-clean</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <target>
                            <echo message="Hello from 'pre-clean' Phase" level="info" />
                        </target>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

If you run this now, you will get output for `pre-clean` and `clean` phases since those are the ones which are mapped.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------< io.codejournal.maven:custom-lifecycle-mapping >------------
[INFO] Building custom-lifecycle-mapping 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-antrun-plugin:3.0.0:run (echo-pre-clean) @ custom-lifecycle-mapping ---
[INFO] Executing tasks
[INFO]      [echo] Hello from 'pre-clean' Phase
[INFO] Executed tasks
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ custom-lifecycle-mapping ---
[INFO] Deleting /home/codejournal/workspace/maven-for-beginners-custom-lifecycle-mappings/target
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.472 s
[INFO] Finished at: 2020-11-22T01:17:18+01:00
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

We can similarly configure multiple `<execution>` and link them to other phases of clean lifecycle. Here is the `pom.xml` with these configured for all phases. 

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>echo-pre-clean</id>
                    <phase>pre-clean</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <target>
                            <echo message="Hello from 'pre-clean' Phase" level="info" />
                        </target>
                    </configuration>
                </execution>
                <execution>
                    <id>echo-clean</id>
                    <phase>clean</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <target>
                            <echo message="Hello from 'clean' Phase" level="info" />
                        </target>
                    </configuration>
                </execution>
                <execution>
                    <id>echo-post-clean</id>
                    <phase>post-clean</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <target>
                            <echo message="Hello from 'post-clean' Phase" level="info" />
                        </target>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

We added another mapping for `clean` phase. Now, Maven will be executing the default [maven-clean-plugin][3] goal as well as the one which we configured. If we run, `mvn post-clean` now, we see all our [`echo`][5] tasks executed.


{{< rawhtml >}}
<pre class="code-output">
-> mvn post-clean
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------< io.codejournal.maven:custom-lifecycle-mapping >------------
[INFO] Building custom-lifecycle-mapping 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-antrun-plugin:3.0.0:run (echo-pre-clean) @ custom-lifecycle-mapping ---
[INFO] Executing tasks
[INFO]      [echo] Hello from 'pre-clean' Phase
[INFO] Executed tasks
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ custom-lifecycle-mapping ---
[INFO] Deleting /home/codejournal/workspace/maven-for-beginners-custom-lifecycle-mappings/target
[INFO] 
[INFO] --- maven-antrun-plugin:3.0.0:run (echo-clean) @ custom-lifecycle-mapping ---
[INFO] Executing tasks
[INFO]      [echo] Hello from 'clean' Phase
[INFO] Executed tasks
[INFO] 
[INFO] --- maven-antrun-plugin:3.0.0:run (echo-post-clean) @ custom-lifecycle-mapping ---
[INFO] Executing tasks
[INFO]      [echo] Hello from 'post-clean' Phase
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.570 s
[INFO] Finished at: 2020-11-22T01:23:03+01:00
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}


The [working code][10] is available on github for reference.


 [1]: https://github.com/the-code-journal/maven-for-beginners/raw/main/002-custom-lifecycle-mappings/maven-for-beginners-custom-lifecycle-mappings.zip
 [2]: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.tml#lifecycle-reference
 [3]: https://maven.apache.org/plugins/maven-clean-plugin/
 [4]: https://ant.apache.org/manual/tasksoverview.html
 [5]: https://ant.apache.org/manual/Tasks/echo.html
 [6]: https://maven.apache.org/plugins/maven-antrun-plugin/
 [7]: https://maven.apache.org/pom.html#plugins
 [8]: https://maven.apache.org/pom.html#build
 [9]: https://maven.apache.org/guides/mini/guide-configuring-plugins.html#configuring-build-plugins
[10]: https://github.com/the-code-journal/maven-for-beginners/tree/main/002-custom-lifecycle-mappings/final