---
date: 2020-08-27
linktitle: Maven Dependency Types - Direct and Transitive
next: /2020/08/maven-dependency-scopes/
prev: /2020/08/maven-co-ordinates-and-depedendency-management/
title: Maven Dependency Types - Direct and Transitive
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

In this article, we will discuss the Maven Dependency Types - **Direct** and **Transitives**. These are discussed in detail in the following video as well.

{{< yt "https://www.youtube.com/embed/JkdWM54gBKc" >}}





## Transitivity

Before discussing the dependency types, you need to understand what is **"Transitivity"**.

We all have learned "Transitivity" in Mathematics back in our school days. What transivity indicates is 

{{< rawhtml >}}
<div class="notification">

    <p>If there are three objects <code>A</code>, <code>B</code> and <code>C</code>, such that, <code>A</code> is related to <code>B</code>, and <code>B</code> is related to <code>C</code>, then we can conclude that <code>A</code> is also related to <code>C</code>.</p>

    <p>In other words, we can say, <code>A</code> is transitively related to <code>C</code>.</p>
</div>
{{< /rawhtml >}}

Below is a quick diagram depicting that.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2020/08/005-maven-dependency-types-direct-transitive/maven-transitivity.png" alt="Maven Transitivity" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Dependency Types

While working with Maven dependencies, you will deal with either **Direct** dependencies or **Transitive** dependencies.


### Direct Dependencies
These dependencies are configured directly in your Project Object Model(pom.xml). You explicity state which dependency your project needs by providing their co-ordinates(`groupId, artifactId, version`).

### Transitive Dependencies
Transitive Dependencies are the ones that you DO NOT configure in your Project Object Model(pom.xml), and gets pulled because your Direct dependencies rely on those.





{{< articlead >}}

## Lets understand this with an example.

Below is a dependency configuration for [junit](https://junit.org/junit5/) that you will put in `pom.xml` to add it to your project

```xml
...
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.11</version>
</dependency>
...

```

Because, we have configured `junit` directly in our `pom.xml`, it will be our **"Direct"** dependency. So, maven will download the junit jar file and add it to your project.

{{< rawhtml >}}
<div class="notification">
    But will it only add the junit jar file? Let's dig a little deeper.
</div>
{{< /rawhtml >}}



You can use [`maven-dependency-plugin`](https://maven.apache.org/plugins/maven-dependency-plugin/) to get the list of all dependencies of your project. The command you need to run is `mvn dependency:tree`.

Here is the output from a project that has just `junit` added to its `pom.xml`.


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
[INFO] \- junit:junit:jar:4.11:compile
[INFO]    \- org.hamcrest:hamcrest-core:jar:1.3:compile


[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.004 s
[INFO] Finished at: 2020-10-08T21:13:06+02:00
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

I have split the relevant section of the `maven-dependency-plugin` in the output to make things stand out.

If you look closely the dependency tree, the project `dependency-management-simple` has `junit` as its dependency, but `junit` itself needs `hamcrest-core`, which is a sub-tree under `junit`.

This `hamcrest-core` is thus, your **"Transitive"** dependency for project `dependency-management-simple`. It got included without you specifying it explicitly, because `junit` is in-turn dependent on it.

Here they are annotated in the logs.

{{< rawhtml >}}
<pre class="code-output">
[INFO] io.codejournal.maven:dependency-management-simple:jar:1.0    # PROJECT
[INFO] \- junit:junit:jar:4.11:compile                              # \- Direct Dependency
[INFO]    \- org.hamcrest:hamcrest-core:jar:1.3:compile             #    \- Transitive Dependency
</pre>
{{< /rawhtml >}}
