---
date: 2022-04-09
linktitle: JaCoCo Maven plugin - Report-Aggregate Goal Configurations
prev: /2022/04/jacoco-maven-plugin-dump-goal-configurations/
next: /2020/08/install-apache-maven-on-linux/
title: JaCoCo Maven plugin - Report-Aggregate Goal Configurations
weight: 10
tags: [ "maven", "devops", "tools", "coverage"]
categories: [ "DevOps", "Maven", "Tools", "Code Coverage" ]
---


[Code coverage][1] is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

In this article, we will be taking a look at how to generate a combined code coverage report for a multi-module Maven project using [JaCoCo Maven plugin][2] with goal - [**report-aggregate**][3].

Here is video for the same.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo

[JaCoCo][4] is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code.

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.




## JaCoCo Goal - report-aggregate

[**report-aggregate**][3] goal is generally used with multi-module Maven projects. The report is created from all modules that a project depends on.

To generate a report for a multi-module Maven project, here is how it needs to be setup.

{{< rawhtml >}}
<pre class="code-output">
parent/
├── child-module1
├── child-module2
├── reports       &lt;!----- Adds dependency for child-module1 and child-module2 modules
└── pom.xml
</pre>
{{< /rawhtml >}}

And when you run the build to generate the aggregate report using the [**report-aggregate**][3] goal, the reports are generated in the `reports` module for the entire project.

Let's generate a report using [**report-aggregate**][3] for a multi-module Maven project and see things in action.





## Project

Here is a multi-module Maven project structure that we will use. We have three child modules - `maven-jacoco-multi-module-demo-numbers`,  `maven-jacoco-multi-module-demo-strings`, and `maven-jacoco-multi-module-demo-reports`.

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
|
├── maven-jacoco-multi-module-demo-numbers
│   ├── pom.xml
│   └── src
│       ├── main/java
│       │       └── io/codejournal/maven/jacoco/numbers
│       │                                       └── LastDigit.java
│       └── test/java
│               └── io/codejournal/maven/jacoco/numbers
│                                               └── LastDigitTest.java
|
├── maven-jacoco-multi-module-demo-strings
│   ├── pom.xml
│   └── src
│       ├── main/java
│       │       └── io/codejournal/maven/jacoco/strings
│       │                                       └── HelloName.java
│       └── test/java
│               └── io/codejournal/maven/jacoco/strings
│                                               └── HelloNameTest.java
|
└── maven-jacoco-multi-module-demo-reports
    └── pom.xml
</pre>
{{< /rawhtml >}}

### Parent - pom.xml

In the parent `pom.xml`, we configure the project co-ordinates, compiler properties, child modules for `numbers, strings and report`, dependencies for [JUnit Jupiter][5] and [AssertJ][6], and plugins that include [Maven Surefire plugin][7] and [JaCoCo Maven plugin][2].



For [JaCoCo Maven plugin][2], we also added the executions for [prepare-agent][8] and [report][9] goals, so that when the build runs, all the child modules can generate their reports individually.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.codejournal.maven.jacoco</groupId>
    <artifactId>maven-jacoco-multi-module-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>maven-jacoco-multi-module-demo-numbers</module>
        <module>maven-jacoco-multi-module-demo-strings</module>
        <module>maven-jacoco-multi-module-demo-reports</module>
    </modules>

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
        </plugins>
    </build>
</project>
```




## maven-multi-module-demo-numbers

The `pom.xml` for numbers module is very minimal. It will use the configurations as available from the parent pom.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>io.codejournal.maven.jacoco</groupId>
        <artifactId>maven-jacoco-multi-module-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>maven-jacoco-multi-module-demo-numbers</artifactId>

</project>
```

Let's take a look at the `LastDigit.java` and `LastDigitTest.java`.

```java
package io.codejournal.maven.jacoco.numbers;

public class LastDigit {

    public final boolean lastDigit(final int a, final int b) {

        final int lastDigitA = a % 10;
        final int lastDigitB = b % 10;

        return (lastDigitA == lastDigitB);
    }
}
```

```java
package io.codejournal.maven.jacoco.numbers;

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
    public void testcase1() {
        // Test code here
    }
}

```





## maven-multi-module-demo-strings

This module is very similar to that of `numbers` module, with change to package and files. The `pom.xml` is also almost same.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>io.codejournal.maven.jacoco</groupId>
        <artifactId>maven-jacoco-multi-module-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>maven-jacoco-multi-module-demo-strings</artifactId>

</project>
```

```java
package io.codejournal.maven.jacoco.strings;

