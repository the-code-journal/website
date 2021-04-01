---
date: 2021-01-08
linktitle: Generating Code using Maven - Java to JSON Schema
next: /2021/01/generating-code-using-maven-json-schema-to-java/
prev: /2020/12/generating-code-using-maven-xsd-to-java/
title: Generating Code using Maven - Java to JSON Schema
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this tutorial,  we will talk about how Maven can be used to generate [JSON schema][1] files for our Java POJO classes, which are really useful if you are generating or processing JSON data files.

Below is the video which demos generating JSON Schema from Java classes.

{{< yt "https://www.youtube.com/embed/qPINhGXKNxA" >}}






## JSON

[JSON][2] stands for JavaScript Object Notation, and it is another data-exchange format. Just like we can share data in a XML, we can use JSON notation to represent the same information. JSON uses name-value pair wherein the object properties are represented as names and value is associated to the name property using colon(:) symbol.

For example, below is the Employee data represented in both XML and JSON for comparison

{{< rawhtml >}}
<pre class="code-output">
+---------------------------+                   +--------------------------+
|             XML           |                   |          JSON            |
+---------------------------+                   +--------------------------+
|                           |                   | {                        |
| &lt;employees&gt;               |                   |   "employees": [         |
|   &lt;employee&gt;              |                   |     {                    |
|     &lt;id&gt;1001&lt;/id&gt;         |                   |       "id": 1001,        |
|     &lt;name&gt;John Doe&lt;/name&gt; |                   |       "name": "John Doe" |
|   &lt;/employee&gt;             |                   |     },                   |
|   &lt;employee&gt;              |                   |     {                    |
|     &lt;id&gt;1002&lt;/id&gt;         |                   |       "id": 1002,        |
|     &lt;name&gt;Foo Bar&lt;/name&gt;  |                   |       "name": "Foo Bar"  |
|   &lt;/employee&gt;             |                   |     }                    |
| &lt;/employees&gt;              |                   |   ]                      |
|                           |                   | }                        |
+---------------------------+                   +--------------------------+
</pre>
{{< /rawhtml >}}

#### Dense(XML) vs Sparse(JSON)

If you compare the XML and JSON representation of the employee data here, you can very well realize that JSON representation is much more clear and XML structure is very dense. That is why, JSON is much easy to read and quickly, while XML data takes a little bit of effort.

#### Bytes used

The dense and sparse nature of these files also reflects in the amount of bytes used. While the example here in XML takes about `166 bytes`, the same JSON data uses only `139 bytes`(about [**19% smaller**]()). And with more data(tens and thousands of employees or even more), XML data usually gets really really big, while the same JSON representation is much smaller.

There are more things that we can compare, but these two are striking difference between XML and JSON. Also, that is why JSON has become really popular these days and as a preferable data-exchange format.





{{< articlead >}}

## [JSON Schema][1]

Just like XML, JSON alone cannot provide information about the structure of a JSON document and thus the need of [JSON Schema][1] is inevitable.

Here is the JSON Schema for the Employee data on above. It is not the exact schema, but you will get a feel of it and would make it easier to understand. The employee data is represented as an object and it has two properties - `id` and `name`. `id` is of type `integer` and `name` is of type `string`. Very similar to XML Schema, but a lot less dense and easy to skim.

```json
{
  "type" : "object",
  "properties" : {
    "id" : {
      "type" : "integer"
    },
    "name" : {
      "type" : "string"
    }
  }
}
```

Don’t worry if you don’t understand anything. On a high level, this schema defines a type as `employee`, which has two elements - `id` and `name`. `id` is of type `integer`, while `name` element is of type `string`.

As you see, this gives us more insights about the XML file and we can be sure of certain things, rather than guessing those from the XML file itself.





## Java to JSON Schema

Just like in [Java to XSD article][3], whenever you are providing or ingesting XML data, XSD files provides a lot of insight about the structure of data. Similarly, whenever you are generating or processing JSON data, JSON Schema files will provide with the exact same benefits.

Let’s look at the left part of this processing pipeline, where we generate the Json Schema for our Java classes automatically during our build with the help of Maven plugin.


