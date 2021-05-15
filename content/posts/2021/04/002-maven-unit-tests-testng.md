---
date: 2021-04-22
linktitle: Unit Testing in Maven - TestNG tests
next: /2021/05/curious-case-of-modulo-operator/
prev: /2021/04/unit-testing-in-maven-migrating-from-junit-4-to-junit5-tests/
title: Unit Testing in Maven - TestNG Tests
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


In this article, we will look at [TestNG][1], which is another testing framework for Java.

Below is the video with the Demo to configure [TestNG][1] tests in your Maven project.

{{< yt "https://www.youtube.com/embed/sb0Q1bKSi0A" >}}





## TestNG

[TestNG][1] is not as popular as [Junit][2], but it still does have quite a strong community, which really loves it and continues to push it for a wider usage.

As far as the comparison goes, both [TestNG][1] and [Junit][2] are very similar to each other, they follow the [similar structure of tests][3], similar features. However, [TestNG][1] provides a more extensive toolkit for testing. Unless your requirements are very specific about how want to write, manage and run the tests, whatever you wish to do with [Junit][2], can be done with [TestNG][3] and vice-versa. There could be slightly different ways to achieve those in [TestNG][3], but the steps are well documented and writing tests should be fairly straight forward.

And then, [Maven-Surefire-Plugin][4], which is responsible for running the tests, works out of the box with [TestNG][1] as well. Let’s see how we can configure [TestNG][1] in a Maven project.




## The Project

Here is the [starting project][5], and it has `pom.xml` and `Runner.java`. Let's look at these.

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
└── src
    └── main
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── unittests
                            └── testng
                                └── Runner.java
</pre>
{{< /rawhtml >}}

First, `pom.xml`. Apart from project co-ordinates and compiler propererties, we have build configuration for [maven-surefire-plugin][6].

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.codejournal.maven</groupId>
  <artifactId>maven-unit-tests-testng</artifactId>
  <version>1.0</version>
  <name>maven-unit-tests-testng</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

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
package io.codejournal.maven.unittests.testng;

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





{{< articlead >}}

## Configuring TestNG

Just like any other dependency, to write [TestNG][1] tests, we need the [TestNG dependency][7]. Add the following dependency in `pom.xml`. You can always look for the latest version on [Maven Central for TestNG][7].

```xml
<dependencies>
  <dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.4.0</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

And we are all set to write TestNG tests.





## Writing the TestNG test

Create the `RunnerTest.java` in your `src/test/java` directory with package - `io.codejournal.maven.unittests.testng`

Here is the project structure after creating `RunnerTest.java`.

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
    │                       └── testng
    │                           └── Runner.java
    └── test
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── unittests
                            └── testng
                                └── RunnerTest.java
</pre>
{{< /rawhtml >}}



### Super Quick TestNG primer

[TestNG documentation][8] provides various examples for pretty much all of your uses cases for writing tests. Here is a super quick primer to TestNG using just annotations.

- `@Test` - Marks a class or method as part of a test
- `@BeforeMethod` - Runs the annotated method before every test of the class
- `@AfterMethod` - Runs the annotated method after every test of the class
- `@BeforeClass` - Runs the annotated method before the first test of the current class
- `@AfterClass` - Runs the annotated method after the last test of the current class

One good thing with `@Test` annotation is that you can use it at the class level. That way, you don't need to mark every test method with `@Test` annotation. For example, usually we write our test methods like below, where each test method is annotated with `@Test`.

```java
public class RunnerTest {

  @Test
  public void sumReturnsCorrectResult1() {
    // ...
  }

  @Test
  public void sumReturnsCorrectResult2() {
    // ...
  }
}
```

But, with [TestNG][1], you can add the `@Test` annotation at the class level and that makes all the methods of the class as a test method.

```java
@Test
public class RunnerTest {

  public void sumReturnsCorrectResult1() {
    // ...
  }

  public void sumReturnsCorrectResult2() {
    // ...
  }
}
```

Here is the full `RunnerTest.java`.

```java
package io.codejournal.maven.unittests.testng;

import static org.testng.Assert.assertEquals;

import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

@Test
public class RunnerTest {

    @BeforeMethod
    public void setUp() {
        System.out.println("\nRunning setUp...");
    }

    public void sumReturnsCorrectResult1() {
        System.out.println("Running sumReturnsCorrectResult1...");
        assertEquals(3, Runner.sum(1, 2));
    }

    public void sumReturnsCorrectResult2() {
        System.out.println("Running sumReturnsCorrectResult2...");
        assertEquals(8, Runner.sum(3, 5));
    }

    @AfterMethod
    public void tearDown() {
        System.out.println("Running tearDown...\n");
    }
}
```





{{< articlead >}}

## Running the TestNG tests with Maven

As already mentioned earlier, [maven-surefire-plugin][6] supports [TestNG][1] out of the box. So, we can run the test via `mvn clean test`.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------< io.codejournal.maven:maven-unit-tests-testng >------------
[INFO] Building maven-unit-tests-testng 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-unit-tests-testng ---
[INFO] Deleting /home/codejournal/workspace/maven-unit-tests-testng/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-unit-tests-testng ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-testng/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-unit-tests-testng ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-testng/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-unit-tests-testng ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/maven-unit-tests-testng/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-unit-tests-testng ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/codejournal/workspace/maven-unit-tests-testng/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ maven-unit-tests-testng ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.unittests.testng.RunnerTest

Running setUp...
Running sumReturnsCorrectResult1...
Running tearDown...

Running setUp...
Running sumReturnsCorrectResult2...
Running tearDown...

[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.585 s - in io.codejournal.maven.unittests.testng.RunnerTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.457 s
[INFO] Finished at: 2021-04-22T02:58:16+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## TestNG Reports

Another good thing with [TestNG][1] is automatic HTML report generated for your tests. You don't need to configure anything as such for this, and test report is generated along with your test execution.

The report is generated at `target/surefire-reports/` folder. You can start a web server this directory and open the reports on the browser.

```
-> python3 -m http.server --directory=target/surefire-reports/
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Two sections in this report that I personally find very useful and insightful.
- `Times` - Provides the execution time of each test
- `Chronological View` - Provides the execution order of the methods that ran in the tests

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/04/maven-testng-report.png" alt="Maven TestNG Report" style="height:400px" />
</div>
{{< /rawhtml >}}



The [working code][9] is available on github for reference.


  [1]: https://testng.org/
  [2]: https://junit.org/junit5/
  [3]: https://en.wikipedia.org/wiki/XUnit
  [4]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/testng.html
  [5]: https://github.com/the-code-journal/maven-for-beginners/tree/main/013-unit-tests-testng/initial
  [6]: https://maven.apache.org/surefire/maven-surefire-plugin
  [7]: https://mvnrepository.com/artifact/org.testng/testng
  [8]: https://testng.org/doc/documentation-main.html
  [9]: https://github.com/the-code-journal/maven-for-beginners/tree/main/013-unit-tests-testng/final
