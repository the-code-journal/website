---
date: 2022-03-14
linktitle: Code Coverage with Maven Jacoco Plugin
next: /2020/08/install-apache-maven-on-linux/
prev: /2022/03/integration-testing-in-maven-failsafe-configurations/
title: Code Coverage with Maven Jacoco Plugin
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---


While [Unit Tests][1] provide a huge confidence in the code that you write, the importance of having [Integration Tests][2] is still unquestionable. [Integration tests][2] allow you to test the components when they interact with each other. [Maven Failsafe plugin][3] is designed to run integration tests. In our video today, we will see how to run this plugin and see some common configurations that you might use in your projects.

{{< yt "https://www.youtube.com/embed/wEr_SXkWZ24" >}}





## Failsafe Plugin

[Maven Failsafe plugin][3] provides 2 goals.

- `integration-test` - Run integration tests
- `verify` - Verify that the integration tests are passing

To add [Maven Failsafe plugin][3] to your project, simple add the following in your `pom.xml`.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>3.0.0-M5</version> <!-- Use the latest version if possible -->
  <executions>
    <execution>
      <goals>
        <goal>integration-test</goal>
        <goal>verify</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

And then to run the integration tests, simply run the `verify` phase of the build lifecycle.

```
mvn clean verify
```





{{< articlead >}}

## Skipping Tests

Every now and then, you might want to skip running the integration tests. This can be because you already ran the tests in a previous step and running the tests again would be redundant, or you may be working post testing phases(like packaging of your artifacts or like building a Docker image or deploying to a Repository) and running the tests everytime takes up time. In such cases, you can simply skip the tests and make your builds faster.

### Skipping Tests in pom.xml

To skip running tests for a Maven project, simply set the `skipITs` property to `true`.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>3.0.0-M5</version>
  <configuration>
    <skipITs>true</skipITs>
  </configuration>
</plugin>
```

### Skipping Tests from Command Line via `skipITs`

While `<skipITs>true</skipITs>` is convinient, changing `pom.xml` everytime to enable/disable tests is cumbersome. To help you with that, Maven allows to skip integration tests via the `skipITs` argument.

```bash
$ mvn install -DskipITs
```

{{< rawhtml >}}
<div class="notification"><code>skipITs</code> however, does not skip the compilation of test classes. Use <code>maven.test.skip</code> instead. This will also disable the unit tests run by Surefire plugin.</div>
{{< /rawhtml >}}


### Skipping Tests from Command Line via maven.test.skip

`skipITs` argument above only skips the test execution. You can also save build time by skipping the compilation of Test classes by using the property - `maven.test.skip`. This is honored by Surefire, Failsafe and the Compiler plugin.

```bash
$ mvn install -Dmaven.test.skip=true
```

Check the [official documentation][4] for more.





{{< articlead >}}

## Including/Excluding Tests

Sometimes, you want to include/exclude only certain tests. For example, there is a Test class which is deprecated and you don’t want to run its test anymore. Another example could be that you are working on a specific component(Service class) and want to run integration tests only for that specific component so that you are sure that your changes aren’t breaking it. And once your changes are complete, you can include integration tests for other components as well.

You can configure test class inclusions and/or exclusions using the `<includes>` and `<excludes>` sections of the Failsafe configuration. Here is an example below.

```xml
<configuration>
  <includes>
    <include>**/codingbat/array1/*TestIT</include>
  </includes>
  <excludes>
    <exclude>**/codingbat/array1/M*TestIT</exclude>
  </excludes>
</configuration>
```

With above configuration, all Test classes ending with `TestIT` that are inside package structure matching `**/codingbat/array1` will be included in the test execution as part of `includes` configuration, and then all classes that `start with M` within the same package and ending with `TestIT`, will be excluded as part of `excludes` configuration.

### Includes/Excludes using Regular Expressions

You can also use [regular expressions][8] as part of your `<include>/<exclude>` configurations. The regular expression pattern is specified with `%regex[__PATTERN__]%`. For example,

```xml
<include>%regex[.*(Circle|Square).*TestIT.*]</include>
```

Check the [official documentation][5] for more.





{{< articlead >}}

## Single Test

Including/excluding configuration of [Failsafe Plugin][3] becomes part of your `pom.xml`. So, if you accidentally committed those changes, your CI/CD pipelines would run only those integration tests and that is not something you would be wanting.

[Failsafe plugin][3] understands this and provides support to run just a single test class or method, without making any changes to `pom.xml`. This helps you to safely run the tests on which you are working on, without the need of running the whole test suite every time. You can provide the test configuration as part of `test` property in your maven command, and only those integration tests will be executed.

### Running Single Test class, All methods

For example, your Test class is named as `CircleTestIT`, you can run all the tests with below command

```bash
$ mvn verify -Dit.test=CircleTestIT
```

### Running Single Test class, Single method

If you only want to run a single test method from test class(`CircleTestIT`), you can use the `#` to specify the test method to run.

