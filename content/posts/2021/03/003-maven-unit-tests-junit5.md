---
date: 2021-03-31
linktitle: Unit Testing in Maven - Junit5 Tests
next: /2021/04/unit-testing-in-maven-migrating-from-junit-4-to-junit5-tests/
prev: /2021/03/unit-testing-in-maven-junit4-tests/
title: Unit Testing in Maven - Junit5 Tests
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


In our [previous article][1], we saw how Junit4 tests can be configured and run with [Maven Surefire plugin][2]. As I already mentioned in that article, [Junit 5][3] is the latest version of Junit. It gets extensive with more features and different testing styles with [Junit 5][3] and that is why there is a shift in the way it is configured in projects.

Below is the video with the Demo to running [Junit 5][3] tests using [Maven Surefire Plugin][2].

{{< yt "https://www.youtube.com/embed/K7k3YkerakM" >}}





### Junit 5

[JUnit 5][3] is composed of several different modules from three different sub-projects.

```
JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
```

### Junit Platform

Junit platform is the set of interfaces and classes that will becomes the standard for launching tests on JVM. It provides [`TestEngine`][4] interface that any library can use to implement their own test running engine. This allows to build any custom engine that developers/teams find befitting.


### Junit Jupiter
JUnit Jupiter provides [`TestEngine`][4] for running tests for Junit 5 on the platform.

### Junit Vintage
JUnit Vintage provides a [`TestEngine`][4] for running JUnit 3 and JUnit 4 based tests on the platform.

{{< rawhtml >}}
<div class="notification">
    JUnit 5 requires Java 8 (or higher) at runtime. However, you can still test code that has been compiled with previous versions of the JDK.
</div>
{{< /rawhtml >}}





{{< articlead >}}

## The Project

Here is the [starting project][5]. It is the same project from [previous article][1]. Let's look at the files as a recap.

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
    │                       └── junit5
    │                           └── Runner.java
    └── test
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── unittests
                            └── junit5
                                └── RunnerTest.java
</pre>
{{< /rawhtml >}}

First, `pom.xml`. It has Junit 4 dependency and maven-surefire-plugin configured.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.codejournal.maven</groupId>
  <artifactId>maven-unit-tests-junit5</artifactId>
  <version>1.0</version>
  <name>maven-unit-tests-junit5</name>

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
package io.codejournal.maven.unittests.junit5;

import static java.lang.String.format;

public class Runner {

    public static int sum(final int number1, final int number2) {
        return number1 + number2;
    }

    public static void main(final String[] args) {
        System.out.println(format("sum(%d, %d) => %d", 5, 10, sum(5, 10)));
    }
}
```

`RunnerTest.java` has Junit 4 test configured with annotations - `@Before, @After` and `@Test`.

```java
package io.codejournal.maven.unittests.junit5;

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

At the time of writing this article, the latest GA version is [5.7.1][6]. Let's first define a property for Junit version.

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>11</maven.compiler.source>
  <maven.compiler.target>11</maven.compiler.target>
  <junit.version>5.7.1</junit.version>
</properties>
```

To configure Junit 5, we need to configure the dependencies - [`junit-jupiter-api`][6] and [`junit-jupiter-engine`][7]. You can also configure [`junit-jupiter`][8] directly, which pulls both of these dependecies.

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

or

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```




## Updating Tests

With Junit 5, the package structure is changed. Below, you can see it compared against each other

```
+----------------------------------------------------------+---------------+
|         Junit 5                                          |    Junit 4    |
+----------------------------------------------------------+---------------+
| - API            - org.junit.jupiter.api.*               | org.junit.*   |
| - Jupiter Engine - org.junit.jupiter.engine.*            |               |
| - Vintage Engine - org.junit.jupiter.migrationsupport.*  |               |
+----------------------------------------------------------+---------------+
```

