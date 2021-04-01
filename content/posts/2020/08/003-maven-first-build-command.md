---
date: 2020-08-04
linktitle: First Build with Apache Maven - Command Prompt
next: /2020/08/maven-co-ordinates-and-depedendency-management/
prev: /2020/08/install-apache-maven-on-mac/
title: First Build with Apache Maven - Command Prompt
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

This article demonstrates the very first build with [Apache Maven][1]. In this article, we will use command prompt to create our simple Maven project and then use `mvn` command to build it

You can watch the entire demo in this video.

{{< yt "https://www.youtube.com/embed/OwcnECUdFro" >}}





## Pre-requisite

You need to have [Apache Maven][1] already installed on your system. If you don't have Maven installed, you can install it by following steps from our previous articles:
- [Installing Maven on Linux][2]
- [Installing Maven on Mac][3]
- [Installing Maven on Windows][4]

You can also watch the videos here.

{{< rawhtml >}}
<div style="display: flex; justify-content: space-between">
    <div style="width: 48%; display: inline-block;">
    {{< yt "https://www.youtube.com/embed/JgFXdAmWoxQ" >}}
    </div>
    <div style="width: 48%; display: inline-block;">
    {{< yt "https://www.youtube.com/embed/NoDHwW9guL8" >}}
    </div>
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Creating a Simple Maven Project

In this tutorial, we will create the Maven project from the command prompt using the `mvn` command. This basic project scaffolding is done using the [maven-archetype-plugin][5]. This plugin the one that is used by the IDEs under the hood to create the Project. 

The archetype plugin has a numerous template projects to chose from and also from various sources. If you type simple `mvn archetype:generate`, you will be presented with 2800+
various project templates. You can use the filter option to limit the search and chose, but in most cases, it is preferable to use an IDE.

The Archetype plugin works in interactive and non-interactive mode. In the interactive mode, you need to select the project template from which the project should be created and then you need to provide the maven coordinates.

If you already know the archetype, it is advisable to use the non-interactive mode. This is th done which we will use in this tutorial.

We will be using the non-interactive mode for our project.

```
mvn archetype:generate \
    -DinteractiveMode=false \
    -DarchetypeGroupId=org.apache.maven.archetypes \
    -DarchetypeArtifactId=maven-archetype-simple \
    -DgroupId=io.codejournal.maven \
    -DartifactId=maven-simple-project \
    -Dversion=1.0.0-SNAPSHOT
```

{{< rawhtml >}}
<div class="notification">
    If you are running this for the very first time, then Maven will download the plugin jars from the Maven central, and you will see those in your console. Don't worry, these will be just downloaded just once and occassionally when there is a new update to the plugin that is being executed.
</div>
{{< /rawhtml >}}

The command is a long one, so let's dissect it one at a time.



#### `archetype:generate`

This is a goal of archetype plugin that is responsible for generating for the project. Maven's plugin execution follows this convention `plugin:goal` to execute the various executions goals provided by the plugin. A full list of archetype plugin's goals are [listed here][6].

Because, we need to create a project from a template, we use the `generate` goal.



#### `-DinteractiveMode=false`

Property arguments to Maven are provided using `-D` prefix. In our case, we are providing a property `interactiveMode` which is used by archetype plugin. This tells the archetype plugin that it should not prompt the user for any mandatory value that is not provided. It will instead error out mentioning which value was missing. In the interactive mode, Maven will use the values that you have provided and if any value is missing, it will ask you to provide them on the prompt.

Since, we are providing all the mandatory values as property arguments, we are disabling the interactive mode by `interactiveMode=false`.



#### `-DarchetypeGroupId=org.apache.maven.archetypes`

This property tells the Archetype plugin to select the groupId of the archetype that need to be used to create the project. There can be multiple organizations or teams that are creating their own archetypes, and they could clash. The `archetypeGroupId` gives the ability to avoid such collitions.

For example, if you type `mvn archetype:generate`, and when the plugin is asking you to provide the archetype id or filter option, type `maven-archetype-simple`. You will see that there are two archetypes that define the same project. Here is the output.


{{< rawhtml >}}
<pre class="code-output">
-> mvn archetype:generate
...
...
2798: remote -> za.co.absa.hyperdrive:component-archetype (-)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 1674: maven-archetype-simple
Choose archetype:
1: remote -> com.haoxuer.maven.archetype:maven-archetype-simple (a simple maven archetype)
2: remote -> org.apache.maven.archetypes:maven-archetype-simple (An archetype which contains a simple Maven project.)
</pre>
{{< /rawhtml >}}

As you can see, the first archetype has `archetypeGroupId=com.haoxuer.maven.archetype` and second one has `archetypeGroupId=org.apache.maven.archetypes`. To allow archetype plugin to chose precisely which of this needs to be used, you should specify `archetypeGroupId`.

{{< rawhtml >}}
<div class="notification">
    If you don't specify <code>archetypeGroupId</code> and there are multiple archetypes, the archetype plugin will use the first one from the list.
</div>
{{< /rawhtml >}}


Now, that the project is created, let's see the files that have been created. A quick run of `tree` command shows the following output.

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
    ├── test
    │   └── java
    │       └── io
    │           └── codejournal
    │               └── maven
    │                   └── AppTest.java
    └── site
        └── site.xml
</pre>
{{< /rawhtml >}}

`App.java` is a very simple java program with hello world, and `AppTest.java` is a Junit test class file. The `site.xml` contains a sample building of Project site.





{{< articlead >}}

## Building the Project

Maven build contains of different lifecycle phases, and depending on your requirements, you specify the goals to execute. A typical Maven build consists of two goals - `clean package`.

`clean` tells Maven to delete the files created from previous builds, while `package` tells Maven to compile, test, and package the project.

Here if the output when you run `mvn clean package`.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean package
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------< io.codejournal.maven:maven-simple-project >--------------
[INFO] Building maven-simple-project 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ maven-simple-project ---
[INFO] Deleting /tmp/maven-simple-project/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.2:resources (default-resources) @ maven-simple-project ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /tmp/maven-simple-project/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.0:compile (default-compile) @ maven-simple-project ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /tmp/maven-simple-project/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.2:testResources (default-testResources) @ maven-simple-project ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /tmp/maven-simple-project/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.0:testCompile (default-testCompile) @ maven-simple-project ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /tmp/maven-simple-project/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.1:test (default-test) @ maven-simple-project ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 s - in io.codejournal.maven.AppTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ maven-simple-project ---
[INFO] Building jar: /tmp/maven-simple-project/target/maven-simple-project-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.872 s
[INFO] Finished at: 2020-09-15T20:26:18+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

{{< rawhtml >}}
<div class="notification">
    If you are running <code>mvn clean package</code> for the very first time, you will see Maven download a lot of plugin jar files, just like the archetype plugin execution earlier. Everytime, there is a new plugin that Maven executes, or an existing plugin for which there is a new updated, Maven will download the new jar files for the first time.
</div>
{{< /rawhtml >}}

And with that, your first Maven build is complete.


  [1]: https://maven.apache.org/
  [2]: /2020/08/install-apache-maven-on-linux/
  [3]: /2020/08/install-apache-maven-on-mac/
  [4]: /2020/08/install-apache-maven-on-windows/
  [5]: https://maven.apache.org/archetype/maven-archetype-plugin/
  [6]: https://maven.apache.org/archetype/maven-archetype-plugin/plugin-info.html
