---
date: 2021-06-13
linktitle: Unit Testing in Maven - JUnit HTML Report
next: /2020/08/install-apache-maven-on-linux/
prev: /2021/06/build-it-with-spring-boot-crud-operations-part-2/
title: Unit Testing in Maven - JUnit HTML Report
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In our [previous article][1], we configured and ran [TestNG][2] tests, and one of things which we saw which came with [TestNG][2] was that it generated a Webpage based report which we could use for better understanding and insights of our tests, that it executes. But, the same HTML report is not available to us when we run the [Junit][3] tests by default.

In this article, we will see how we can generate a HTML report to get the same understanding and insights for our [Junit][3] tests.

Below is the video with the Demo to generate HTML report for [Junit][3] tests using [Maven Surefire Report Plugin][4].

{{< yt "https://www.youtube.com/embed/Tn8u_j5zqU8" >}}





## Project

To have a report for [Junit][3] tests, we need a project which has [Junit][3] tests. You can use any of your projects or use the previous ones that we created in our [Unit Testing with Maven - Junit4][5] or [Unit Testing with Maven - Junit 5][6] articles.

Or, you can also use my project, [code-challenges-java][7] that I have for writing Java code for various coding platforms. There are a lot of tests in various packages, so the report generated will have some realistic data to play with.

Let's take a look at the `pom.xml` that I have.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.codejournal.code-challenges</groupId>
  <artifactId>code-challenges-java</artifactId>
  <version>1.0</version>
  <name>code-challenges-java</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.20</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.8.0-M1</version>
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

It's a pretty simplistics `pom.xml`. We have

- Project Co-ordinates
- Properties to set the Java version for Maven Compiler Plugin
- Dependencies - We have [`lombok`][8] for auto-generating getters and setters in some classes, `junit-jupiter-api` since our tests are [Junit 5][3], and then we have [`assertj-core`][9] to use assert methods from there.
- In the build section, we have [`maven-surefire-plugin`][10] plugin to run our tests.

Let's run the tests with `mvn clean test` to verify that everything is good and the tests are running just fine.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test
[INFO] Scanning for projects...
[INFO] 
[INFO] --------< io.codejournal.code-challenges:code-challenges-java >---------
[INFO] Building code-challenges-java 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ code-challenges-java ---
[INFO] Deleting /home/codejournal/workspace/code-challenges-java/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ code-challenges-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 188 source files to /home/codejournal/workspace/code-challenges-java/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ code-challenges-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 188 source files to /home/codejournal/workspace/code-challenges-java/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ code-challenges-java ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.java.codingbat.array1.MakeLastTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.232 s - in io.codejournal.java.codingbat.array1.MakeLastTest

...
[logs removed for reduce verbosity]
...

[INFO] Running io.codejournal.hackerrank.thirtydaysofcode.Day07ArraysTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 s - in io.codejournal.hackerrank.thirtydaysofcode.Day07ArraysTest
[INFO] Running io.codejournal.hackerrank.thirtydaysofcode.Day13AbstractClassesTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.001 s - in io.codejournal.hackerrank.thirtydaysofcode.Day13AbstractClassesTest
[INFO] 
[INFO] Results:
[INFO] 
[WARNING] Tests run: 1005, Failures: 0, Errors: 0, Skipped: 1
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  17.783 s
[INFO] Finished at: 2021-06-13T23:49:19+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Peeping inside the Surefire Test files

It's not that Surefire doesn't generate any reports for the tests. It's just that it doesn't generate a human readable output. It generates the test reports in the form of XML files, which can be used by any compliant processor to generate the final human readable reports. This part is usually take care by CI/CD pipelines where they detect(or configured) these report files, and then they are processed to generate the report in appropriate presentation format.

Here is a quick list of files generated under `target/surefire-reports/` directory.

{{< rawhtml >}}
<pre class="code-output">
-> ls -l target/surefire-reports/
total 2276
-rw-rw-r-- 1 codejournal codejournal  365 Jun 13 23:49 io.codejournal.hackerrank.intro.CurrencyFormatterTest.txt
-rw-rw-r-- 1 codejournal codejournal  349 Jun 13 23:49 io.codejournal.hackerrank.intro.DataTypesTest.txt

...
[logs removed for reduce verbosity]
...

