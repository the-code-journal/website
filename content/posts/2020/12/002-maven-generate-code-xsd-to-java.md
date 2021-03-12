---
date: 2020-12-29
linktitle: Generating Code using Maven - XSD to Java
next: /2021/01/generating-code-using-maven-java-to-json-schema/
prev: /2020/11/generating-code-using-maven-java-to-xsd/
title: Generating Code using Maven - XSD to Java
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this tutorial, we will talk about how Maven can be used to generate Java classes from XML Schema definitions or XSDs. While dealing with XML data, this can really help to cut down your development effort to map the XML structure to Java classes, and you can also avoid a lot of errors during XML processing, by using information from the XSD.

Below is the video which demos generating Java classes from XSD.

{{< yt "https://www.youtube.com/embed/bBitPsMFyJM" >}}





## XSD to Java

In the [previous article](/2020/11/generating-code-using-maven-java-to-xsd/), we saw how we can generate the XSD schema file from Java classes that we can give to our clients or consumers, so that they can know what is the structure of XML data they will be processing.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2020/12/002-maven-generating-code-xsd-to-java/maven-xsd-to-java.png" alt="Maven Java to XSD" />
</div>
{{< /rawhtml >}}

Now, let’s look at the right side of this setup where your application is the consumer of the XML data, and how you can use the XSD schema to generate the Java classes which represent the XML data.





{{< articlead >}}

## Project

The initial project used in this article is [available here][1].

Here are the contents of the Project

{{< rawhtml >}}
<pre class="code-output">
-> tree .
.
├── pom.xml
├── schema1.xsd
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
</pre>
{{< /rawhtml >}}

We have `pom.xml` exists with bare minimum configuration below. It has project coordinates, and Java compiler properties.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>io.codejournal.maven</groupId>
    <artifactId>maven-generate-code-xsd-to-java</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>
</project>
```

Another file is the `schema1.xsd` that we created in the [previous article](/2020/11/generating-code-using-maven-java-to-xsd/). Here are the contents of the XSD file.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" version="1.0">

  <xs:complexType name="department">
    <xs:sequence>
      <xs:element name="id" type="xs:int" />
      <xs:element name="name" type="xs:string" minOccurs="0" />
    </xs:sequence>
  </xs:complexType>

  <xs:complexType name="employee">
    <xs:sequence>
      <xs:element name="id" type="xs:int" />
      <xs:element name="name" type="xs:string" minOccurs="0" />
      <xs:element name="dept" type="department" minOccurs="0" />
    </xs:sequence>
  </xs:complexType>
</xs:schema>
```





{{< articlead >}}

## Maven Jaxb Plugin

We will use [jaxb2-maven-plugin][2] and it has following goals.

This plugin has following goals.
- `jaxb2:schemagen` - Mojo that creates XML schema(s) from compile-scope Java sources or binaries by invoking the JAXB SchemaGenerator.
- `jaxb2:testSchemagen` - Mojo that creates XML schema(s) from test-scope Java testSources or binaries by invoking the JAXB SchemaGenerator.
- `jaxb2:testXjc` - Mojo that creates test-scope Java source or binaries from XML schema(s) by invoking the JAXB XJC binding compiler.
- `jaxb2:xjc` - Mojo that creates compile-scope Java source or binaries from XML schema(s) by invoking the JAXB XJC binding compiler.

For our use-case, XSD to Java, we will need [`xjc`][3] goal. Let's start configuring the plugin in `pom.xml`.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>jaxb2-maven-plugin</artifactId>
            <version>2.5.0</version>
            <executions>
                <execution>
                    <id>xsd-to-java</id>
                    <goals>
                        <goal>xjc</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <packageName>io.codejournal.maven.xsd2java</packageName>
            </configuration>
        </plugin>
    </plugins>
</build>
```

So, we configured the goal `xjc`, and in `configuration`, we specified the package where we want the Java files to be generated.

Now, if your Maven project is configured for Java 10 or below, then you wont need anything. That's because JAXB was part of JDK and it was [deprecated in Java 9][3]. But, since Java 11, JAXB classes are [removed from JDK 11][4].

Hence, for Java 11, we will need to add the JAXB dependencies - [API][5] and [an implementation][6]. These dependencies are below.

```xml
<dependencies>
	<dependency>
		<groupId>javax.xml.bind</groupId>
		<artifactId>jaxb-api</artifactId>
		<version>2.3.1</version>
	</dependency>
	<dependency>
		<groupId>com.sun.xml.bind</groupId>
		<artifactId>jaxb-impl</artifactId>
		<version>3.0.0</version>
	</dependency>
