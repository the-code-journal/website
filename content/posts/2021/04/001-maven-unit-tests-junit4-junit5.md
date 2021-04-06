---
date: 2021-04-06
linktitle: Unit Testing in Maven - Migrating from Junit4 to Junit5 Tests
next: /2020/08/install-apache-maven-on-linux/
prev: /2021/03/unit-testing-in-maven-junit5-tests/
title: Unit Testing in Maven - Migrating from Junit 4 to Junit5 Tests
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


In our [previous article][1], we looked at configuring and running [Junit 5][2] tests with Maven. If you are starting a new project today, then you can already use the latest Junit 5, which is stable and has lots of support from the community.

However, for projects that have been around for about 2-3 years or more, it's quite likely that they are using [Junit 4][3] tests. [Junit 5][2] team provides a very convenient approach where you can easily include the [Junit 5][2] libraries in your project and start using it. This allows you to slowly migrate your [Junit 4][3] test to [Junit 5][2], and eventually migration to [Junit 5][2] completely.

Below is the video with the Demo to configure [Junit 5][2] along with [Junit 4][3] tests in your Maven project.

{{< yt "https://www.youtube.com/embed/8oOC51bcuCY" >}}





## The Project

Here is the [starting project][4]. It is the same project from [configuring Junit 4 article][5]. Let's look at the files as a recap.

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
    │                   └── unittests
    │                       └── junit4andjunit5
    │                           └── Runner.java
    └── test
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── unittests
                            └── junit4andjunit5
                                └── RunnerTest.java
</pre>
{{< /rawhtml >}}

First, `pom.xml`. It has [Junit 4][3] dependency and [maven-surefire-plugin][6] configured.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.codejournal.maven</groupId>
  <artifactId>maven-unit-tests-junit4-and-junit5</artifactId>
  <version>1.0</version>
  <name>maven-unit-tests-junit4-and-junit5</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
      </plugin>
    </plugins>
  </build>

</project>
```

`Runner.java` contains a method `sum` which takes two integers and returns their sum.

```java
package io.codejournal.maven.unittests.junit4andjunit5;

import static java.lang.String.format;

public class Runner {

    public static int sum(final int number1, final int number2) {
        return number1 + number2;
    }

    public static void main(String[] args) {
        System.out.println(format("sum(%d, %d) => %d", 5, 10, sum(5, 10)));
    }
}
```

`RunnerTest.java` has [Junit 4][3] test configured with annotations - `@Before, @After` and `@Test`.

```java
package io.codejournal.maven.unittests.junit4andjunit5;

import static org.junit.Assert.assertEquals;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class RunnerTest {

    @Before
    public void setUp() {
        System.out.println("\nRunning setUp...");
    }

    @Test
    public void sumReturnsCorrectResult() {

        System.out.println("Running sumReturnsCorrectResult...");

        final int number1 = 3;
        final int number2 = 5;

        final int expected = 8;

        final int actual = Runner.sum(number1, number2);

        assertEquals(expected, actual);
    }

    @After
    public void tearDown() {
        System.out.println("Running tearDown...\n");
    }
}
```





{{< articlead >}}

## Configuring Junit 5

At the time of writing this article, the latest GA version is [5.7.1][7]. As already mentioned in our [previous article][1], in [Junit 5][2], you just need the [`junit-jupiter-api`][7] in your project to configure the tests. The engine that runs these tests will be be provided by [Maven Surefire Plugin][6]. Let's configure this.

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.7.1</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```





## Writing the JUnit 5 test

Let's rename the `RunnerTest.java` to `RunnerJunit4Test.java` to indicate the [Junit 4][3] test class. And then, create another copy as `RunnerJunit5Test.java`.

{{< rawhtml >}}
<div class="notification">
    You don't need to do this for your existing projects that are in your company or personal projects. It is purely done here for aesthetics of this article.
</div>
{{< /rawhtml >}}


The project structure looks like this now.

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
    │                   └── unittests
    │                       └── junit4andjunit5
    │                           └── Runner.java
    └── test
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── unittests
                            └── junit4andjunit5
                                ├── RunnerJunit4Test.java
                                └── RunnerJunit5Test.java
</pre>
{{< /rawhtml >}}


