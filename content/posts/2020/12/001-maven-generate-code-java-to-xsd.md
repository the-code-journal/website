---
date: 2020-12-28
linktitle: Generating Code using Maven - Java to XSD
next: /2020/12/generating-code-using-maven-xsd-to-java/
prev: /2020/11/maven-lifecycle-phase-mapping-to-plugin-goals/
title: Generating Code using Maven - Java to XSD
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this tutorial, we will talk about how Maven can be used to generate code files, specifically XML Schema definitions or XSD in short. We will be using Maven Jaxb plugin for this, and we will see how Java class files can be mapped to these XML schema files which can be really useful while dealing with XML files.

Below is the video which demos generating XML Schema(XSD) from Java classes.

{{< yt "https://www.youtube.com/embed/lNjaU92cT4g" >}}





## JAXB

[JAXB][1] stands for Java Architecture for XML Binding, and it provides a framework where you can map Java objects and XML documents.

Say, for example, you have an XML file with employee data like this.

{{< rawhtml >}}
<pre class="code-output">
+---------------------------+
|                           |
| &lt;employees>               |
|   &lt;employee>              |
|     &lt;id>1001&lt;/id>         |          +-----------------+
|     &lt;name>John Doe&lt;/name> |          |  Employee.java  |
|   &lt;/employee>             | ======>  +-----------------+
|   &lt;employee>              | &lt;======  | - id            |
|     &lt;id>1002&lt;/id>         |          | - name          |
|     &lt;name>Foo Bar&lt;/name>  |          +-----------------+
|   &lt;/employee>             |
| &lt;/employees>              |
|                           |
+---------------------------+
</pre>
{{< /rawhtml >}}

Now, to be able to process this XML file, you need to have a corresponding `Employee` Java class with fields mapping to XML structure, and when you read the file by running your code, you will get two objects of the Employee class, and then you can choose what you do with those.

And in this, JAXB helps us to map the XML structure to Java classes and vice versa, thereby making our life simpler when we try to process XML files.





{{< articlead >}}

## XML Schema Definition (XSD)

Before processing any XML files, you need to know what is the structure of the XML data. And if you don’t have the Schema or XSD file, it can be really challenging to figure out the structure. XML Schema file define the structure of the XML data and additional metadata, like if an element will be present in xml all the time or not. Guessing that just from the actual XML is impossible.

Here is the Schema File for the XML file in left.

{{< rawhtml >}}
<pre class="code-output">
+---------------------------+    +-------------------------------------------------+
|            XML            |    |                      XSD                        |
+---------------------------+    +-------------------------------------------------+
|                           |    |                                                 |
| &lt;employees>               |    | &lt;xs:schema                                      |
|   &lt;employee>              |    |   xmlns:xs="http://www.w3.org/2001/XMLSchema"   |
|     &lt;id>1001&lt;/id>         |    |   version="1.0">                                |
|     &lt;name>John Doe&lt;/name> |    |  &lt;xs:complexType name="employee">               |
|   &lt;/employee>             |    |    &lt;xs:sequence>                                |
|   &lt;employee>              |    |      &lt;xs:element name="id" type="xs:int"/>      |
|     &lt;id>1002&lt;/id>         |    |      &lt;xs:element name="name" type="xs:string"/> |
|     &lt;name>Foo Bar&lt;/name>  |    |    &lt;/xs:sequence>                               |
|   &lt;/employee>             |    |  &lt;/xs:complexType>                              |
| &lt;/employees>              |    | &lt;/xs:schema>                                    |
|                           |    |                                                 |
+---------------------------+    +-------------------------------------------------+
</pre>
{{< /rawhtml >}}

Don’t worry if you don’t understand anything. On a high level, this schema defines a type as `employee`, which has two elements - `id` and `name`. `id` is of type `integer`, while `name` element is of type `string`.

As you see, this gives us more insights about the XML file and we can be sure of certain things, rather than guessing those from the XML file itself.





## Java to XSD

When would you need to generate XSD from Java classes?

When your application is generating XML data for some other consumer or client that does not have access to your code. XSD files here are a convenient way of giving information about the XML files, without sharing the code. Clients can use the XSD file to validate the XML files, and/or generate code themselves that adheres the structure of the XML files, thereby making their life easy.

Let’s see how Maven can help you here by automatically generating the latest schema definition from your Java class files. The left block in the diagram below.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2020/12/001-maven-generating-code-java-to-xsd/maven-java-to-xsd.png" alt="Maven Java to XSD" />
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Project

The initial project used in this article is [available here][2].

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
    <artifactId>maven-generate-code-java-to-xsd</artifactId>
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