public class HelloName {

    public final String helloName(final String name) {

        String greeting = "Hello";

        if (name != null && !name.isEmpty()) {
            greeting += " " + name;
        }

        return greeting + "!";
    }
}
```

```java
package io.codejournal.maven.jacoco.strings;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class HelloNameTest {

    private HelloName fixture;

    @BeforeEach
    public void setUp() {
        fixture = new HelloName();
    }

    @Test
    public void testcase() {
        // Test code here
    }
}
```





## maven-multi-module-demo-reports

This is the module that will generate the aggregated reports. It needs to add `maven-jacoco-multi-module-demo-numbers` and `maven-jacoco-multi-module-demo-strings` as dependencies so that `reports` module is run at the very end. This way, coverage data for all the dependent modules is generated and available which `report-aggregate` goal can use and generate a combined report.

Here is the `pom.xml`.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>io.codejournal.maven.jacoco</groupId>
        <artifactId>maven-jacoco-multi-module-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>maven-jacoco-multi-module-demo-reports</artifactId>

    <dependencies>
        <dependency>
            <groupId>io.codejournal.maven.jacoco</groupId>
            <artifactId>maven-jacoco-multi-module-demo-numbers</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>io.codejournal.maven.jacoco</groupId>
            <artifactId>maven-jacoco-multi-module-demo-strings</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.8</version>
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
</project>
```

Let's run the build now with `mvn clean verify` and you will get coverage data generated. Here is the output snippet from the build logs.

{{< rawhtml >}}
<pre class="code-output">
...
...
[INFO] --- jacoco-maven-plugin:0.8.8:report-aggregate (jacoco-report-aggregate) @ maven-jacoco-multi-module-demo-reports ---
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-multi-module-demo/maven-jacoco-multi-module-demo-numbers/target/jacoco.exec
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-multi-module-demo/maven-jacoco-multi-module-demo-strings/target/jacoco.exec
[INFO] Analyzed bundle 'maven-jacoco-multi-module-demo-numbers' with 1 classes
[INFO] Analyzed bundle 'maven-jacoco-multi-module-demo-strings' with 1 classes
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for maven-jacoco-multi-module-demo 0.0.1-SNAPSHOT:
[INFO] 
[INFO] maven-jacoco-multi-module-demo ..................... SUCCESS [  0.186 s]
[INFO] maven-jacoco-multi-module-demo-numbers ............. SUCCESS [  2.091 s]
[INFO] maven-jacoco-multi-module-demo-strings ............. SUCCESS [  1.051 s]
[INFO] maven-jacoco-multi-module-demo-reports ............. SUCCESS [  0.030 s]
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

The final combined report is available in the `reports` module at `target/site/jacoco-aggregate`. Notice the folder `jacoco-aggregate` in the folder path.

{{< rawhtml >}}
<pre class="code-output">
-> tree maven-jacoco-multi-module-demo-reports/target
maven-jacoco-multi-module-demo-reports/target
├── maven-archiver
│   └── pom.properties
├── maven-jacoco-multi-module-demo-reports-0.0.1-SNAPSHOT.jar
└── site
    └── jacoco-aggregate     &lt;-------- Folder is jacoco-aggregate
        ├── index.html
        ├── jacoco.csv
        ├── jacoco-resources
        ├── jacoco-sessions.html
        ├── jacoco.xml
        |
        ├── maven-jacoco-multi-module-demo-numbers     &lt;-------- Report for numbers module
        │   └── index.html
        |
        └── maven-jacoco-multi-module-demo-strings     &lt;-------- Report for strings module
            └── index.html
</pre>
{{< /rawhtml >}}

Here is how the report looks like.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2022/04/maven-jacoco-multi-module.png" alt="JaCoCo Maven plugin - Multi Module Report" />
</div>
{{< /rawhtml >}}

The project uses the minimalistic configuration to generate a combined report using `report-aggregate` goal. Take a look at the [goal documentation here][3] to configure additional properties.



  [1]: https://en.wikipedia.org/wiki/Code_coverage
  [2]: https://www.jacoco.org/jacoco/trunk/doc/maven.html
  [3]: https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html
  [4]: https://www.jacoco.org
  [5]: https://junit.org/junit5/
  [6]: https://assertj.github.io/doc/
  [7]: https://maven.apache.org/surefire/maven-surefire-plugin/
  [8]: /2022/04/code-coverage-with-jacoco-maven-plugin/#jacoco-goal---prepare-agent
  [9]: /2022/04/jacoco-maven-plugin-report-goal-configurations/