-rw-rw-r-- 1 codejournal codejournal 7362 Jun 13 23:49 TEST-io.codejournal.codingbat.warmup2.StringXTest.xml
-rw-rw-r-- 1 codejournal codejournal 6885 Jun 13 23:49 TEST-io.codejournal.codingbat.warmup2.StringYakTest.xml
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Configure the Project site and JUnit HTML Reports

To generate the HTML reports, we need the [Maven Surefire Report Plugin][4]. The [usage page][11] of the plugin documentation provides how to configure these in `pom.xml`. Essentially, we need to provide the plugin co-ordinates under the `<reporting>` section. Here it is.

```xml
<reporting>
  <plugins>
    <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-report-plugin</artifactId>
    <version>3.0.0-M5</version>
    </plugin>
  </plugins>
</reporting>
```

Also, this reporting will be used when the project is running the [`site`][12] phase. So, we will also need to add the [`maven-site-plugin`][13] to our `<build>` section.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.9.1</version>
</plugin>
```

The final `pom.xml`, looks like below.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.codejournal.code-challenges</groupId>
  <artifactId>code-challenges-java</artifactId>
  <version>1.0</version>
  <name>code-challenges-java</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.20</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.8.0-M1</version>
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
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-site-plugin</artifactId>
        <version>3.9.1</version>
      </plugin>
    </plugins>
  </build>

  <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-report-plugin</artifactId>
        <version>3.0.0-M5</version>
      </plugin>
    </plugins>
  </reporting>

</project>
```

Time to run, and see the generated HTML report.





{{< articlead >}}

## Generate the Project site and JUnit HTML Reports

To generate the project site and the JUnit HTML report, we need to run run the [lifecycle - site][12], apart from running the `default` lifecycle.

Nothing complicated. Simply, run - `mvn clean test site`

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean test site
[INFO] Scanning for projects...
[INFO] 
[INFO] --------< io.codejournal.code-challenges:code-challenges-java >---------
[INFO] Building code-challenges-java 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ code-challenges-java ---
[INFO] Deleting /home/codejournal/workspace/code-challenges-java/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ code-challenges-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 188 source files to /home/codejournal/workspace/code-challenges-java/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ code-challenges-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 188 source files to /home/codejournal/workspace/code-challenges-java/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ code-challenges-java ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running io.codejournal.java.codingbat.array1.MakeLastTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.232 s - in io.codejournal.java.codingbat.array1.MakeLastTest

...
[logs removed for reduce verbosity]
...

[INFO] Running io.codejournal.hackerrank.thirtydaysofcode.Day07ArraysTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 s - in io.codejournal.hackerrank.thirtydaysofcode.Day07ArraysTest
[INFO] Running io.codejournal.hackerrank.thirtydaysofcode.Day13AbstractClassesTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.001 s - in io.codejournal.hackerrank.thirtydaysofcode.Day13AbstractClassesTest
[INFO] 
[INFO] Results:
[INFO] 
[WARNING] Tests run: 1005, Failures: 0, Errors: 0, Skipped: 1
[INFO] 
[INFO] 
[INFO] --- maven-site-plugin:3.9.1:site (default-site) @ code-challenges-java ---
[INFO] configuring report plugin org.apache.maven.plugins:maven-surefire-report-plugin:3.0.0-M5
[INFO] preparing maven-surefire-report-plugin:report report requires '[surefire]test' forked phase execution
[INFO] 
[INFO] >>> maven-surefire-report-plugin:3.0.0-M5:report > [surefire]test @ code-challenges-java >>>
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ code-challenges-java ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ code-challenges-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/codejournal/workspace/code-challenges-java/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ code-challenges-java ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:3.0.0-M5:test (default-test) @ code-challenges-java ---
[INFO] Skipping execution of surefire because it has already been run for this configuration
[INFO] 
[INFO] <<< maven-surefire-report-plugin:3.0.0-M5:report < [surefire]test @ code-challenges-java <<<
[INFO] 
[INFO] '[surefire]test' forked phase execution for maven-surefire-report-plugin:report report preparation done
[INFO] 3 reports detected for maven-surefire-report-plugin:3.0.0-M5: failsafe-report-only, report, report-only
[WARNING] Report plugin org.apache.maven.plugins:maven-project-info-reports-plugin has an empty version.
[WARNING] 
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING] 
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[INFO] configuring report plugin org.apache.maven.plugins:maven-project-info-reports-plugin:3.1.2
[INFO] 15 reports detected for maven-project-info-reports-plugin:3.1.2: ci-management, dependencies, dependency-info, dependency-management, distribution-management, index, issue-management, licenses, mailing-lists, modules, plugin-management, plugins, scm, summary, team
[INFO] Rendering site with default locale English (en)
[WARNING] No project URL defined - decoration links will not be relativized!
[INFO] Rendering content with org.apache.maven.skins:maven-default-skin:jar:1.3 skin.
[INFO] Skipped "Surefire Report" report (maven-surefire-report-plugin:3.0.0-M5:report-only), file "surefire-report.html" already exists.
[INFO] Generating "Surefire Report" report --- maven-surefire-report-plugin:3.0.0-M5:report
[WARNING] Unable to locate Test Source XRef to link to - DISABLED
[INFO] Generating "Dependencies" report  --- maven-project-info-reports-plugin:3.1.2:dependencies
[INFO] Generating "Dependency Information" report --- maven-project-info-reports-plugin:3.1.2:dependency-info
[INFO] Generating "About" report         --- maven-project-info-reports-plugin:3.1.2:index
[INFO] Generating "Plugin Management" report --- maven-project-info-reports-plugin:3.1.2:plugin-management
[INFO] Generating "Plugins" report       --- maven-project-info-reports-plugin:3.1.2:plugins
[INFO] Generating "Summary" report       --- maven-project-info-reports-plugin:3.1.2:summary
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  17.783 s
[INFO] Finished at: 2021-06-13T23:49:19+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