</dependencies>
```





{{< articlead >}}

## Building

Before we run the build, the last thing we need to do is placing the `schema1.xsd` in the right folder. The plugin expects the schema files to be placed inside `src/main/xsd` directory.

So, let's create a new folder, `xsd` inside `src/main` and move the `schema1.xsd` inside it.

```
[codejournal@codejournal-box ~/workspace/maven-generate-code-xsd-to-java]
-> mkdir src/main/xsd

[codejournal@codejournal-box ~/workspace/maven-generate-code-xsd-to-java]
-> mv schema1.xsd src/main/xsd/

[codejournal@codejournal-box ~/workspace/maven-generate-code-xsd-to-java]
-> tree src/main/
src/main/
├── java
├── resources
└── xsd
    └── schema1.xsd
```

And that should do it. When you run `mvn clean compile`, maven will run the `xjc` goal during the `generate-sources` phase and generate the Java classes for the schema.

Here is a sample run.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean compile
[INFO] Scanning for projects...
[INFO] 
[INFO] --------< io.codejournal.maven:maven-generate-code-xsd-to-java >--------
[INFO] Building maven-generate-code-xsd-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-xsd-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-xsd-to-java/final/target
[INFO] 
[INFO] --- jaxb2-maven-plugin:2.5.0:xjc (xsd-to-java) @ maven-generate-code-xsd-to-java ---
[INFO] Created EpisodePath [/home/codejournal/workspace/maven-generate-code-xsd-to-java/final/target/generated-sources/jaxb/META-INF/JAXB]: true
[INFO] Ignored given or default xjbSources [/home/codejournal/workspace/maven-generate-code-xsd-to-java/final/src/main/xjb], since it is not an existent file or directory.
[INFO] Created EpisodePath [/home/codejournal/workspace/maven-generate-code-xsd-to-java/final/target/generated-sources/jaxb/META-INF/JAXB]: true
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-xsd-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 1 resource
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-xsd-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 4 source files to /home/codejournal/workspace/maven-generate-code-xsd-to-java/final/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.489 s
[INFO] Finished at: 2020-12-29T19:06:54+01:00
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

Let's look inside the `target/generated-sources/jaxb` and you will see the Java files in there.

{{< rawhtml >}}
<pre class="code-output">
-> tree target/generated-sources/jaxb/
target/generated-sources/jaxb/
├── io
│   └── codejournal
│       └── maven
│           └── xsd2java
│               ├── Department.java     &lt;--------
│               ├── Employee.java       &lt;--------
│               └── ObjectFactory.java  &lt;--------
└── META-INF
    └── JAXB
        └── episode_xsd-to-java.xjb
</pre>
{{< /rawhtml >}}

The contents of the `Employee.java` and `Department.java` are below.

{{< rawhtml >}}
<pre class="code-output">
+----------------------------------------------------------------+--------------------------------------------------------------+
|                      Employee.java                             |                        Department.java                       |
+----------------------------------------------------------------+--------------------------------------------------------------+
package io.codejournal.maven.xsd2java;                           |  package io.codejournal.maven.xsd2java;
                                                                 |  
import javax.xml.bind.annotation.XmlAccessType;                  |  import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;                |  import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlType;                        |  import javax.xml.bind.annotation.XmlType;
                                                                 |  
@XmlAccessorType(XmlAccessType.FIELD)                            |  @XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "employee", propOrder = {                        |  @XmlType(name = "department", propOrder = {
    "id",                                                        |      "id",
    "name",                                                      |      "name"
    "dept"                                                       |  })
})                                                               |  public class Department {
public class Employee {                                          |  
                                                                 |      protected int id;
    protected int id;                                            |      protected String name;
    protected String name;                                       |  
    protected Department dept;                                   |      /**
                                                                 |       * Gets the value of the id property.
    /**                                                          |       * 
     * Gets the value of the id property.                        |       */
     *                                                           |      public int getId() {
     */                                                          |          return id;
    public int getId() {                                         |      }
        return id;                                               |  
    }                                                            |      /**
                                                                 |       * Sets the value of the id property.
    /**                                                          |       * 
     * Sets the value of the id property.                        |       */
     *                                                           |      public void setId(int value) {
     */                                                          |          this.id = value;
    public void setId(int value) {                               |      }
        this.id = value;                                         |  
    }                                                            |      /**
                                                                 |       * Gets the value of the name property.
    /**                                                          |       * 
     * Gets the value of the name property.                      |       * @return
     *                                                           |       *     possible object is
     * @return                                                   |       *     {@link String }
     *     possible object is                                    |       *     
     *     {@link String }                                       |       */
     *                                                           |      public String getName() {
     */                                                          |          return name;
    public String getName() {                                    |      }
        return name;                                             |  
    }                                                            |      /**
                                                                 |       * Sets the value of the name property.
    /**                                                          |       * 
     * Sets the value of the name property.                      |       * @param value
     *                                                           |       *     allowed object is
     * @param value                                              |       *     {@link String }
     *     allowed object is                                     |       *     
     *     {@link String }                                       |       */
     *                                                           |      public void setName(String value) {
     */                                                          |          this.name = value;
    public void setName(String value) {                          |      }
        this.name = value;                                       |  }
    }                                                            |  
                                                                 |  
    /**                                                          |  
     * Gets the value of the dept property.                      |  
     *                                                           |  
     * @return                                                   |  
     *     possible object is                                    |  
     *     {@link Department }                                   |  
     *                                                           |  
     */                                                          |  
    public Department getDept() {                                |  
        return dept;                                             |  
    }                                                            |  
                                                                 |  
    /**                                                          |  
     * Sets the value of the dept property.                      |  
     *                                                           |  
     * @param value                                              |  
     *     allowed object is                                     |  
     *     {@link Department }                                   |  
     *                                                           |  
     */                                                          |  
    public void setDept(Department value) {                      |  
        this.dept = value;                                       |  
    }                                                            |  
}
</pre>
{{< /rawhtml >}}

There is another class `ObjectFactory.java` which is the Factory class to allow you to create `Employee` and `Department` objects easily.

{{< rawhtml >}}
<pre class="code-output">
package io.codejournal.maven.xsd2java;

import javax.xml.bind.annotation.XmlRegistry;

@XmlRegistry
public class ObjectFactory {


    /**
     * Create a new ObjectFactory that can be used to create new instances
     * of schema derived classes for package: io.codejournal.maven.xsd2java
     * 
     */
    public ObjectFactory() {
    }

    /**
     * Create an instance of {@link Department }
     * 
     */
    public Department createDepartment() {
        return new Department();
    }

    /**
     * Create an instance of {@link Employee }
     * 
     */
    public Employee createEmployee() {
        return new Employee();
    }
}
</pre>
{{< /rawhtml >}}

These Java classes are available for you to use in your code. Any changes to the `schema1.xsd` will automatically be reflected in the Java classes.

Here is a sample `Runner.java` that uses these classes generated by Maven.

```java
package io.codejournal.maven.xsd2java;

public class Runner {

    public static void main(final String[] args) {

        final ObjectFactory factory = new ObjectFactory();

        final Department dept = factory.createDepartment();
        dept.setId(1);
        dept.setName("Finance");

        final Employee emp = factory.createEmployee();
        emp.setId(1001);
        emp.setName("John Doe");
        emp.setDept(dept);

        System.out.println(emp);
    }
}
```

The [working code][7] is available on github for reference.


 [1]: https://github.com/the-code-journal/maven-for-beginners/raw/main/004-generate-code-xsd-to-java/maven-generate-code-xsd-to-java.zip
 [2]: https://github.com/mojohaus/jaxb2-maven-plugin
 [3]: https://docs.oracle.com/javase/9/docs/api/deprecated-list.html
 [4]: https://www.oracle.com/java/technologies/javase/jdk-11-relnote.html#JDK-8190378
 [5]: https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api
 [6]: https://mvnrepository.com/artifact/com.sun.xml.bind/jaxb-impl
 [7]: https://github.com/the-code-journal/maven-for-beginners/raw/main/004-generate-code-xsd-to-java/final
