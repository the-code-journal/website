---
date: 2021-03-12
linktitle: Unit Testing in Maven - Junit4 Tests
next: /2021/03/unit-testing-in-maven-junit5-tests/
prev: /2021/03/unit-testing-in-maven-pojo-tests/
title: Unit Testing in Maven - Junit4 Tests
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


In our [previous article][1], we looked at how Maven uses Surefire plugin to run Pojo Tests, not very popular but still supported.

In this article, we will look at running [Junit][2] tests. [Junit][2] is the most popular library in Java ecosystem, and is used by organizations and open-source projects all over the world. It is a very mature, is actively worked upon and hugely supported by the community.

The latest version of Junit, which is [Junit 5][3] was [released in 2017][4], but till today, there are a huge number of projects that still use the [Junit 4][5], and have yet not migrated to the newer [Junit 5][3]. [JUnit 5][3] was a major change from [Junit 4][5], and that is why there is a difference in the way tests and dependencies are configured.

Below is the video with the Demo to running [Junit 4][5] tests using [Maven Surefire Plugin][6].

{{< yt "https://www.youtube.com/embed/ROm9If78nTo" >}}





### Maven Surefire Plugin and Junit 4 Tests

Maven Surefire Plugin has a [dedicated article][7] on configuring Junit 4 tests. Essentially, to use Junit, you will need to configure Junit dependency in `pom.xml`.

As of writing of this article, the latest version of Junit 4 is [4.13.2][8]. Add the below dependency to your `pom.xml`.

```xml
  <dependencies>
  [...]
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  [...]
  </dependencies>
```

Apart from this, add the `maven-surefire-plugin` in the `build` section as well.

```xml
  <plugins>
  [...]
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.0.0-M5</version>
    </plugin>
  [...]
  </plugins>
```





{{< articlead >}}

## Fixture Method

Let's write the simplest [fixture method][9] of adding two numbers passed to the method, and returning their sum. Here is the method with its implementation.

```java
public class Runner {

    public static int sum(final int number1, final int number2) {
        return number1 + number2;
    }
}
```





## Junit 4 Test

A quick into to Junit 4.

- Add `@Test` annotation to all the test methods. Method names `DO NOT` require to have `test*`.
- Use `@Before` for `setUp` method to execute before every test
- Use `@After` for `tearDown` method to execute after every test
- Use [`Assert.*`][10] to compare the expected and actual values

Here is a Junit 4 test for the fixture method above.

```java
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

And that's it. Surefire plugin will be able to pick this up and run the test.


{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------< io.codejournal.maven:maven-unit-tests-junit4 >-------------
[INFO] Building maven-unit-tests-junit4 1.0
[INFO] --------------------------------[ jar ]---------------------------------
...
...
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ maven-unit-tests-junit4 ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.maven.unittests.junit4.RunnerTest

Running setUp...
Running sumReturnsCorrectResult...
Running tearDown...

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.035 s - in io.codejournal.maven.unittests.junit4.RunnerTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.239 s
[INFO] Finished at: 2021-03-12T23:05:50+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

The [working code][11] is available on github for reference.


  [1]: /2021/03/unit-testing-in-maven-pojo-tests/
  [2]: https://junit.org/
  [3]: https://junit.org/junit5/
  [4]: https://junit.org/junit5/docs/5.0.0/user-guide/#release-notes-5.0.0
  [5]: https://junit.org/junit4/
  [6]: https://maven.apache.org/surefire/maven-surefire-plugin/index.html
  [7]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html
  [8]: https://mvnrepository.com/artifact/junit/junit
  [9]: https://en.wikipedia.org/wiki/Test_fixture
 [10]: https://junit.org/junit4/javadoc/latest/org/junit/Assert.html
 [11]: https://github.com/the-code-journal/maven-for-beginners/tree/main/010-unit-tests-junit4/final
