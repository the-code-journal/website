---
date: 2022-04-01
linktitle: Code Coverage with JaCoCo Maven Plugin
prev: /2022/03/integration-testing-in-maven-failsafe-configurations/
next: /2022/04/jacoco-maven-plugin-check-goal-configurations/
title: Code Coverage with JaCoCo Maven Plugin
weight: 10
tags: [ "maven", "devops", "tools", "coverage"]
categories: [ "DevOps", "Maven", "Tools", "Code Coverage" ]
---


[Code coverage][1] is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

Now, there are multiple tools available to help you measure the code coverage for your Java code like [Open Clover][2], [Cobertura][3], [JCov][4]. However, in this article, we will be taking a look at the most popular tool in this list - [JaCoCo][5].

Here is video for the same.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo

[JaCoCo][5] is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code.

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.

The most popular usage of JaCoCo is performing the instrumentation using the [JaCoCo Maven Plugin][6]. Let's see how we can configure this for a simple Java project.





## Project

Here is a project that we will use to configure the [JaCoCo Maven Plugin][6] and generate coverage report.

```
.
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── io/codejournal/maven/jacocodemo/numbers
    │                                           └── LastDigit.java
    └── test
        └── java
            └── io/codejournal/maven/jacocodemo/numbers
                                                └── LastDigitTest.java
```

Here is the code for `LastDigit.java`. This is the solution to [CodingBat][7] problem [(Warmup-1 -> lastDigit)][8]. 

```java
package io.codejournal.maven.jacocodemo.numbers;

public class LastDigit {

    public final boolean lastDigit(final int a, final int b) {

        final int lastDigitA = a % 10;
        final int lastDigitB = b % 10;

        return (lastDigitA == lastDigitB);
    }
}
```

And here is the code for `LastDigitTest.java`.

```java
package io.codejournal.maven.jacocodemo.numbers;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class LastDigitTest {

    private LastDigit fixture;

    @BeforeEach
    public void setUp() {
        fixture = new LastDigit();
    }

    @Test
    public void shouldReturnTrueWhenLastDigitsAreSame() {

        final int num1 = 17;
        final int num2 = 7;

        final boolean expected = true;

        final boolean actual = fixture.lastDigit(num1, num2);

        assertThat(actual).isEqualTo(expected);
    }

    @Test
    public void shouldReturnFalseWhenLastDigitsAreNotSame() {

        final int num1 = 17;
        final int num2 = 13;

        final boolean expected = false;

        final boolean actual = fixture.lastDigit(num1, num2);

        assertThat(actual).isEqualTo(expected);
    }
}
```

And finally, the `pom.xml`. It has project co-ordinates, maven compiler properties, dependencies for [JUnit Jupiter][9] and [AssertJ][10], and then we have [Maven Surefire Plugin][11], which will run our tests.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>io.codejournal.maven</groupId>
    <artifactId>maven-jacoco-plugin-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.19.0</version>
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





{{< articlead >}}

## Configure JaCoCo plugin

