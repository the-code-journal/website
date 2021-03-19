---
date: 2021-01-14
linktitle: Generating Code using Maven - JSON Schema to Java
next: /2021/02/generating-code-using-maven-soap-wsdl-to-java/
prev: /2021/01/generating-code-using-maven-java-to-json-schema/
title: Generating Code using Maven - JSON Schema to Java
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this tutorial, we will talk about how Maven can be used to generate Java classes from [JSON Schema][1]. This really helps you to get started with processing of JSON data, as your Java classes gets mapped the JSON structure automatically and you don’t have to invest time doing that.

Below is the video which demos generating Java classes from JSON Schema.

{{< yt "https://www.youtube.com/embed/4oUJL1U1zac" >}}





## JSON Schema to Java

In the [previous article][2], we saw the left side of the processing pipeline - generating JSON schema from Java classes.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/01/maven-json-schema-to-java.png" alt="Maven JSON Schema to Java" />
</div>
{{< /rawhtml >}}

In this tutorial, we will take the JSON schema files created in the [previous article][2], and generate the Java files for them.





{{< articlead >}}

## Project

The initial project used in this article is [available here][3].

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
        │               └── jsonschema2java
        └── resources
            └── schemas
                ├── department.json
                └── employee.json
</pre>
{{< /rawhtml >}}

We have `pom.xml` exists with bare minimum configuration below. It has project coordinates, and Java compiler properties.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>io.codejournal.maven</groupId>
    <artifactId>maven-generate-code-json-schema-to-java</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>
</project>
```

We have the two JSON schema files - `employee.json` and `department.json`, that we generated in the [previous article][2].

```json
{
  "_comment": "employee.json",
  "type": "object",
  "id": "urn:jsonschema:io:codejournal:maven:java2jsonschema:Employee",
  "properties": {
    "id": {
      "type": "integer"
    },
    "name": {
      "type": "string"
    }
  }
}
```
```json
{
  "_comment": "departmet.json",
  "type": "object",
  "id": "urn:jsonschema:io:codejournal:maven:java2jsonschema:Department",
  "properties": {
    "id": {
      "type": "integer"
    },
    "name": {
      "type": "string"
    }
  }
}
```





{{< articlead >}}

## javaschema2pojo plugin

We will be using [javaschema2pojo][4] to generate our Java classes from JSON. There is an [online version][5] also for this tool, where you can generate the Java class online itself, and there are lots of configuration options to fiddle around with.

The plugin has a single goal - `generate` that is attached to the `generate-sources` phase by default. Here is the [documentation][4] for this plugin.

Let's start configuring the plugin in `pom.xml`.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jsonschema2pojo</groupId>
            <artifactId>jsonschema2pojo-maven-plugin</artifactId>
            <version>1.0.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <sourceDirectory>${basedir}/src/main/resources/schemas</sourceDirectory>
                <targetPackage>io.codejournal.maven.jsonschema2json</targetPackage>
            </configuration>
        </plugin>
    </plugins>
</build>
```

So, we configured the goal `generate`, which is by default attached to `generate-sources` phase.

To specify where our schema files exists, we provided the path via `<sourceDirectory>`, and the package where the class files need to be generated, that is specified using `<targetPackage>`.

And that is all you need to get this working.





{{< articlead >}}

## Building

When you run `mvn clean generate-sources`, maven will run the `generate` goal during the `generate-sources` phase and generate the Java classes for the schema.

Here is a sample run.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean generate-sources
[INFO] Scanning for projects...
[INFO] 
[INFO] ----< io.codejournal.maven:maven-generate-code-json-schema-to-java >----
[INFO] Building maven-generate-code-json-schema-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-json-schema-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-json-schema-to-java/target
[INFO] 
[INFO] --- jsonschema2pojo-maven-plugin:1.0.2:generate (default) @ maven-generate-code-json-schema-to-java ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.524 s
[INFO] Finished at: 2021-01-15T01:22:17+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

Let's look inside the `target/generated-sources/jsonschema2pojo` and you will see the Java files in there.

{{< rawhtml >}}
<pre class="code-output">
-> tree target/generated-sources/jsonschema2pojo
target/generated-sources/jsonschema2pojo
└── io
    └── codejournal
        └── maven
            └── jsonschema2json
                ├── Department.java
                └── Employee.java

4 directories, 2 files
</pre>
{{< /rawhtml >}}

{{< articlead >}}

Let's loook at the contents of the `Employee.java`, as an example.

{{< rawhtml >}}
<pre class="code-output">
package io.codejournal.maven.jsonschema2json;

import java.util.HashMap;
import java.util.Map;
import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({
    "id",
    "name"
})
public class Employee {

    @JsonProperty("id")
    private Integer id;
    @JsonProperty("name")
    private String name;
    @JsonIgnore
    private Map<String, Object> additionalProperties = new HashMap<String, Object>();

    @JsonProperty("id")
    public Integer getId() {
        return id;
    }

    @JsonProperty("id")
    public void setId(Integer id) {
        this.id = id;
    }

    @JsonProperty("name")
    public String getName() {
        return name;
    }

    @JsonProperty("name")
    public void setName(String name) {
        this.name = name;
    }

    @JsonAnyGetter
    public Map<String, Object> getAdditionalProperties() {
        return this.additionalProperties;
    }

    @JsonAnySetter
    public void setAdditionalProperty(String name, Object value) {
        this.additionalProperties.put(name, value);
    }

    @Override
    public String toString() {
        // Generated toString
    }

    @Override
    public int hashCode() {
        // Generated hashCode
    }

    @Override
    public boolean equals(Object other) {
        // Generated equals
    }
}
</pre>
{{< /rawhtml >}}

You will notice that, there are annotations(`@JsonProperty, @JsonInclude` etc.). These are from [Jackson library][7] and to make sure the compilation of these classes works, we need to include the Jackson library in our `pom.xml`.

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.12.0</version>
    </dependency>
</dependencies>
```

And now when you run `mvn clean compile`, the build should be successfully compiling these classes.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean compile
[INFO] Scanning for projects...
[INFO] 
[INFO] ----< io.codejournal.maven:maven-generate-code-json-schema-to-java >----
[INFO] Building maven-generate-code-json-schema-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-json-schema-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-json-schema-to-java/target
[INFO] 
[INFO] --- jsonschema2pojo-maven-plugin:1.0.2:generate (default) @ maven-generate-code-json-schema-to-java ---
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-json-schema-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 2 resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-json-schema-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 3 source files to /home/codejournal/workspace/maven-generate-code-json-schema-to-java/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.437 s
[INFO] Finished at: 2021-01-15T01:28:53+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

The [working code][6] is available on github for reference.


  [1]: https://json-schema.org/
  [2]: /2021/01/generating-code-using-maven-java-to-json-schema/
  [3]: https://github.com/the-code-journal/maven-for-beginners/raw/main/006-generate-code-json-schema-to-java/maven-generate-code-json-schema-to-java.zip
  [4]: https://github.com/joelittlejohn/jsonschema2pojo/wiki/Getting-Started#the-maven-plugin
  [5]: http://www.jsonschema2pojo.org/
  [6]: https://github.com/the-code-journal/maven-for-beginners/tree/main/006-generate-code-json-schema-to-java/final
  [7]: https://github.com/FasterXML/jackson
