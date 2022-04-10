---
date: 2022-04-07
linktitle: JaCoCo Maven plugin - Dump Goal Configurations
prev: /2022/04/jacoco-maven-plugin-report-goal-configurations/
next: /2022/04/jacoco-maven-plugin-report-aggregate-goal-configurations/
title: JaCoCo Maven plugin - Dump Goal Configurations
weight: 10
tags: [ "maven", "devops", "tools", "coverage"]
categories: [ "DevOps", "Maven", "Tools", "Code Coverage" ]
---


[Code coverage][1] is a metric that can help you understand how much of your source is tested. It's a very useful metric that can help you assess the quality of your test suite. A program with high test coverage has more of its source code executed during testing, which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage. So, it really depends on how well your tests are written.

In this article, we will be taking a look at how to generate code coverage data using [JaCoCo Maven plugin][2] with goal - [**dump**][3].

Here is video for the same.

{{< yt "https://www.youtube.com/embed/yqfvoEG4l5U" >}}





## JaCoCo

[JaCoCo][4] is short form for **Ja**va **Co**de **Co**verage. It provides two modes of instrumentation of the code.

- Java Agent - Instrumenting the bytecode on the fly while running the code with a Java Agent
- Offline - Coverage in offline mode(i.e. Prior to execution of code). This mode of code coverage isn’t quite popular and is typically used in special cases.




## JaCoCo Goal - dump

[**dump**][3] goal requests a code coverage dump over TCP/IP from a JaCoCo agent running in `tcpserver` mode. This is usually done when the application is run in a separate environment and the tests are run against it. And once all the tests are executed against the application, you can get the code coverage data generated and transferred over the TCP/IP.

In short, [**dump**][3] goal will connect to the JaCoCo agent running on the remote/local host, transfer the coverage data over TCP/IP and write the dump(or coverage) data to the configured output file.

Below are the configuration properties of this goal.

- `<address>` - IP address or hostname to connect to
- `<port>` - Port number to connect to(defaults to 6300)
- `<append>` - Whether to keep adding the coverage data to the destination file if it already exists
- `<destFile>` - File to which coverage dump data is written by the `dump` goal.
- `<dump>` - Sets whether the execution data should be downloaded from the remote host or not
- `<reset>` - Sets whether the agent should be reset after execution data has been dumped
- `<retryCount>` - Number of attempts to establish a connection with the remote JaCoCo agent
- `<skip>` - Flag to enable/disable goal execution


Let's run a simple [Spring Boot][5] application, generate its coverage data using the [**dump**][3] goal and view the reports.





## Project

We will use a very minimalist [Spring Boot][5] application which contains an endpoint that we can call. You can use [Spring Initializer][6] to generate this project. You need to just add `Spring Web` as the dependency and you should be good.

Also, check out my [previous article][7] on setting up [JaCoCo Maven plugin][2] to add the plugin to your `pom.xml`.

And finally, we will add an endpoint which says "Hello" when you call it. Here are the files of the project.

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
└── src
    ├── main/java/io/codejournal/maven/jacoco/dumpdemo
    │                                         └── Application.java
    |
    └── test/java/io/codejournal/maven/jacoco/dumpdemo
                                              └── ApplicationTests.java
</pre>
{{< /rawhtml >}}

Let's see the contents of the files. In the `pom.xml`, apart from the usual project co-ordinates and properties, we have dependency for `spring-boot-starter-web` which starts a Web Server, and then we have `spring-boot-starter-test` which provides support for testing Spring components.

For our build plugins, we have `spring-boot-maven-plugin` to allow running of Spring application via Maven, and then we have configured our [JaCoCo Maven plugin][2] with [prepare-agent][8] goal.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.6</version>
        <relativePath />
        <!-- lookup parent from repository -->
    </parent>

    <groupId>io.codejournal.jacoco</groupId>
    <artifactId>maven-jacoco-dump-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>maven-jacoco-dump-demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
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
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

And the `Application.java` and `ApplicationTests.java` are same as they got generated from Spring Initializer.

