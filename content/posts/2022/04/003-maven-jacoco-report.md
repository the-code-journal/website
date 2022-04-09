---
date: 2022-04-05
linktitle: JaCoCo Maven Plugin - Report Goal Configurations
prev: /2022/04/jacoco-maven-plugin-check-goal-configurations/
next: /2022/04/jacoco-maven-plugin-dump-goal-configurations/
title: JaCoCo Maven Plugin - Report Goal Configurations
weight: 10
tags: [ "maven", "devops", "tools", "coverage"]
categories: [ "DevOps", "Maven", "Tools", "Code Coverage" ]
---


[Code coverage][1] is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

In this article, we will be taking a look at how to generating code coverage report using [JaCoCo Maven Plugin][2] with goal - [**report**][3]. We will also take a look at the various configuration options available as part of the [**report**][3] goal.

Here is video for the same.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## JaCoCo

[JaCoCo][4] is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code.

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.





## JaCoCo Goal - report

[**report**][3] goal creates a code coverage report for tests of a single project in multiple formats (HTML, XML, and CSV). It binds to `verify` phase of the [default lifecycle][5]. Here is how you will configure it in your `pom.xml`.

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
                <id>jacoco-report</id>
                <goals>
                    <goal>report</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
<!--[...]-->
```

And then when you run your build(`mvn clean verify`), you should see your reports under `target/site/jacoco` folder.





## JaCoCo Goal - report Configurations

Since, this goal is most widely used, let's discuss its configuration parameters.

### &lt;formats>

Report format to generate. Supported reports are - HTML, CSV and XML. By default, all reports are generated. Here is an example which only generates XML report

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

### &lt;includes> / &lt;excludes>

A list of class files to include/exclude in the report. You may use wildcard characters (* and ?) as part of the configuration.

Below is an example in which, all classes which has `jacocodemo` in it is package are included except if they have `jacocodemo/strings` in them.

```xml
<execution>
    <id>jacoco-report</id>
    <goals>
        <goal>report</goal>
    </goals>
    <configuration>
        <includes>
            <include>**/jacocodemo/**/*</include>
        </includes>
        <excludes>
            <exclude>**/jacocodemo/strings/*</exclude>
        </excludes>
    </configuration>
</execution>
```

### &lt;dataFile>

Data file with execution data for which report is generated. Default file that JaCoCo looks is `${project.build.directory}/jacoco.exec`

### &lt;outputDirectory>

Output directory for the reports. The default path is `${project.reporting.outputDirectory}/jacoco`. _Note that this parameter is only relevant if the goal is run from the command line or from the [default build lifecycle][5]. If the goal is run indirectly as part of a site generation, the output directory configured in the [Maven Site Plugin][6] is used instead._

### &lt;skip>

Flag used to enable/disable goal execution

### &lt;title

Name of the root node in HTML report pages. The default value is `${project.name}`

### &lt;footer>

Footer text used in the HTML report pages.



  [1]: https://en.wikipedia.org/wiki/Code_coverage
  [2]: https://www.jacoco.org/jacoco/trunk/doc/maven.html
  [3]: https://www.jacoco.org/jacoco/trunk/doc/report-mojo.html
  [4]: https://www.jacoco.org
  [5]: /2020/11/maven-lifecycles/
  [6]: https://maven.apache.org/plugins/maven-site-plugin/
