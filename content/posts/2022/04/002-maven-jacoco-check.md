---
date: 2022-04-02
linktitle: JaCoCo Maven Plugin - Check Goal Configurations
prev: /2022/04/code-coverage-with-jacoco-maven-plugin/
next: /2022/04/jacoco-maven-plugin-report-goal-configurations/
title: JaCoCo Maven Plugin - Check Goal Configurations
weight: 10
tags: [ "maven", "devops", "tools", "coverage"]
categories: [ "DevOps", "Maven", "Tools", "Code Coverage" ]
---


[Code coverage][1] is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

In this article, we will be taking a look at how to perform [**check**][3] against code coverage data data generated by the [JaCoCo Maven Plugin][2], so that if the code is not conforming to agreed metrics, the build will fail.

Here is video for the same.

{{< yt "https://www.youtube.com/embed/5mJ1hJGIguw" >}}





## JaCoCo

[JaCoCo][4] is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code.

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.





## JaCoCo Goal - check

[**check**][3] validates that the code coverage metrics are being met. Otherwise, the build will start to fail. And this can be a good guardrail to ensure that code coverage metrics are being met and they never go below/above certain defined thresholds.

[**check**][3] goal is bound to `verify` phase of the [default lifecycle][5].

Here is an example configuration, which configures a basic rule that your project needs to have minimum 80% code coverage.

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

### &lt;element> Types

`<element>` defines on which component the rule should be applied. The possible values are as below.

- BUNDLE - The rule is applicable for the entire project/module
- PACKAGE - The rule is applicable for each package in the project/module
- CLASS - The rule is applicable for each Class file in the project/module
- SOURCEFILE - The rule is applicable for each Java file(SourceFile) in the project/module.
- METHOD - The rule is applicable for each Method in the project/module.

The difference between `CLASS` and `SOURCEFILE` is that `CLASS` parameter looks at each compiled class files, while `SOURCEFILE` looks at your source Java files. Consider the below `Shape.java`.

```java
public class Shape {
}

class Triangle extends Shape {
}

class Square extends Shape {
}
```

Now, here there is only `1(one) SOURCEFILE` or Java file, which has `3(three) CLASS` in it.



### &lt;limit> Types

`<limit>` defines which kind of limit counter is configured in the rule. Possible values are:

- INSTRUCTION - Instructions in the compiled class file
- LINE - Source code lines
- BRANCH - Branches in the code execution flow
- COMPLEXITY - [Cyclomatic Complexity][6] of the methods
- METHOD - Represents methods
- CLASS - Represents class



### &lt;value> Types

`<value>` defines which metric should be applied in the rule. Possible values are:

- TOTALCOUNT - Represents the total count of occurrence for a value
- COVEREDCOUNT - Represents the count of covered code for a value
- MISSEDCOUNT - Represents the count of missed code for a value
- COVEREDRATIO - Represents the covered code as a ratio(0.0 - 1.0 or 0% - 100%)
- MISSEDRATIO - Represents the missed code as a ratio(0.0 - 1.0 or 0% - 100%)

If a value refers to a ratio it must be in the range from 0.0 to 1.0 where the number of decimal places will also determine the precision in error messages. A ratio may optionally be declared as a percentage where 0.80 and 80% represent the same value.





{{< articlead >}}

## JaCoCo Goal - check parameters

Below are the optional parameters that can configure as part of `check` goal.

- `<dataFile>` - Location of Code Coverage data file
- `<excludes>` - A list of classes to exclude from the check rule validation.
- `<includes>` - A list of classes to include into the check rule validation.
- `<skip>` - Skip execution of the goal





## JaCoCo Goal - check Rule Examples

### Code Coverage of the Project should be minimum 80% and no class should be missing its coverage

```xml
<rule>
  <element>BUNDLE</element>
  <limits>
    <limit>
      <counter>INSTRUCTION</counter>
      <value>COVEREDRATIO</value>
      <minimum>0.80</minimum>
    </limit>
    <limit>
      <counter>CLASS</counter>
      <value>MISSEDCOUNT</value>
      <maximum>0</maximum>
    </limit>
  </limits>
</rule>
```

### Methods can have at most 150 lines of code

```xml
<rule>
  <element>METHOD</element>
  <limits>
    <limit>
      <counter>LINE</counter>
      <value>TOTALCOUNT</value>
      <maximum>150</maximum>
    </limit>
  </limits>
</rule>
```

### Methods can have at most 10 as Complexity

```xml
<rule>
  <element>METHOD</element>
  <limits>
    <limit>
      <counter>COMPLEXITY</counter>
      <value>TOTALCOUNT</value>
      <maximum>10</maximum>
    </limit>
  </limits>
</rule>
```



  [1]: https://en.wikipedia.org/wiki/Code_coverage
  [2]: https://www.jacoco.org/jacoco/trunk/doc/maven.html
  [3]: https://www.jacoco.org/jacoco/trunk/doc/check-mojo.html
  [4]: https://www.jacoco.org
  [5]: /2020/11/maven-lifecycles/
  [6]: https://en.wikipedia.org/wiki/Cyclomatic_complexity