Let's make changes to the `RunnerJunit5Test.java`.

Here are the changes that we need to do.
- `@Before` needs to be replaced with [`@BeforeEach`][8] in `org.junit.jupiter.api` package.
- `@Test` needs to be replaced with [`@Test`][9] in `org.junit.jupiter.api` package.
- `@After` needs to be replaced with [`@AfterEach`][10] in `org.junit.jupiter.api` package.
- `Assert.assertEquals` needs to be replaced with `Assertions.assertEquals` in `org.junit.jupiter.api` package.

And those are all the changes for our test class. Here is the completed test class.

```java
package io.codejournal.maven.unittests.junit4andjunit5;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class RunnerJunit5Test {

    @BeforeEach
    public void setUp() {
        System.out.println("\nRunning setUp...");
    }

    @Test
    public void sumReturnsCorrectResult() {

        System.out.println("Running sumReturnsCorrectResult...");

        final int number1 = 3;
        final int number2 = 5;

        final int expected = 8;

        final int actual = Runner.sum(number1, number2);

        assertEquals(expected, actual);
    }

    @AfterEach
    public void tearDown() {
        System.out.println("Running tearDown...\n");
    }
}
```





{{< articlead >}}

## Running the Tests - Junit4 and Junit5

And that should be it. If you run the tests - `mvn clean test`, you can see that [Maven Surefire Plugin][6] runs both the tests.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] ------< io.codejournal.maven:maven-unit-tests-junit4-and-junit5 >-------
[INFO] Building maven-unit-tests-junit4-and-junit5 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] Deleting /home/codejournal/workspace/maven-unit-tests-junit4-and-junit5/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit4-and-junit5/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-junit4-and-junit5/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit4-and-junit5/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /home/codejournal/workspace/maven-unit-tests-junit4-and-junit5/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ maven-unit-tests-junit4-and-junit5 ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.unittests.junit4andjunit5.RunnerJunit5Test

Running setUp...
Running sumReturnsCorrectResult...
Running tearDown...

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.037 s - in io.codejournal.maven.unittests.junit4andjunit5.RunnerJunit5Test
[INFO] Running io.codejournal.maven.unittests.junit4andjunit5.RunnerJunit4Test

Running setUp...
Running sumReturnsCorrectResult...
Running tearDown...

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.008 s - in io.codejournal.maven.unittests.junit4andjunit5.RunnerJunit4Test
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.315 s
[INFO] Finished at: 2021-04-07T03:10:29+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Specifying Engine in Surefire Plugin

While in majority of cases, [Maven Surefire Plugin][6] should be able to run both [Junit 4][3] and [Junit 5][2] tests successfully, without any additional configuraion.

However, if there is a specific version of Engine that has some specific feature that you want to use, or there is a bug which is resolved in the latest version of engine, you can configure the engine for [Surefire Plugin][6].

Here is how you can specify the [Junit 4 Vintage Runner][11] and [Junit 5 Jupiter][12] Engine.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.0.0-M5</version>
      <dependencies>
        <dependency>
          <groupId>org.junit.jupiter</groupId>
          <artifactId>junit-jupiter-engine</artifactId>
          <version>${junit5.version}</version>
        </dependency>
        <dependency>
          <groupId>org.junit.vintage</groupId>
          <artifactId>junit-vintage-engine</artifactId>
          <version>${junit5.version}</version>
        </dependency>
      </dependencies>
    </plugin>
  </plugins>
</build>
```



The [working code][13] is available on github for reference.


  [1]: /2021/03/unit-testing-in-maven-junit5-tests/
  [2]: https://junit.org/junit5/
  [3]: https://junit.org/junit4/
  [4]: https://github.com/the-code-journal/maven-for-beginners/tree/main/012-unit-tests-junit4-junit5/initial
  [5]: /2021/03/unit-testing-in-maven-junit4-tests/
  [6]: https://maven.apache.org/surefire/maven-surefire-plugin/index.html
  [7]: https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api
  [8]: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/BeforeEach.html
  [9]: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Test.html
 [10]: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/AfterEach.html
 [11]: https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine
 [12]: https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine
 [13]: https://github.com/the-code-journal/maven-for-beginners/tree/main/012-unit-tests-junit4-junit5/final
