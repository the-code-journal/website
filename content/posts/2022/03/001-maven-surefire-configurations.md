---
date: 2022-03-14
linktitle: Unit Testing in Maven - Surefire Configurations
next: /2022/03/integration-testing-in-maven-failsafe-configurations/
prev: /2021/06/build-it-with-spring-boot-crud-operations-part-5/
title: Unit Testing in Maven - Surefire Configurations
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


This is the last article with respect to Unit Testing in Maven, which is Surefire Configurations. While most times, [Surefire plugin][1] works out of the box and you won’t need any configurations in your build. However, when the project gets bigger, there are a large number of tests that run and sometimes it is beneficial to configure certain things to ensure the tests are run efficiently or you just don’t care about test anymore and don’t want test execution to be slowing you down. Not only it will help you to reduce your build time, but also allow you to better isolate test setups, and increase your productivity.

So, lets see some useful configuration options with [Surefire plugin][1]. Below is the video with the demo of these configurations.

{{< yt "https://www.youtube.com/embed/GAebeKHcc78" >}}





## Skipping Tests

Every now and then, you might want to skip running the tests. This can be because you already ran the tests in a previous step and running the tests again would be redundant, or you may be working post testing phases(like packaging of your artifacts or like building a Docker image or deploying to a Repository) and running the tests everytime takes up time. In such cases, you can simply skip the tests and make your builds faster.

### Skipping Tests in `pom.xml`

To skip running tests for a Maven project, simply set the `skipTests` property to `true`.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.0.0-M5</version>
  <configuration>
    <skipTests>true</skipTests>
  </configuration>
</plugin>
```

### Skipping Tests from Command Line via `skipTests`

While `<skipTests>true</skipTests>` is convinient, changing `pom.xml` everytime to enable/disable tests is cumbersome. To help you with that, Maven allows to skip tests via the `skipTests` argument.

```bash
$ mvn install -DskipTests
```

{{< rawhtml >}}
<div class="notification"><code>skipTests</code> however, does not skip the compilation of test classes. Use <code>maven.test.skip</code> instead.</div>
{{< /rawhtml >}}


### Skipping Tests from Command Line via `maven.test.skip`

`skipTests` argument above only skips the test execution. You can also save build time by skipping the compilation of Test classes by using the property - `maven.test.skip`. This is honored by Surefire, Failsafe and the Compiler plugin.

```bash
$ mvn install -Dmaven.test.skip=true
```





{{< articlead >}}

## Including/Excluding Tests

Sometimes, you want to include/exclude only certain tests. For example, there is a Test class which is deprecated and you don’t want to run its test anymore. Another example could be that you are working on a specific component(Service class) and want to run tests only for that specific component so that you are sure that your changes aren’t breaking it. And once your changes are complete, you can include tests for other components as well.

You can configure test class inclusions and/or exclusions using the `<includes>` and `<excludes>` sections of the Surefire configuration. Here is an example below.

```xml
<configuration>
  <includes>
    <include>**/codingbat/array1/*Test</include>
  </includes>
  <excludes>
    <exclude>**/codingbat/array1/M*</exclude>
  </excludes>
</configuration>
```

With above configuration, all Test classes that are inside package structure matching `**/codingbat/array1` will be included in the test execution as part of `includes` configuration, and then all classes that `start with M` within the same package, will be excluded as part of `excludes` configuration.

### Includes/Excludes using Regular Expressions

You can also use [regular expressions][2] as part of your `<include>/<exclude>` configurations. The regular expression pattern is specified with `%regex[__PATTERN__]%`. For example,

```xml
<include>%regex[.*(Cat|Dog).*Test.*]</include>
```





## Single Test

Including/excluding configuration of [Surefire Plugin][1] becomes part of your `pom.xml`. So, if you accidentally committed those changes, your CI/CD pipelines would run only those tests and that is not something you would be wanting.

[Surefire plugin][1] understands this and provides support to run just a single test class or method, without making any changes to `pom.xml`. This helps you to safely run the tests on which you are working on, without the need of running the whole test suite every time. You can provide the test configuration as part of `test` property in your maven command, and only those tests will be executed.

### Running Single Test class, All methods

For example, your Test class is named as `CircleTest`, you can run all the tests with below command

```bash
$ mvn test -Dtest=CircleTest
```

### Running Single Test class, Single method

If you only want to run a single test method from test class(`CircleTest`), you can use the `#` to specify the test method to run.

```bash
$ mvn test -Dtest=CircleTest#test1
```

### Running Single Test class, Multiple methods

You can also specify multiple methods to execute. Below will execute both, `test1` and `test2` methods from `CirlceTest`.

```bash
$ mvn test -Dtest=CircleTest#test1+test2
```

### Running Single Test class, Multiple methods with patterns

You can also use patterns to select the methods for executions. Below will execute all methods `starting with test` in `CircleTest` along with 

```bash
$ mvn test -Dtest=CircleTest#test* # All methods starting with test
$ mvn test -Dtest=CircleTest#test*+anothertest # All methods starting with test and method - anothertest
$ mvn test -Dtest=CircleTest#test?? # All methods starting with test and just two additional letters
```

### Going crazy

`test` property can take multiple class/method configurations and you can really go crazy with them. For example,

```bash
mvn test '-Dtest=???Test, *Test#test*One+testTwo???, %regex[#fast.*|slow.*], !Unstable*' test
```

{{< rawhtml >}}
<div class="notification"><code>!</code> is used to mark anything for exclusion in the example above.</div>
{{< /rawhtml >}}





{{< articlead >}}

## Parallel Tests

