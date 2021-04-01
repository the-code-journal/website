---
date: 2021-02-21
linktitle: Generating Code using Maven - Swagger to Java
next: /2021/03/unit-testing-in-maven-pojo-tests/
prev: /2021/01/generating-code-using-maven-soap-wsdl-to-java/
title: Generating Code using Maven - Swagger to Java
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this article, we will talk about generating Java code from [Swagger][1] files, which are essentially API specification files for a Web Service.

Below video discusses this with the demo.

{{< yt "https://www.youtube.com/embed/C3myXuX-l8Y" >}}





## Open API

In our [previous article][2], we talked about [SOAP][3] services. The challenge with SOAP services is that they tend to be heavy because of XML data usage, and there is a very tight coupling between the client and server. With time, [REST Web Services][4] have become the de-facto standard for Web Services, because of their extensibility.

REST APIs are documented using the [Open API][5] specifications. [Open API][5] is a specification for machine-readable interface files for describing, producing, consuming and visualizing RESTful Web Services. These were earlier called swagger specifications when they were developed by [SmartBear Software]. Because of a standard specification, APIs created with this have a common structure and it allows to build tooling around the APIs to cut down on the amount of boilerplate code.

Let's see an example Swagger API.





## Petstore Swagger API

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/02/maven-swagger-petstore-api.png" alt="Swagger Petstore API" style="height:700px" />
</div>
{{< /rawhtml >}}

[Petstore API][6] is the example Swagger API hosted at [swagger.io][7]. The API allows you to manage pets in a store. The API can be viewed in the [Swagger UI][8], which gives a very detailed view of the API, the available operations, the request parameters, headers, authorization, response, all of it. It becomes a handy document for reference as well.



## Swagger-Codegen

Since, Swagger API has well-defined structure, it can be used to generate Java classes for Rest API service and get the API integration going in no time. [Swagger Codegen][9] is designed for exact same purpose. 

Its comes as a standalone binary as well as Maven plugin, to generate the code, and that is what we will use for the [Petstore API][6] demo.





{{< articlead >}}

## Project

The initial project used in this article is [available here][10].

Here are the contents of the Project

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
</pre>
{{< /rawhtml >}}

Only `pom.xml` exists with bare minimum configuration below.

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>io.codejournal.maven</groupId>
  <artifactId>maven-generate-code-swagger-to-java</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
  </properties>