```java
package io.codejournal.maven.jacoco.dumpdemo;

import java.util.Optional;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
package io.codejournal.maven.jacoco.dumpdemo;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ApplicationTests {

	@Test
	void contextLoads() {
	}
}
```





{{< articlead >}}

## Adding the Say-Hello endpoint

Let's add a Rest endpoint which prints `Hello` when you call it. It can greet you with the name if you provide it via the query parameter `name`.

{{< rawhtml >}}
<div class="notification">To keep things concise, I am adding the <code>RestController</code> in the <code>Application.java</code> itself. However, you are free to structure the controller as per your liking.</div>
{{< /rawhtml >}}

```java
package io.codejournal.maven.jacoco.dumpdemo;

import java.util.Optional;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
class SayHelloController {

    @GetMapping("/say-hello")
    public String sayHello(@RequestParam(required = false) final String name) {
        return "Hello " + Optional.ofNullable(name).orElse("") + "\n";
    }
}
```

Let's build the executable jar file now. Run the `package` phase with tests skipped.

```
mvn clean package -Dmaven.test.skip=true
```

And the executable jar is available in the `target` folder.

{{< rawhtml >}}
<pre class="code-output">
-> ls -l target
total 17168
drwxrwxr-x 3 codejournal codejournal     4096 Apr  9 00:26 classes
drwxrwxr-x 3 codejournal codejournal     4096 Apr  9 00:26 generated-sources
drwxrwxr-x 3 codejournal codejournal     4096 Apr  9 00:26 generated-test-sources
drwxrwxr-x 2 codejournal codejournal     4096 Apr  9 00:26 maven-archiver
-rw-rw-r-- 1 codejournal codejournal 17539264 Apr  9 00:26 maven-jacoco-dump-demo-0.0.1-SNAPSHOT.jar   <--------------------- Executable Jar
-rw-rw-r-- 1 codejournal codejournal     4104 Apr  9 00:26 maven-jacoco-dump-demo-0.0.1-SNAPSHOT.jar.original
drwxrwxr-x 3 codejournal codejournal     4096 Apr  9 00:26 maven-status
drwxrwxr-x 2 codejournal codejournal     4096 Apr  9 00:26 surefire-reports
drwxrwxr-x 3 codejournal codejournal     4096 Apr  9 00:26 test-classes
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Running the Application with JaCoCo Javaagent

Now, it's time to run the application with JaCoCo agent. If you take a look at the build logs from the previous step(`mvn clean package -Dmaven.test.skip=true`), you will see that JaCoCo plugin execution for `prepare-agent` goal. Here is the log snippet.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.8:prepare-agent (jacoco-prepare-agent) @ maven-jacoco-dump-demo ---
[INFO] argLine set to -javaagent:/home/codejournal/.m2/repository/org/jacoco/org.jacoco.agent/0.8.8/org.jacoco.agent-0.8.8-runtime.jar=destfile=/home
/codejournal/workspace/maven-jacoco-dump-demo/target/jacoco.exec
...
</pre>
{{< /rawhtml >}}

This is the [**-javaagent**][9] argument that we need to pass when we run our application. Also, right now, this argument is set to write the coverage file when the application exists, but that is not what we want to do. We need to run the JaCoCo agent in `tcpserver` mode, so that later we can generate the coverage data using the `dump` goal.

{{< rawhtml >}}
<div class="notification">
Check out the documentation for <a href="https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-mojo.html" target="_blank"><code>prepare-agent</code></a> if you need to configure additional parameters. You can run <code>mvn clean package -Dmaven.test.skip=true</code> again to get the entire <code>-javaagent</code> argument line.
</div>
{{< /rawhtml >}}

Let's start our application. Here is the command.

```bash
java \
    -javaagent:/home/codejournal/.m2/repository/org/jacoco/org.jacoco.agent/0.8.8/org.jacoco.agent-0.8.8-runtime.jar=output=tcpserver \
    -jar target/maven-jacoco-dump-demo-0.0.1-SNAPSHOT.jar
```

You will see the Spring application start and [Tomcat][10] server will start listening for requests.