Let's configure [JaCoCo Maven Plugin][6] now. Simply add the plugin co-ordinates.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
</plugin>
```

Now, we need to define how the instrumentation happens for our project. To do that, we need to pass the JaCoCo agent as [`-javaagent`][23] argument to instrument the code when the application runs. This is done using the `prepare-agent` goal(`prepare-agent-integration` in case of integration tests). When the tests/application is run, the coverage data is recorded by the agent and written to a file, which then later can be used to either check rules([`check`][21] goal) or simply generate a report([`report`][16] goal) for manual analysis and reporting.

For now, let's just generate the [`report`][16], so we will need to run the `prepare-agent` goal and then [`report`][16] goal. Here is how we will configure the plugin eventually.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <id>jacoco-prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-report</id>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Let's run the build with `mvn clean verify` to generate our report.

The output for the maven build isn't complicated. There are two things that are worth noting in the build output.

First one is the setting of the `argLine` property. It sets the [`-javaagent`][23] path so that instrumentation can be done for the code. Here is the section from the build output.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.8:prepare-agent (jacoco-prepare-agent) @ maven-jacoco-plugin-demo ---
[INFO] argLine set to -javaagent:/home/codejournal/.m2/repository/org/jacoco/org.jacoco.agent/0.8.8/org.jacoco.agent-0.8.8-runtime.jar=destfile=/home/
codejournal/workspace/maven-jacoco-plugin-demo/target/jacoco.exec
...
</pre>
{{< /rawhtml >}}

And the second part is the generation of the code coverage report. This is done at the end after the tests are executed, and [`report`][16] goal executes to generate the report.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.8:report (jacoco-report) @ maven-jacoco-plugin-demo ---
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-plugin-demo/target/jacoco.exec
[INFO] Analyzed bundle 'maven-jacoco-plugin-demo' with 1 classes
...
</pre>
{{< /rawhtml >}}

If you go to the `target/site/jacoco` directory, you will find the reports.

{{< rawhtml >}}
<pre class="code-output">
-> tree target/site/jacoco/
target/site/jacoco/
├── index.html                &lt;!-- Project Report in HTML -->
├── jacoco.csv                &lt;!-- Project Report in CSV -->
├── jacoco.xml                &lt;!-- Project Report in XML -->
├── jacoco-sessions.html
├── jacoco-resources
|
└── io.codejournal.maven.jacocodemo.numbers   &lt;!-- Package specific Report -->
    ├── index.html
    ├── index.source.html
    ├── LastDigit.html
    └── LastDigit.java.html
</pre>
{{< /rawhtml >}}

Here is the screenshot for the report.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2022/04/maven-jacoco-report.png" alt="JaCoCo Maven Plugin Report" />
</div>
{{< /rawhtml >}}

The above is a very simplistic and trivial use case. JaCoCo provides more goals that you can use for more complicated use cases and these are discussed below.





{{< articlead >}}

## JaCoCo Maven Plugin - Goals

- `prepare-agent` - Prepares a property pointing to the JaCoCo runtime agent that can be passed as a VM argument to the application under test.
- `prepare-agent-integration` - Same as `prepare-agent`, but provides default values suitable for integration-tests
- `check` - Checks that the code coverage metrics are being met.
- `report` - Creates a code coverage report for tests of a single project
- `report-integration` - Same as `report`, but provides default values suitable for integration-tests
- `merge` - Merging a set of execution data files (*.exec) into a single file
- `report-aggregate` - Creates a structured code coverage report from multiple projects
- `dump` - Request a dump over TCP/IP from a JaCoCo agent running in `tcpserver` mode.
- `instrument` - Performs offline instrumentation
- `restore-instrumented-classes` - Restores original classes as they were before offline instrumentation

Let's talk about these goals in details.





## JaCoCo Goal - prepare-agent

This goal prepares a property pointing to the JaCoCo runtime agent that can be passed as a VM argument to the application under test. The default property name is `argLine`. This goal binds to `initialize` phase of the [default lifecycle][12].

If your project doesn't pass any arguments to your tests/application, then you don't need to configure anything. However, if your application does pass some arguments, you will need to configure this.

For instance, if you define `argLine` property of [Maven Surefire Plugin][11], then you need to configure as below.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration>
    <argLine>@{argLine} -your -extra -arguments</argLine>
  </configuration>
</plugin>
```

In most cases, you won't be needing any additional configuration, however, there are [configuration properties][13] available that you can use if your need them. Check the [documentation here][13] to find all of them.





## JaCoCo Goal - prepare-agent-integration

Same as `prepare-agent`, but provides default values suitable for integration-tests. It also generates the data to a different output file(`${project.build.directory}/jacoco-it.exec`) and it binds to `pre-integration-test` phase of [default lifecycle][12]. The [configuration options][14] too are very similar to that of `prepare-agent` goal.





{{< articlead >}}

## JaCoCo Goal - report

[`report`][15] goal creates a code coverage report for tests of a single project in multiple formats (HTML, XML, and CSV). It binds to `verify` phase of the [default lifecycle][12].

I have a [dedicated article on report goal][16] that you can check as well. Below are the list of configuration parameters for this goal.

- `<formats>` - Report format to generate. Supported reports are - HTML, CSV and XML. By default, all reports are generated. Here is an example which only generates XML report
```xml
<execution>
    <id>jacoco-report</id>
    <goals>
        <goal>report</goal>
    </goals>
    <configuration>
        <formats>
            <format>XML</format>
        </formats>
    </configuration>
</execution>
```
- `<includes>/<excludes>` - A list of class files to include/exclude in the report. You may use wildcard characters (* and ?) as part of the configuration. Below is an example in which, all classes which has `jacocodemo` in it is package are included except if they have `jacocodemo/strings` in them.
```xml
<configuration>
    <includes>
        <include>**/jacocodemo/**/*</include>
    </includes>
    <excludes>
        <exclude>**/jacocodemo/strings/*</exclude>
    </excludes>
</configuration>
```
- `<dataFile>` - Data file with execution data for which report is generated. Default file that JaCoCo looks is `${project.build.directory}/jacoco.exec`
- `<title>` - Name of the root node in HTML report pages. The default value is `${project.name}`
- `<footer>` - Footer text used in the HTML report pages.
- `<skip>` - Flag used to enable/disable report generation

Here is the video that demonstrates usage of the [`report`][16] goal.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo Goal - report-integration

Same as [`report`][16], but provides default values suitable for integration-tests. It looks for different output file in the project(`${project.build.directory}/jacoco-it.exec`) and it binds to `verify` phase of [default lifecycle][12]. The [configuration options][17] too are very similar to that of `report` goal.





{{< articlead >}}

## JaCoCo Goal - merge