</project>
```

It has project coordinates, and Java compiler properties.





{{< articlead >}}

## Download petstore.json

Let's first download the JSON file for Petstore API and save it to `src/main/resources/petstore.json`.

{{< rawhtml >}}
<pre class="code-output">
-> curl https://petstore.swagger.io/v2/swagger.json -o src/main/resources/petstore.json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13843    0 13843    0     0  14586      0 --:--:-- --:--:-- --:--:-- 14571
</pre>
{{< /rawhtml >}}





## Swagger Codegen Maven Plugin

There are currently 2 versions of Open API specifications which are being used. And thus, Swagger Codegen Maven Plugin is also supported for both of those versions - [Verxion 2.x][11] and [Version 3.x][12], which support Open API v2 and v3 specifications respectively. We will use the latest [Version 3.x][12].

To configure the [swagger-codegen plugin][12], add the below configuration.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.swagger.codegen.v3</groupId>
      <artifactId>swagger-codegen-maven-plugin</artifactId>
      <version>3.0.25</version>
      <executions>
        <execution>
          <goals>
            <goal>generate</goal>
          </goals>
          <configuration>
            <inputSpec>${project.basedir}/src/main/resources/petstore.yaml</inputSpec>
            <language>java</language>
            <library>resttemplate</library>
            <apiPackage>io.codejournal.maven.swagger2java.api</apiPackage>
            <modelPackage>io.codejournal.maven.swagger2java.model</modelPackage>
            <invokerPackage>io.codejournal.maven.swagger2java.handler</invokerPackage>
            <generateApiTests>false</generateApiTests>
            <generateApiDocumentation>false</generateApiDocumentation>
            <generateModelTests>false</generateModelTests>
            <generateModelDocumentation>false</generateModelDocumentation>
            <generateSupportingFiles>true</generateSupportingFiles>
            <configOptions>
              <interfaceOnly>true</interfaceOnly>
              <dateLibrary>java8</dateLibrary>
            </configOptions>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Let's dissect the above configuration.

We have the plugin coordinates with `groupId, artifactId` and `version`. With `execution` we specify what needs to be done by the plugin. The plugin automatically attaches to the `generate-sources` [lifecycle phase][13], and the goal to execute is `generate`.

Let's see the `configuration` section now.

- `inputSpec` - Location of the Open API specification file(json or yaml)
- `language` - Programming Language in which the code should be generated. The full list can be looked up [here][14].
- `library` - Libraries used to generate the classes. For Java, options are `jersey1, jersey2, feign, okhttp-gson(default), retrofit, retrofit2, google-api-client, rest-assured, resttemplate`. Since, a lot of applications use Spring Boot, it makes sense to use the Spring Web Rest Template, and that is what we configured.
- `apiPackage` - Package where classes for the API alone are generated
- `modelPackage` - Package where request, response classes(model classes) are generated
- `invokerPackage` - Package where classes that should be used to make the calls to the API and get parsed resposne
- `generateApiTests` - Generation of API test classes
- `generateApiDocumentation` - Generate API documentation files
- `generateModelTests` - Generate Model test classes
- `generateModelDocumentation` - Generate Model documentation files
- `generateSupportingFiles` - Generate supporting files(invoker client classes) and other files (like `pom.xml, gradle.properties, build.sbt`)

Config Options are not that well documented, but we can take a [look at the code][15] to figure out what options are supported.
- `interfaceOnly` - This ensures that we are only generating the interface classes for API client and we don't need the server classes.
- `dateLibrary` - Date library to use. By default, [joda][16] is used, but you can set the value as `java8` to use [Java 8's new Date-Time][17] classes


{{< articlead >}}


Ok. Let's run the build to generate the classes and see what files are generated - `mvn clean generate-sources`.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean generate-sources
[INFO] Scanning for projects...
[INFO] 
[INFO] ------< io.codejournal.maven:maven-generate-code-swagger-to-java >------
[INFO] Building maven-generate-code-swagger-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-swagger-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-swagger-to-java/target
[INFO] 
[INFO] --- swagger-codegen-maven-plugin:3.0.25:generate (default) @ maven-generate-code-swagger-to-java ---
[WARNING] Output directory does not exist, or is inaccessible. No file (.swagger-codegen-ignore) will be evaluated.
[WARNING] ApiResponse (reserved word) cannot be used as model name. Renamed to ModelApiResponse
...
...
...
[INFO] writing file /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/.swagger-codegen/VERSION
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.395 s
[INFO] Finished at: 2021-02-21T17:07:43+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

The files are generated inside `target/generated-sources/swagger/` directory.

{{< rawhtml >}}
<pre class="code-output">
-> tree target/generated-sources/swagger/
target/generated-sources/swagger/
├── build.gradle                   ────┐
├── build.sbt                          |
├── git_push.sh                        |
├── gradle                             |
│   └── wrapper                        |
│       ├── gradle-wrapper.jar         |
│       └── gradle-wrapper.properties  ├────» Other Supporting files
├── gradle.properties                  |
├── gradlew                            |
├── gradlew.bat                        |
├── pom.xml                            |
├── README.md                          |
├── settings.gradle                ────┘
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── io
                └── codejournal
                    └── maven
                        └── swagger2java
                            ├── api
                            │   ├── PetApi.java        ────┐
                            │   ├── StoreApi.java          ├────» API Client classes
                            │   └── UserApi.java       ────┘
                            ├── handler
                            │   ├── ApiClient.java           ────┐
                            │   ├── auth                         |
                            │   │   ├── ApiKeyAuth.java          |
                            │   │   ├── Authentication.java      ├────» Library Client classes(Supporting files)
                            │   │   ├── HttpBasicAuth.java       |
                            │   │   ├── OAuthFlow.java           |
                            │   │   └── OAuth.java               |
                            │   └── RFC3339DateFormat.java   ────┘
                            └── model
                                ├── Body1.java          ────┐
                                ├── Body.java               |
                                ├── Category.java           |
                                ├── ModelApiResponse.java   ├────» Model classes
                                ├── Order.java              |
                                ├── Pet.java                |
                                ├── Tag.java                |
                                └── User.java           ────┘

</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Dependencies

The API classes are generated, but the project still doesnt compile, because we are missing the dependencies. Let's configure these now as below.

```xml
<dependencies>
  <dependency>
    <groupId>io.swagger.core.v3</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>2.1.7</version>
  </dependency>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.4</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.4</version>
  </dependency>
  <dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
  </dependency>