{{< rawhtml >}}
<pre class="code-output">
...
2022-04-09 00:54:44.266  INFO 99035 --- [   main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-04-09 00:54:44.275  INFO 99035 --- [   main] i.c.maven.jacoco.dumpdemo.Application    : Started Application in 2.041 seconds (JVM running for 2.573)
</pre>
{{< /rawhtml >}}





## Configuring dump goal

Let's now configure the `dump` goal and `report` goal for `jacoco-maven-plugin` in our `pom.xml`. Just like we have `<execution>` section for `prepare-agent`, we will add another section for these. Here is the final plugin configuration for `jacoco-maven-plugin`.

```xml
<!--[...] -->
    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.8</version>
        <executions>
            <execution>
                <id>jacoco-dump</id>
                <goals>
                    <goal>dump</goal>
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
<!--[...] -->
```

{{< rawhtml >}}
<div class="notification">
The <code>prepare-agent</code> goal was added to get the <code>-javaagent</code> argument line. We don't need it any longer and hence, I have removed it from the plugin configuration above. It is fine if you keep it as well.
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Generating Coverage data with dump goal

Now, let's run the build with skipping the tests.

```
mvn verify -Dmaven.test.skip=true
```

And with that, the `dump` goal is executed and the coverage file data is generated. And then `report` goal is executed to generate the report. Here are the logs from the build.

{{< rawhtml >}}
<pre class="code-output">
...
[INFO] --- jacoco-maven-plugin:0.8.8:dump (jacoco-dump) @ maven-jacoco-dump-demo ---
[INFO] Connecting to localhost/127.0.0.1:6300
[INFO] Dumping execution data to /home/codejournal/workspace/maven-jacoco-dump-demo/target/jacoco.exec
[INFO] 
[INFO] --- jacoco-maven-plugin:0.8.8:report (jacoco-report) @ maven-jacoco-dump-demo ---
[INFO] Loading execution data file /home/codejournal/workspace/maven-jacoco-dump-demo/target/jacoco.exec
[INFO] Analyzed bundle 'maven-jacoco-dump-demo' with 2 classes
...
</pre>
{{< /rawhtml >}}

Here is the report.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2022/04/maven-jacoco-dump-run1.png" alt="JaCoCo Maven plugin Dump - Coverage Report 1" />
</div>
{{< /rawhtml >}}

Looking at the report, we have two classes `Application` and `SayHelloController`. `Application` is covered 100% and `SayHelloController` is covered only 30%. That is because `sayHello` method is never executed until now. Let's call it now.

```
-> curl http://localhost:8080/say-hello?name=CodeJournal
Hello CodeJournal
```

You will see that there is some activity in the application logs as well.

{{< rawhtml >}}
<pre class="code-output">
...
2022-04-09 INFO 109782 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]   : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-04-09 INFO 109782 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet    : Initializing Servlet 'dispatcherServlet'
2022-04-09 INFO 109782 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet    : Completed initialization in 1 ms
</pre>
{{< /rawhtml >}}

And when we run `mvn verify -Dmaven.test.skip=true` again, we see another `dump` and `report` generated, and this time our coverage data for `SayHelloController` is 100%.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2022/04/maven-jacoco-dump-run2.png" alt="JaCoCo Maven plugin Dump - Coverage Report 2" />
</div>
{{< /rawhtml >}}

And that's how we generated the coverage data using the `dump` goal. We used minimalistic configuration to achieve this. Take a look at the [configuration properties for dump goal][3], if you need to set some of them for your use cases.



  [1]: https://en.wikipedia.org/wiki/Code_coverage
  [2]: https://www.jacoco.org/jacoco/trunk/doc/maven.html
  [3]: https://www.jacoco.org/jacoco/trunk/doc/dump-mojo.html
  [4]: https://www.jacoco.org
  [5]: https://spring.io/projects/spring-boot
  [6]: https://start.spring.io
  [7]: /2022/04/code-coverage-with-jacoco-maven-plugin/#configure-jacoco-plugin
  [8]: /2022/04/code-coverage-with-jacoco-maven-plugin/#jacoco-goal---prepare-agent
  [9]: https://docs.oracle.com/en/java/javase/17/docs/api/java.instrument/java/lang/instrument/package-summary.html
 [10]: https://tomcat.apache.org