```bash
$ mvn verify -Dit.test=CircleTestIT#test1
```

### Running Single Test class, Multiple methods

You can also specify multiple methods to execute. Below will execute both, `test1` and `test2` methods from `CircleTestIT`.

```bash
$ mvn verify -Dit.test=CircleTestIT#test1+test2
```

### Running Single Test class, Multiple methods with patterns

You can also use patterns to select the methods for executions. Below will execute all methods `starting with test` in `CircleTestIT` along with 

```bash
$ mvn verify -Dit.test=CircleTestIT#test* # All methods starting with test
$ mvn verify -Dit.test=CircleTestIT#test*+anothertest # All methods starting with test and method - anothertest
$ mvn verify -Dit.test=CircleTestIT#test?? # All methods starting with test and just two additional letters
```

### Going crazy

`it.test` property can take multiple class/method configurations and you can really go crazy with them. For example,

```bash
mvn verify '-Dit.test=???Test, *Test#test*One+testTwo???, %regex[#fast.*|slow.*], !Unstable*'
```

{{< rawhtml >}}
<div class="notification"><code>!</code> is used to mark anything for exclusion in the example above.</div>
{{< /rawhtml >}}

Check the [official documentation][6] for more.





{{< articlead >}}

## Parallel Tests

As your tests grow in size, your build time will increase and more often than not, it will be the test execution that will be contributing to the overall build time.

Thus to be able to run tests in parallel is very crucial and failsafe provides extensive support for these.

{{< rawhtml >}}
<div class="notification">A quick side note here that unless your test executions take over 5 minutes or more, the benefits of parallel tests aren’t quite evident.</div>
{{< /rawhtml >}}

The core configuration for Parallel Tests are `parallel` property and `threadCount` property.
- `parallel` - Defines which component should be parallelized. Values can be `methods, classes, both, suites, suitesAndClasses, suitesAndMethods, classesAndMethods, all`
- threadCount - Defines how many threads should be used to run the components in parallel

To achieve the best performance for your tests, there are [some additional properties][7] to get the most out of your builds.





## Forked Tests

By default all tests are run in the same JVM as the build is running. However, to make use of even more parallelism and if your hardware can support, you can create additional JVMs that just run your tests. If your tests take a very long time and you want to cut down on the test execution time, forking should definitely help you.

Again, the core configurations around forked tests are below:
- `forkCount` - Maximum JVMs to spawn to execute tests. If you terminate the value with a `C`, that value will be multiplied with the number of available CPU cores in your system. For example `forkCount=2.5C` on a Quad-Core system will result in forking up to `10 concurrent JVM processes` that execute tests.
- `reuseForks` - Whether to reuse the forked JVM to execute the next tests. If set to `false`, the fork will be terminated and the new fork will be created for next test
- `argLine` - Can be used to specify additional parameters passed to the JVM process(such as memory settings)
- `systemPropertyVariables` - System properties passed to the JVM process

You can use `${surefire.forkNumber}` placeholder within the configuration to reference the runtime fork number of the JVM process.

The following is an example configuration that makes use of up to three forked processes that execute the tests and then terminate. A system property `databaseSchema` is passed to the processes, that shall specify the database schema to use during the tests. The values for that will be `MY_TEST_SCHEMA_1, MY_TEST_SCHEMA_2`, and `MY_TEST_SCHEMA_3` for the three JVM processes. Additionaly by specifying custom `workingDirectory` each of processes will be executed in a separate working directory to ensure isolation on file system level. And finally, to be able to identify which tests ran in which fork, we specify the `reportsDirectory` so that each fork can write the test result in their own separate directory.

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
      <reportsDirectory>target/failsafe-reports-${surefire.forkNumber}</reportsDirectory>
  </configuration>
</plugin>
```

Check the [official documentation][8] for more.



  [1]: https://en.wikipedia.org/wiki/Unit_testing
  [2]: https://en.wikipedia.org/wiki/Integration_testing
  [3]: https://maven.apache.org/surefire/maven-failsafe-plugin/
  [4]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/skipping-tests.html
  [5]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/inclusion-exclusion.html
  [6]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/single-test.html
  [7]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html#Parallel_Test_Execution
  [8]: https://www.liquidweb.com/kb/what-are-regular-expressions/
  [9]: https://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html#Forked_Test_Execution