{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/01/maven-java-to-json-schema.png" alt="Maven Java to JSON Schema" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Project

The initial project used in this article is [available here][4].

Here are the contents of the Project

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
└── src
    └── main
        ├── java
        │   └── io
        │       └── codejournal
        │           └── maven
        │               └── java2jsonschema
        │                   ├── Department.java
        │                   └── Employee.java
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
    <artifactId>maven-generate-code-java-to-json-schema</artifactId>
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

## The POJO classes

The `Employee.java` and `Department.java` are two POJO classes for which we will be generating the JSON schema for. Here they are.

```java
package io.codejournal.maven.java2jsonschema;

public class Employee {

    private int id;

    private String name;

    // Getters
    // Setters
}
```

```java
package io.codejournal.maven.java2xsd;

public class Department {

    private int id;

    private String name;

    // Getters
    // Setters
}
```





{{< articlead >}}

## Maven Jaxb Plugin

We will be using [json-schema-generator-maven-plugin][5] to help our use case. It has just one goal - `generate-json-schemas`, which generates the JSON schema for the Java classes. Let's configure this.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>io.gravitee.maven.plugins</groupId>
            <artifactId>json-schema-generator-maven-plugin</artifactId>
            <version>1.3.0</version>
            <executions>
                <execution>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>generate-json-schemas</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <includes>
                    <include>io/codejournal/maven/java2jsonschema/*.class</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

We added the plugin and configured it to run during the `prepare-package` phase.

And to tell the plugin, the classes for which JSON Schema needs to be generated, we use the `<includes>` section.

There are more configuration options on the [plugin documentation page here][5].

And that should do it. When you run the build, you should be getting the JSON Schema for `Employee.java` and `Department.java`.

Here is a sample run.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean package
[INFO] Scanning for projects...
[INFO] 
[INFO] ----< io.codejournal.maven:maven-generate-code-java-to-json-schema >----
[INFO] Building maven-generate-code-java-to-json-schema 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-java-to-json-schema ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-java-to-json-schema/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-java-to-json-schema ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-java-to-json-schema ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /home/codejournal/workspace/maven-generate-code-java-to-json-schema/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven-generate-code-java-to-json-schema ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven-generate-code-java-to-json-schema ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven-generate-code-java-to-json-schema ---
[INFO] 
[INFO] --- json-schema-generator-maven-plugin:1.3.0:generate-json-schemas (default) @ maven-generate-code-java-to-json-schema ---
[INFO] Created JSON Schema: /home/codejournal/workspace/maven-generate-code-java-to-json-schema/target/classes/schemas/urn_jsonschema_io_codejournal_maven_java2jsonschema_Employee.json
[INFO] Created JSON Schema: /home/codejournal/workspace/maven-generate-code-java-to-json-schema/target/classes/schemas/urn_jsonschema_io_codejournal_maven_java2jsonschema_Department.json
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ maven-generate-code-java-to-json-schema ---
[INFO] Building jar: /home/codejournal/workspace/maven-generate-code-java-to-json-schema/target/maven-generate-code-java-to-json-schema-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.667 s
[INFO] Finished at: 2021-01-08T23:47:48+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

Now, if we look inside the `target/classes/schemas` directory, we will get our schema files.

{{< rawhtml >}}
<pre class="code-output">
-> tree target/classes/schemas/
target/classes/schemas/
├── urn_jsonschema_io_codejournal_maven_java2jsonschema_Department.json      &lt;-------- Schema File
└── urn_jsonschema_io_codejournal_maven_java2jsonschema_Employee.json        &lt;-------- Schema File
</pre>
{{< /rawhtml >}}


Below is the generated schema for `Employee.java`.

```json
{
  "type" : "object",
  "id" : "urn:jsonschema:io:codejournal:maven:java2jsonschema:Employee",
  "properties" : {
    "id" : {
      "type" : "integer"
    },
    "name" : {
      "type" : "string"
    }
  }
}
```

And here is the generated schema for `Department.java`.

```json
{
  "type" : "object",
  "id" : "urn:jsonschema:io:codejournal:maven:java2jsonschema:Department",
  "properties" : {
    "id" : {
      "type" : "integer"
    },
    "name" : {
      "type" : "string"
    }
  }
}
```


The [working code][6] is available on github for reference.


  [1]: https://json-schema.org/
  [2]: https://www.json.org/json-en.html
  [3]: /2020/12/generating-code-using-maven-java-to-xsd/
  [4]: https://github.com/the-code-journal/maven-for-beginners/tree/main/005-generate-code-java-to-json-schema/initial
  [5]: https://github.com/gravitee-io/json-schema-generator-maven-plugin
  [6]: https://github.com/the-code-journal/maven-for-beginners/tree/main/005-generate-code-java-to-json-schema/final
