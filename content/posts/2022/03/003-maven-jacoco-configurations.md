---
date: 2022-03-27
linktitle: Code Coverage with Maven Jacoco Plugin
next: /2020/08/install-apache-maven-on-linux/
prev: /2022/03/integration-testing-in-maven-failsafe-configurations/
title: Code Coverage with Maven Jacoco Plugin
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


Code coverage is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

Now, there are multiple tools available to help you measure the code coverage for your Java code like Open Clover, Cobertura, JCov. However, in this article, we will be taking a look at the most popular tool in this list - JaCoCo.

Here is video for the same.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo

JaCoCo is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.

The most popular usage of JaCoCo is performing the instrumentation using the Maven JaCoCo plugin. Let's see how we can configure this for a simple Java project.





## JaCoCo Maven Plugin

Here is a project that we will use to configure the Maven JaCoCo plugin and generate coverage report.

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

Here is the code for `LastDigit.java`. This is the solution to CodingBat problem [(Warmup-1 -> lastDigit)](https://codingbat.com/prob/p125339). 

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

And finally, the `pom.xml`. It has project co-ordinates, maven compiler properties, dependencies for JUnit and AssertJ, and then we have Maven Surefire Plugin, which will run our tests.

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

Let's configure Maven JaCoCo plugin now. Simply add the plugin co-ordinates.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
</plugin>
```

Now, we need to define how the instrumentation happens for our project. To do that, we need to prepare the Java Agent to instrument the code when the application runs. This is done using the `prepare-agent` goal(`prepare-agent-integration` in case of integration tests). When the tests/application is run, the coverage data is recorded and written to a file, which then later can be used to either check rules(`check` goal) or simply generate a report(`report` goal) for manual analysis and reporting.

For now, let's just generate the report, so we will need to run the `prepare-agent` goal and then `report` goal. Here is how we will configure the plugin eventually.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
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

Let's run the build with `mvn clean verify` to generate our report. Now, the output for the maven build isn't complicated. There are two things that will be worth noting in the build output.

First one is the setting of the `argLine` property. It sets the `-javaagent` path so that instrumentation can be done for the code. Here is the section from the build output.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.7:prepare-agent (jacoco-prepare-agent) @ maven-jacoco-plugin-demo ---
[INFO] argLine set to -javaagent:/home/codejournal/.m2/repository/org/jacoco/org.jacoco.agent/0.8.7/org.jacoco.agent-0.8.7-runtime.jar=destfile=/home/
codejournal/workspace/maven-jacoco-plugin-demo/target/jacoco.exec
...
</pre>
{{< /rawhtml >}}

And the second part is the generation of the code coverage report. This is done at the end after the tests are executed.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.7:report (jacoco-report) @ maven-jacoco-plugin-demo ---
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-plugin-demo/target/jacoco.exec
[INFO] Analyzed bundle 'maven-jacoco-plugin-demo' with 1 classes
...
</pre>
{{< /rawhtml >}}

Now, if you go to your `target/site/jacoco` directory, you will find the reports.

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
    <img src="/images/2022/03/maven-jacoco-report.png" alt="Maven JaCoco Plugin Report" />
</div>
{{< /rawhtml >}}

The above is a very simplistic use case. JaCoCo provides more goals that you can use for more complicated use cases and these are discussed below.





## JaCoCo Maven Plugin - Goals

- `prepare-agent` - Prepares a property pointing to the JaCoCo runtime agent that can be passed as a VM argument to the application under test.
- `prepare-agent-integration` - Same as `prepare-agent`, but provides default values suitable for integration-tests
- `check` - Checks that the code coverage metrics are being met.
- `report` - Creates a code coverage report for tests of a single project
- `report-integration` - Same as `report`, but provides default values suitable for integration-tests
- `merge` - Mojo for merging a set of execution data files (*.exec) into a single file
- `report-aggregate` - Creates a structured code coverage report from multiple projects
- `dump` - Request a dump over TCP/IP from a JaCoCo agent running in `tcpserver` mode.
- `instrument` - Performs offline instrumentation
- `restore-instrumented-classes` - Restores original classes as they were before offline instrumentation

Let's talk about these goals in details.

## JaCoCo Goal - prepare-agent

This goal prepares a property pointing to the JaCoCo runtime agent that can be passed as a VM argument to the application under test. The default property name is `argLine`. This goal binds to `initialize` phase of the default lifecycle.

If your project doesn't pass any arguments to your tests/application, then you don't need to configure anything. However, if your application does pass some arguments, you will need to configure this.

For instance, if you define `argLine` property of Maven Surefire Plugin, then you need to configure as below.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration>
    <argLine>@{argLine} -your -extra -arguments</argLine>
  </configuration>
</plugin>
```

In most cases, you won't needing any additional configuration, however, there are additional configurations available that you can use if your need them. Check the [documentation here]() to find all of them.





## JaCoCo Goal - prepare-agent-integration

Same as `prepare-agent`, but provides default values suitable for integration-tests. It also generated the data to a different output file(`${project.build.directory}/jacoco-it.exec`) and it binds to `pre-integration-test` phase of default lifecycle. The [configuration options]() too are very similar to that of `prepare-agent` goal.





## JaCoCo Goal - report

`report` goal creates a code coverage report for tests of a single project in multiple formats (HTML, XML, and CSV). It binds to `verify` phase of the default lifecycle. Since, this goal is most widely used, let's discuss its configuration parameters.

- `<skip>` - Flag used to enable/disable report generation
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
- `<dataFile>` - Data file with execution data for which report is generated. Default file that JaCoCo looks is `${project.build.directory}/jacoco.exec`
- `<title>` - Name of the root node in HTML report pages. The default value is `${project.name}`
- `<footer>` - Footer text used in the HTML report pages.
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





## JaCoCo Goal - report-integration

Same as `report`, but provides default values suitable for integration-tests. It looks for different output file in the project(`${project.build.directory}/jacoco-it.exec`) and it binds to `verify` phase of default lifecycle. The [configuration options]() too are very similar to that of `report` goal.





## JaCoCo Goal - merge

If you generate multiple JaCoCo coverage files with your build, and want to combine them together, you can use the `merge` goal. You can configure the file path locations using `<fileSet>` and JaCoCo will merge all the instrumentation output files that it locates. Here is the [documentation for this goal](https://www.jacoco.org/jacoco/trunk/doc/merge-mojo.html).





## JaCoCo Goal - report-aggregate

This goal is generally used with multi-module Maven projects. The report is created from all modules this project depends on. From those projects class and source files as well as JaCoCo execution data files will be collected. In addition execution data is collected from the project itself. This also allows to create coverage reports when tests are in separate projects than the code under test, for example in case of integration tests.

To generate a report for a multi-module Maven project, here is how it needs to be setup.

{{< rawhtml >}}
<pre class="code-output">
maven-jacoco-multi-module-demo/
├── maven-jacoco-multi-module-demo-numbers
├── maven-jacoco-multi-module-demo-strings
├── maven-jacoco-multi-module-demo-reports
└── pom.xml
</pre>
{{< /rawhtml >}}

The parent `pom.xml` just sets up the `prepare-agent` so that every module can instrument their own module code.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
    <executions>
        <execution>
            <id>jacoco-prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

To generate the report, we created an additional module `maven-jacoco-multi-module-demo-reports` and all our rest of the modules will be included as dependency here. This is done to ensure that all other modules will have their code instrumented first and the data files will be available to generate the report. Here is the `pom.xml` for module - `maven-jacoco-multi-module-demo-reports`.

```xml
<dependencies>
    <dependency>
        <groupId>io.codejournal.jacocodemo</groupId>
        <artifactId>maven-jacoco-multi-module-demo-numbers</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>io.codejournal.jacocodemo</groupId>
        <artifactId>maven-jacoco-multi-module-demo-strings</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.7</version>
            <executions>
                <execution>
                    <id>jacoco-report-aggregate</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>report-aggregate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

And then when you run the whole build, you can see the reports getting generated.

{{< rawhtml >}}
<pre class="code-output">
[INFO] --- jacoco-maven-plugin:0.8.7:report-aggregate (jacoco-report-aggregate) @ maven-jacoco-multi-module-demo-reports ---
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-multi-module-demo/maven-jacoco-multi-module-demo-numbers/target/jacoco.exec
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-multi-module-demo/maven-jacoco-multi-module-demo-strings/target/jacoco.exec
[INFO] Analyzed bundle 'maven-jacoco-multi-module-demo-numbers' with 1 classes
[INFO] Analyzed bundle 'maven-jacoco-multi-module-demo-strings' with 1 classes
</pre>
{{< /rawhtml >}}

Additional configuration option for this goal can be [found here](https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html).





## JaCoCo Goal - check

This goal is generally used with multi-module Maven projects. The report is created from all modules this project depends on. From those projects class and source files as well as JaCoCo execution data files will be collected. In addition execution data is collected from the project itself. This also allows to create coverage reports when tests are in separate projects than the code under test, for example in case of integration tests.





## JaCoCo Goal - dump


## JaCoCo Goal - instrument

## JaCoCo Goal - restore-instrumented-classes











{{< articlead >}}




  [1]: https://en.wikipedia.org/wiki/Unit_testing
  [2]: https://en.wikipedia.org/wiki/Integration_testing
  [3]: https://maven.apache.org/surefire/maven-failsafe-plugin/
  [4]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/skipping-tests.html
  [5]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/inclusion-exclusion.html
  [6]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/single-test.html
  [7]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html#Parallel_Test_Execution
  [8]: https://www.liquidweb.com/kb/what-are-regular-expressions/
  [9]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html#Forked_Test_Execution