To create the XSD Schema file, you need the POJO Java classes. Let's create the classic `Employee` and `Department` classes as below.

```java
package io.codejournal.maven.java2xsd;

import static java.lang.String.format;

public class Employee {

    private int id;

    private String name;

    private Department dept;

    // Getters
    // Setters
}
```

```java
package io.codejournal.maven.java2xsd;

import static java.lang.String.format;

public class Department {

    private int id;

    private String name;

    // Getters
    // Setters
}
```





{{< articlead >}}

## Maven Jaxb Plugin

To help you generate the XML Schema during build, we will use [jaxb2-maven-plugin][3].

This plugin has following goals.
- `jaxb2:schemagen` - Mojo that creates XML schema(s) from compile-scope Java sources or binaries by invoking the JAXB SchemaGenerator.
- `jaxb2:testSchemagen` - Mojo that creates XML schema(s) from test-scope Java testSources or binaries by invoking the JAXB SchemaGenerator.
- `jaxb2:testXjc` - Mojo that creates test-scope Java source or binaries from XML schema(s) by invoking the JAXB XJC binding compiler.
- `jaxb2:xjc` - Mojo that creates compile-scope Java source or binaries from XML schema(s) by invoking the JAXB XJC binding compiler.

For our use-case, Java to XSD, we will need [`schemagen`][4] goal. Let's start configuring the plugin in `pom.xml`.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>jaxb2-maven-plugin</artifactId>
            <version>2.5.0</version>
            <executions>
                <execution>
                    <id>schemagen</id>
                    <goals>
                        <goal>schemagen</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

And that should do it. When you run now, maven will run the `schemagen` goal during the `generate-resources` phase. Here is a sample run.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean compile
[INFO] Scanning for projects...
[INFO] 
[INFO] --------< io.codejournal.maven:maven-generate-code-java-to-xsd >--------
[INFO] Building maven-generate-code-java-to-xsd 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-java-to-xsd ---
[INFO] 
[INFO] --- jaxb2-maven-plugin:2.5.0:schemagen (schemagen) @ maven-generate-code-java-to-xsd ---
[INFO] Created EpisodePath [/home/codejournal/workspace/maven-generate-code-java-to-xsd/target/generated-resources/schemagen/META-INF/JAXB]: true
[INFO] Created EpisodePath [/home/codejournal/workspace/maven-generate-code-java-to-xsd/target/generated-resources/schemagen/META-INF/JAXB]: true
Note: Writing /home/codejournal/workspace/maven-generate-code-java-to-xsd/schema1.xsd
Note: Writing /home/codejournal/workspace/maven-generate-code-java-to-xsd/target/generated-resources/schemagen/META-INF/JAXB/episode_schemagen.xjb
Note: Writing /home/codejournal/workspace/maven-generate-code-java-to-xsd/target/generated-resources/schemagen/META-INF/JAXB/episode_schemagen.xjb
[INFO] XSD post-processing: Adding JavaDoc annotations in generated XSDs.
[INFO] Processing [2] java sources.
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-java-to-xsd ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-java-to-xsd ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /home/codejournal/workspace/maven-generate-code-java-to-xsd/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.092 s
[INFO] Finished at: 2021-01-06T22:53:48+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

Now, if we look inside the `target/generate-resources` directory, we will get our schema file.

{{< rawhtml >}}
<pre class="code-output">
-> cd target/generated-resources/ && tree .
.
└── schemagen
    ├── META-INF
    │   └── JAXB
    │       └── episode_schemagen.xjb
    └── schema1.xsd                       &lt;-------- Schema File
</pre>
{{< /rawhtml >}}

And the contents of the `schema1.xsd` is below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" version="1.0">
  <xs:complexType name="department">
    <xs:sequence>
      <xs:element name="id" type="xs:int"/>
      <xs:element minOccurs="0" name="name" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
  <xs:complexType name="employee">
    <xs:sequence>
      <xs:element minOccurs="0" name="dept" type="department"/>
      <xs:element name="id" type="xs:int"/>
      <xs:element minOccurs="0" name="name" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:schema>
```

The [working code][5] is available on github for reference.


 [1]: https://javaee.github.io/jaxb-v2/
 [2]: https://github.com/the-code-journal/maven-for-beginners/raw/main/003-generate-code-java-to-xsd/maven-generate-code-java-to-xsd.zip
 [3]: https://github.com/mojohaus/jaxb2-maven-plugin
 [4]: https://www.mojohaus.org/jaxb2-maven-plugin/Documentation/v2.5.0/plugin-info.html
 [5]: https://github.com/the-code-journal/maven-for-beginners/tree/main/003-generate-code-java-to-xsd/final
