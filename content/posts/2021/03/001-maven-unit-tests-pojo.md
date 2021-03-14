---
date: 2021-03-11
linktitle: Unit Testing in Maven - Pojo Tests
next: /2021/03/unit-testing-in-maven-junit4-tests/
prev: /2021/02/generating-code-using-maven-swagger-to-java/
title: Unit Testing in Maven - Pojo Tests
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


In this article, we will look at how Maven is used to run the Unit tests. Unit tests are equally important as code itself, and they help you validate and verify the code, by testing it with various inputs, thereby having more confidence on the code written and lesser bugs when it runs in Production environment.

Pojo Tests are not very popular and they were used initially when [Junit][1] or [TestNG][2] were not very popular. But these days, you will see projects using JUnit primarily and TestNG sparsely. But for sake of completeness, Surefire still has support for these, in case you are using neither of these testing frameworks or if you have a project that is very very old, like 2005 or earlier, and they have Pojo tests in them.

Below is the video with the Demo to running Pojo tests using [Maven Surefire Plugin][3].

{{< yt "https://www.youtube.com/embed/gwc3SUumsM8" >}}






## Maven Surefire Plugin

When it comes to Unit tests, maven uses [Maven Surefire Plugin][3] to execute the tests. It is the default plugin that is by default linked to the [test lifecycle][4] phase. That means, even if you don’t configure this plugin explicitly in your `pom.xml`, maven will automatically use the appropriate version of this plugin and run the tests.

Surefire plugin supports all popular Unit Testing frameworks - [Junit][1] and [TestNG][2], and if don’t want to use these, it can run the old Pojo tests as well.



### Maven Surefire Plugin and Pojo Tests

To have Surefile plugin run the [Pojo tests][6], you need to follow below three things.

1. Test class names should end with `Test` - ex. `RunnerTest`
2. Test methods should start with `test` - ex. `testSomething()`
3. For set-up and tear-down methods which execute before and after of each test method, they need to be specified as below.

```java
    public void setUp();
    public void tearDown();
```





{{< articlead >}}

## Fixture Method

Let's write the simplest [fixture method][5] of adding two numbers passed to the method, and returning their sum. Here is the method with its implementation.

```java
public class Runner {

    public static int sum(final int number1, final int number2) {
        return number1 + number2;
    }
}
```





## Pojo Test

Taking into account for requirements for [Pojo Test][6], below is the Pojo test class. It uses [`assert`][8] keyword to compare the actual and expected values, and in case of mis-match, fails the test.

```java
public class RunnerTest {

    public void testSum() {

        System.out.println("Running testSum1...");

        final int number1 = 3;
        final int number2 = 5;

        final int expected = 8;

        final int actual = Runner.sum(number1, number2);

        assert actual == expected;
    }
}
```

And that's it. Surefire plugin will be able to pick this up and run the test.


{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------< io.codejournal.maven:maven-unit-tests-pojo >-------------
[INFO] Building maven-unit-tests-pojo 1.0
[INFO] --------------------------------[ jar ]---------------------------------
...
...
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven-unit-tests-pojo ---
[INFO] Surefire report directory: /home/codejournal/workspace/maven-unit-tests-pojo/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running io.codejournal.maven.unittests.pojo.RunnerTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.003 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.239 s
[INFO] Finished at: 2021-03-11T01:05:20+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

{{< rawhtml >}}
<div class="notification">
    As you can see, without any configuration in <code>pom.xml</code>, Surefire plugin was still invoked(an old version - 2.14.2) and it executed the Pojo test that we added.
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Surefire Plugin Configuration

You can always use the default Surefire plugin configured with Maven, but it will be an old version usually and it is advisable to configure the plugin with its version. It avoids surprise failures during Maven upgrades, and also, you can use latest features/bug fixes with newer versions.

To configure Surefire plugin, simple configure the plugin co-ordinates with its version.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.0.0-M5</version>
    </plugin>
  </plugins>
</build>
```

You can also configure the plugin in `<pluginManagement>` section, to have multiple build sections use same surefire plugin.

```xml
<build>
  <pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
      </plugin>
    </plugins>
  </pluginManagement>
</build>
```





{{< articlead >}}

## Test Failures

Whenever any test fails, Surefire plugin reports the failed test with details so that when you look at the logs, it is easily to know what expectation was not fulfilled and what value was expected.

Below is a sample run for a failed test.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------< io.codejournal.maven:maven-unit-tests-pojo >-------------
[INFO] Building maven-unit-tests-pojo 1.0
[INFO] --------------------------------[ jar ]---------------------------------
...
...
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven-unit-tests-pojo ---
[INFO] Surefire report directory: /home/codejournal/workspace/maven-unit-tests-pojo/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running io.codejournal.maven.unittests.pojo.RunnerTest
Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 0.002 sec <<< FAILURE!
io.codejournal.maven.unittests.pojo.RunnerTest.testSum()  Time elapsed: 0.001 sec  <<< FAILURE!
java.lang.AssertionError
	at io.codejournal.maven.unittests.pojo.RunnerTest.testSum(RunnerTest.java:20)


Results :

Failed tests:   io.codejournal.maven.unittests.pojo.RunnerTest.testSum()

Tests run: 1, Failures: 1, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.214 s
[INFO] Finished at: 2021-03-11T01:21:41+05:30
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test (default-test) on project maven-unit-tests-pojo: There are test failures.
[ERROR] 
[ERROR] Please refer to /home/codejournal/workspace/maven-unit-tests-pojo/target/surefire-reports for the individual test results.
[ERROR] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
</pre>
{{< /rawhtml >}}


The [working code][8] is available on github for reference.


  [1]: https://junit.org/
  [2]: https://testng.org/
  [3]: https://maven.apache.org/surefire/maven-surefire-plugin/
  [4]: /2020/11/maven-lifecycles/#lifecycle---default---testing-phases
  [5]: https://en.wikipedia.org/wiki/Test_fixture
  [6]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/pojo-test.html
  [7]: https://docs.oracle.com/javase/7/docs/technotes/guides/language/assert.html
  [8]: https://github.com/the-code-journal/maven-for-beginners/tree/main/009-unit-tests-pojo/final