You can see some last few lines that the apart from running the tests, there are some reports that generated just before the build got completed.

The generates site is available in folder `target/site` directory. A quick look at its contents, and we see that there are bunch of HTML files generated and among those is `surefire-report.html`.

{{< rawhtml >}}
<pre class="code-output">
-> ls -l target/site/
total 420
drwxrwxr-x 2 codejournal codejournal   4096 Jun 14 00:13 css
-rw-rw-r-- 1 codejournal codejournal  14289 Jun 14 00:14 dependencies.html
-rw-rw-r-- 1 codejournal codejournal   4606 Jun 14 00:14 dependency-info.html
drwxrwxr-x 3 codejournal codejournal   4096 Jun 14 00:14 images
-rw-rw-r-- 1 codejournal codejournal   2928 Jun 14 00:14 index.html
-rw-rw-r-- 1 codejournal codejournal   3746 Jun 14 00:14 plugin-management.html
-rw-rw-r-- 1 codejournal codejournal   5123 Jun 14 00:14 plugins.html
-rw-rw-r-- 1 codejournal codejournal   4214 Jun 14 00:14 project-info.html
-rw-rw-r-- 1 codejournal codejournal   2851 Jun 14 00:14 project-reports.html
-rw-rw-r-- 1 codejournal codejournal   3886 Jun 14 00:14 summary.html
-rw-rw-r-- 1 codejournal codejournal 362689 Jun 14 00:14 surefire-report.html
</pre>
{{< /rawhtml >}}

Let's open this site in a browser. You can run a server as well on this directory - `python3 -m http.server -d target/site/` and navigate to http://localhost:8000/surefire-report.html, you can see the HTML report for your tests.

Below is a screenshot for reference.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/06/maven-unit-tests-junit-html-report.png" alt="Unit Testing with Maven - JUnit HTML Report" />
</div>
{{< /rawhtml >}}

And you can use this report for further insights to your tests. If any test is failing, or taking a lot of time, you can decide to some fixes or optimizations as required.



  [1]: /2021/04/unit-testing-in-maven-testng-tests/
  [2]: https://testng.org/
  [3]: https://junit.org/junit5/
  [4]: https://maven.apache.org/surefire/maven-surefire-report-plugin/
  [5]: /2021/03/unit-testing-in-maven-junit4-tests/
  [6]: /2021/03/unit-testing-in-maven-junit5-tests/
  [7]: https://github.com/the-code-journal/code-challenges-java
  [8]: https://projectlombok.org
  [9]: https://assertj.github.io/doc/
 [10]: https://maven.apache.org/surefire/maven-surefire-plugin/index.html
 [11]: https://maven.apache.org/surefire/maven-surefire-report-plugin/usage.html
 [12]: /2020/11/maven-lifecycles/#lifecycle---site
 [13]: https://maven.apache.org/plugins/maven-site-plugin/