</dependencies>
```

And now, if we compile the project, everything works well.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean compile
[INFO] Scanning for projects...
[INFO] 
[INFO] ------< io.codejournal.maven:maven-generate-code-swagger-to-java >------
[INFO] Building maven-generate-code-swagger-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-swagger-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-swagger-to-java/target
[INFO] 
[INFO] --- swagger-codegen-maven-plugin:3.0.25:generate (default) @ maven-generate-code-swagger-to-java ---
[WARNING] Output directory does not exist, or is inaccessible. No file (.swagger-codegen-ignore) will be evaluated.
[WARNING] ApiResponse (reserved word) cannot be used as model name. Renamed to ModelApiResponse
...
...
...
[INFO] writing file /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/.swagger-codegen/VERSION
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-swagger-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-swagger-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 19 source files to /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/classes
[WARNING] /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java: /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java uses or overrides a deprecated API.
[WARNING] /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java: Recompile with -Xlint:deprecation for details.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.010 s
[INFO] Finished at: 2021-02-21T17:32:12+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Testing Client

Now, that our client classes are generated and getting compiled, let's write a small code to make the API call and get response. We will use the endpoint - `findPetsByStatus(AVAILABLE)`

```java
package io.codejournal.maven.swagger2java;

import static java.util.Collections.singletonList;
import static io.codejournal.maven.swagger2java.model.Pet.StatusEnum.AVAILABLE;

import java.util.List;

import io.codejournal.maven.swagger2java.api.PetApi;
import io.codejournal.maven.swagger2java.model.Pet;

public class PetStoreRunner {

    public static void main(final String[] args) {

        final PetApi api = new PetApi();

        final List<Pet> petsByStatus = api.findPetsByStatus(singletonList(AVAILABLE.getValue()));

        petsByStatus.forEach(System.out::println);
    }
}
```

And let's run it now to get the response.

{{< rawhtml >}}
<pre class="code-output">

-> mvn clean compile exec:java -Dexec.mainClass="io.codejournal.maven.swagger2java.PetStoreRunner"

[INFO] Scanning for projects...
[INFO] 
[INFO] ------< io.codejournal.maven:maven-generate-code-swagger-to-java >------
[INFO] Building maven-generate-code-swagger-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-swagger-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-swagger-to-java/target
[INFO] 
[INFO] --- swagger-codegen-maven-plugin:3.0.25:generate (default) @ maven-generate-code-swagger-to-java ---
[WARNING] Output directory does not exist, or is inaccessible. No file (.swagger-codegen-ignore) will be evaluated.
...
...
...
[INFO] writing file /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/.swagger-codegen/VERSION
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-swagger-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-swagger-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 19 source files to /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/classes
[WARNING] /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java: /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java uses or overrides a deprecated API.
[WARNING] /home/codejournal/workspace/maven-generate-code-swagger-to-java/target/generated-sources/swagger/src/main/java/io/codejournal/maven/swagger2java/handler/RFC3339DateFormat.java: Recompile with -Xlint:deprecation for details.
[INFO] 
[INFO] --- exec-maven-plugin:3.0.0:java (default-cli) @ maven-generate-code-swagger-to-java ---

class Pet {
    id: 9222968140497556423
    category: class Category {
        id: 0
        name: string
    }
    name: fish
    photoUrls: [string]
    tags: [class Tag {
        id: 0
        name: string
    }]
    status: available
}
...
...
class Pet {
    id: 9222968140497559130
    category: class Category {
        id: 0
        name: string
    }
    name: fish
    photoUrls: [string]
    tags: [class Tag {
        id: 0
        name: string
    }]
    status: available
}

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.154 s
[INFO] Finished at: 2021-02-21T17:40:44+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

And we have the list of available pets in the store, displayed on the output.

The [working code][5] is available on github for reference.



  [1]: https://swagger.io/
  [2]: /2021/01/generating-code-using-maven-soap-wsdl-to-java/
  [3]: https://en.wikipedia.org/wiki/SOAP
  [4]: https://en.wikipedia.org/wiki/Representational_state_transfer
  [5]: https://www.openapis.org/
  [6]: https://petstore.swagger.io/
  [7]: https://swagger.io/
  [8]: https://swagger.io/tools/swagger-ui/
  [9]: https://github.com/swagger-api/swagger-codegen
 [10]: https://github.com/the-code-journal/maven-for-beginners/tree/main/008-generate-code-swagger-to-java/initial
 [11]: https://github.com/swagger-api/swagger-codegen/tree/master/modules/swagger-codegen-maven-plugin
 [12]: https://github.com/swagger-api/swagger-codegen/tree/3.0.0/modules/swagger-codegen-maven-plugin
 [13]: /2020/11/maven-lifecycles/
 [14]: https://swagger.io/docs/open-source-tools/swagger-codegen/#style-guide-26
 [15]: https://github.com/swagger-api/swagger-codegen/blob/068b1ebcb7b04a48ad38f1cadd24bb3810c9f1ab/modules/swagger-codegen/src/main/java/io/swagger/codegen/languages/SpringCodegen.java#L75
 [16]: https://www.joda.org/joda-time/
 [17]: https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html
 [18]: https://github.com/the-code-journal/maven-for-beginners/tree/main/008-generate-code-swagger-to-java/final