Also, the annotation in Junit 5 are changed. Here is the [full list of annotations][9] in Junit 5. For our test, we have changes for [`@Before`][10] and [`@After`][11]. The corresponding equivalents in Junit 5 are [`@BeforeEach`][12] and [`@AfterEach`][13] which are more explicit and readable. Let's update our test class to update the imports and update the annotations.

```java
package io.codejournal.maven.unittests.junit5;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.AfterEach;                 // Replaces org.junit.After
import org.junit.jupiter.api.BeforeEach;                // Replaces org.junit.Before
import org.junit.jupiter.api.Test;                      // Replaces org.junit.Test

public class RunnerTest {

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

And with that, your Test class is now conforming to Junit 5. You can run the maven build - `mvn clean test` and the tests should be running.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------< io.codejournal.maven:maven-unit-tests-junit5 >------------
[INFO] Building maven-unit-tests-junit5 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-unit-tests-junit5 ---
[INFO] Deleting /home/codejournal/workspace/maven-unit-tests-junit5/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-unit-tests-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit5/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-unit-tests-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-junit5/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-unit-tests-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit5/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-unit-tests-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-junit5/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ maven-unit-tests-junit5 ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.unittests.junit5.RunnerTest

Running setUp...
Running sumReturnsCorrectResult...
Running tearDown...

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.041 s - in io.codejournal.maven.unittests.junit5.RunnerTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.020 s
[INFO] Finished at: 2021-03-31T21:32:16+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Smart Resolution of TestEngine by Surefire Plugin

With Junit 5, you don't need to have the [`junit-jupiter-engine`][7] dependency. Your tests, for most use cases, will only use classes from [`junit-jupiter-api`][6] dependency. It is the responsibility of the build framework to chose the engine and run the tests. Surefire plugin also suggests to use this in their [documentation page][14].

Essentially, you only configure [`junit-jupiter-api`][6] as your test dependency, and configure [`junit-jupiter-engine`][7] as part of plugin dependency so that Surefire plugin can use the new Junit 5 engine to run the tests.

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
</dependencies>

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
          <version>${junit.version}</version>
        </dependency>
      </dependencies>
    </plugin>
  </plugins>
</build>
```

Now, if you run the tests, you will still have them executed, but your code doesn't have any indication which engine was used.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------< io.codejournal.maven:maven-unit-tests-junit5 >------------
[INFO] Building maven-unit-tests-junit5 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-unit-tests-junit5 ---
[INFO] Deleting /home/codejournal/workspace/maven-unit-tests-junit5/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-unit-tests-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit5/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-unit-tests-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-junit5/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-unit-tests-junit5 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-junit5/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-unit-tests-junit5 ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-junit5/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ maven-unit-tests-junit5 ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.unittests.junit5.RunnerTest

Running setUp...
Running sumReturnsCorrectResult...
Running tearDown...

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.043 s - in io.codejournal.maven.unittests.junit5.RunnerTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.120 s
[INFO] Finished at: 2021-03-31T21:44:54+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}



The [working code][15] is available on github for reference.


  [1]: /2021/03/unit-testing-in-maven-junit4-tests/
  [2]: https://maven.apache.org/surefire/maven-surefire-plugin/index.html
  [3]: https://junit.org/junit5/
  [4]: https://junit.org/junit5/docs/current/api/org.junit.platform.engine/org/junit/platform/engine/TestEngine.html
  [5]: https://github.com/the-code-journal/maven-for-beginners/tree/main/011-unit-tests-junit5/initial
  [6]: https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api
  [7]: https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine
  [8]: https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter
  [9]: https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations
 [10]: https://junit.org/junit4/javadoc/latest/org/junit/Before.html
 [11]: https://junit.org/junit4/javadoc/latest/org/junit/After.html
 [12]: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/BeforeEach.html
 [13]: https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/AfterEach.html
 [14]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit-platform.html
 [15]: https://github.com/the-code-journal/maven-for-beginners/tree/main/011-unit-tests-junit5/final