If you generate multiple JaCoCo coverage files with your build, and want to combine them together, you can use the `merge` goal. You can configure the file path locations using `<fileSet>` and JaCoCo will merge all the instrumentation output files that it locates. Here is the [documentation for this goal][18].

Here is the video that demonstrates the `merge` goal.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo Goal - report-aggregate

[`report-aggregate`][20] goal is generally used with multi-module Maven projects. The report is created from all modules this project depends on. Additional configuration option for this goal can be [found here][19].

I have a [dedicated article on report-aggregate goal][20] that you can check as well. Here is the video that demonstrates the `report-aggregate` goal in detail.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo Goal - check

[`check`][21] validates that the code coverage metrics are being met. Otherwise, the build will start to fail. And this can be a good guardrail to ensure that code coverage metrics are being met and they never go below certain defined thresholds. It is bound to `verify` phase of the [default lifecycle][12].

I have a [dedicated article on check goal][21] that you can check as well.

Here is an example, which configures a basic rule that your project needs to have minimum 80% code coverage.

```xml
<!--[...]-->
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.8</version>
      <executions>
        <execution>
          <id>jacoco-prepare-agent</id>
          <goals>
            <goal>prepare-agent</goal>
          </goals>
        </execution>
        <execution>
          <id>jacoco-check</id>
          <goals>
            <goal>check</goal>
          </goals>
          <configuration>
            <rules>
              <rule>
                <element>BUNDLE</element>
                <limits>
                  <limit>
                    <counter>INSTRUCTION</counter>
                    <value>COVEREDRATIO</value>
                    <minimum>0.80</minimum>
                  </limit>
                </limits>
              </rule>
            </rules>
          </configuration>
        </execution>
      </executions>
    </plugin>
<!--[...]-->
```

Here is the video that demonstrates the [`check`][21] goal in detail.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





{{< articlead >}}

## JaCoCo Goal - dump

[`dump`][22] goal requests a code coverage dump over TCP/IP from a JaCoCo agent running in `tcpserver` mode. This is usually done when the application is run in a separate environment and the tests are run against it. And once all the tests are executed against the application, you can get the code coverage data generated and transferred over the TCP/IP.

[`dump`][22] goal will connect to the JaCoCo agent running on the remote/local host, transfer the coverage data over TCP/IP and write the dump(or coverage) data to the configured output file.

I have an [article that demonstrates the dump goal][22] of a Spring Boot application and generation of report from the coverage dump. Here is the video for the same as well.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo Goal - instrument

The most common and widely used way to generate code coverage data is by using on the fly instrumentation using [Java Agent][23]. However, there can be situations where on-the-fly instrumentation is not suitable, for example:

- Runtime environments that do not support Java agents
- Deployments where it is not possible to configure JVM options
- Conflicts with other agents that do dynamic classfile transformation

In such scenarios, you can perform [offline instrumentation][24] using the [`instrument`][25] goal. Note that after execution of test you must restore original classes with help of [`restore-instrumented-classes`][26] goal.





## JaCoCo Goal - restore-instrumented-classes

When you have performed [offline instrumentation][24] using the [`instrument`][25], you must restore the original classes. This is done using the [`restore-instrumented-classes`][26] goal.



  [1]: https://en.wikipedia.org/wiki/Code_coverage
  [2]: https://openclover.org
  [3]: https://github.com/cobertura/cobertura
  [4]: https://wiki.openjdk.java.net/display/CodeTools/jcov
  [5]: https://www.jacoco.org
  [6]: https://www.jacoco.org/jacoco/trunk/doc/maven.html
  [7]: https://codingbat.com
  [8]: https://codingbat.com/prob/p125339
  [9]: https://junit.org/junit5/
 [10]: https://assertj.github.io/doc/
 [11]: https://maven.apache.org/surefire/maven-surefire-plugin/
 [12]: /2020/11/maven-lifecycles/
 [13]: https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-mojo.html
 [14]: https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-integration-mojo.html
 [15]: https://www.jacoco.org/jacoco/trunk/doc/report-mojo.html
 [16]: /2022/04/jacoco-maven-plugin-report-goal-configurations/
 [17]: https://www.jacoco.org/jacoco/trunk/doc/report-integration-mojo.html
 [18]: https://www.jacoco.org/jacoco/trunk/doc/merge-mojo.html
 [19]: https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html
 [20]: /2022/04/jacoco-maven-plugin-report-aggregate-goal-configurations/
 [21]: /2022/04/jacoco-maven-plugin-check-goal-configurations/
 [22]: /2022/04/jacoco-maven-plugin-dump-goal-configurations/
 [23]: https://docs.oracle.com/en/java/javase/17/docs/api/java.instrument/java/lang/instrument/package-summary.html
 [24]: https://www.jacoco.org/jacoco/trunk/doc/offline.html
 [25]: https://www.jacoco.org/jacoco/trunk/doc/instrument-mojo.html
 [26]: https://www.jacoco.org/jacoco/trunk/doc/restore-instrumented-classes-mojo.html
 