As your tests grow in size, your build time will increase and more often than not, it will be the test execution that will be contributing to the overall build time.

Thus to be able to run tests in parallel is very crucial and surefire provides extensive support for these.

{{< rawhtml >}}
<div class="notification">A quick side note here that unless your test executions take over 5 minutes or more, the benefits of parallel tests aren’t quite evident.</div>
{{< /rawhtml >}}

The core configuration for Parallel Tests are `parallel` property and `threadCount` property.
- `parallel` - Defines which component should be parallelized. Values can be `methods, classes, both, suites, suitesAndClasses, suitesAndMethods, classesAndMethods, all`
- threadCount - Defines how many threads should be used to run the components in parallel

To achieve the best performance for your tests, there are [some additional properties][3] to get the most out of your builds.

### JUnit 5 native support for Parallel Tests

[JUnit 5 supports parallel tests][4] natively and you can enable the parallel test execution by providing the following Junit properties.

- `junit.jupiter.execution.parallel.enabled` - Boolean flag to enable/disable parallel test execution
- `junit.jupiter.execution.parallel.mode.default` - Values can be `SAME_THREAD`(default) or `CONCURRENT`
- `junit.jupiter.execution.parallel.mode.classes.default` - Values can be `SAME_THREAD`(default) or `CONCURRENT`

Below are some example configurations

#### Execute all tests in parallel

```
junit.jupiter.execution.parallel.enabled = true
junit.jupiter.execution.parallel.mode.default = concurrent
```

#### Execute top-level classes in parallel but methods in same thread

```
junit.jupiter.execution.parallel.enabled = true
junit.jupiter.execution.parallel.mode.default = same_thread
junit.jupiter.execution.parallel.mode.classes.default = concurrent
```

#### Execute top-level classes sequentially but their methods in parallel

```
junit.jupiter.execution.parallel.enabled = true
junit.jupiter.execution.parallel.mode.default = concurrent
junit.jupiter.execution.parallel.mode.classes.default = same_thread
```





## Forcing tests to Single Thread with @NotThreadSafe

Sometimes, you want your tests to be run in single thread since the code under tests is not thread safe. In such cases, you can mark the classes with JCIP annotation `@NotThreadSafe`, and all such classes will be executed in a Single thread at the end. All other tests can safely leverage the parallel test execution provided by Surefire.

Just use project dependency `net.jcip:jcip-annotations:1.0` or another artifact `com.github.stephenc.jcip:jcip-annotations:1.0-1` with Apache License 2.0.

```xml
<dependencies>
<!-- [...] -->
     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.7</version><!-- 4.7 or higher -->
       <scope>test</scope>
     </dependency>
     <dependency>
       <groupId>com.github.stephenc.jcip</groupId>
       <artifactId>jcip-annotations</artifactId>
       <version>1.0-1</version>
       <scope>test</scope>
     </dependency>
<!-- [...] -->
</dependencies>
```





{{< articlead >}}

## Forked Tests

By default all tests are run in the same JVM as the build is running. However, to make use of even more parallelism and if your hardware can support, you can create additional JVMs that just run your tests. If your tests take a very long time and you want to cut down on the test execution time, forking should definitely help you.

Again, the core configurations around forked tests are below:
- `forkCount` - Maximum JVMs to spawn to execute tests. If you terminate the value with a `C`, that value will be multiplied with the number of available CPU cores in your system. For example `forkCount=2.5C` on a Quad-Core system will result in forking up to `10 concurrent JVM processes` that execute tests.
- `reuseForks` - Whether to reuse the forked JVM to execute the next tests. If set to `false`, the fork will be terminated and the new fork will be created for next test
- `argLine` - Can be used to specify additional parameters passed to the JVM process(such as memory settings)
- `systemPropertyVariables` - System properties passed to the JVM process

You can use `${surefire.forkNumber}` placeholder within the configuration to reference the runtime fork number of the JVM process.

The following is an example configuration that makes use of up to three forked processes that execute the tests and then terminate. A system property `databaseSchema` is passed to the processes, that shall specify the database schema to use during the tests. The values for that will be `MY_TEST_SCHEMA_1, MY_TEST_SCHEMA_2`, and `MY_TEST_SCHEMA_3` for the three JVM processes. Additionaly by specifying custom `workingDirectory` each of processes will be executed in a separate working directory to ensure isolation on file system level.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>3.0.0-M5</version>
  <configuration>
      <forkCount>3</forkCount>
      <reuseForks>true</reuseForks>
      <argLine>-Xmx1024m -XX:MaxPermSize=256m</argLine>
      <systemPropertyVariables>
          <databaseSchema>MY_TEST_SCHEMA_${surefire.forkNumber}</databaseSchema>
      </systemPropertyVariables>
      <workingDirectory>FORK_DIRECTORY_${surefire.forkNumber}</workingDirectory>
  </configuration>
</plugin>
```

To locate which tests got executed in which fork, you can specify the `reportsDirectory` so that each forked JVM process can write the test result files in their own separate directory.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>3.0.0-M5</version>
  <configuration>
    <forkCount>2</forkCount>
    <reuseForks>false</reuseForks>
    <reportsDirectory>target/surefire-reports-${surefire.forkNumber}</reportsDirectory>
  </configuration>
</plugin>
```



  [1]: https://maven.apache.org/surefire/maven-surefire-report-plugin/
  [2]: https://www.liquidweb.com/kb/what-are-regular-expressions/
  [3]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/fork-options-and-parallel-execution.html#Parallel_Test_Execution
  [4]: https://junit.org/junit5/docs/snapshot/user-guide/#writing-tests-parallel-execution
