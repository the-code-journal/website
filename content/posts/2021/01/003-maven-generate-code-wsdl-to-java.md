---
date: 2021-01-28
linktitle: Generating Code using Maven - SOAP WSDL to Java
next: /2021/02/generating-code-using-maven-swagger-to-java/
prev: /2021/01/generating-code-using-maven-json-schema-to-java/
title: Generating Code using Maven - SOAP WSDL to Java
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



In this tutorial, we will talk about how Maven can be used to generate Java classes for a SOAP Web Service from a SOAP WSDL file.

Below is the video which demos this.

{{< yt "https://www.youtube.com/embed/24uE_gVNVB0" >}}





## SOAP

[SOAP][1] stands for **Simple Object Access Protocol**. It is an XML based protocol to exchange messages with a Web Service. It primarily uses HTTP for the communication to Web services, and since it is XML based, the request and response exchanged are in XML. Let’s understand this by a diagram below.

{{< rawhtml >}}
<div class="image">
    <img src="/images/2021/01/soap.png" alt="Simple Object Access Protocol (SOAP)" />
</div>
{{< /rawhtml >}}

So, you will have a client and a server, and the server has a web service configured. This web service clearly defines how the communication can be done and the rules of exchanges(like what protocol to use and is there any authentication mechanism), and the structure of request and response formats.

Now, client would make the request, which will be in the form of XML data over the protocol(HTTP in most cases), and then server processes the request, and sends back the response, which again is in XML.

It’s like any other HTTP request-response flow, just that, adhering to SOAP provides clearly defines interfaces that can be called by client, and exchange between client and server has to be in XML.

If you have an application that has web services and has been running for about 5-10 years or more, then it is possible that those web services are using SOAP.





## WSDL

[WSDL][2] or Web Services Description Language is the XML based description language for Web Services. WSDL is used to define the web service interfaces that provides information about what services are provided by the server and how can they be called. 

A typical WSDL had these sections.

- **Service** - Defines the Service itself
- **Endpoint** - Defines the Address or Connection Point for the Service
- **Binding** - specifies the interface and binding style and transport mechanism
- **Interface** - Defines the web service operation and message used to execute the operation
- **Operation** - is the actual SOAP method that gets called on the Server
- **Message** - defines the structure of the message
- **Types** - XML Schema of the message indicating which properties are of which type

Usually, the WSDL files are not written, but generated from the code. When a Web Service is configured, [JAXB][3] annotations are used to indicate the service, endpoint, operation, etc and the server returns the generated WSDL for the web service.

And then this WSDL is given to teams or clients who wish to communicate with the web service. We will be configuring this use case in this article - You already have a WSDL and now you want to make calls to the Web Service.

If you go about configuring the various classes for a WSDL yourself, then it is quite likely that you will miss something and you can spend hours and hours finding the bugs. So, never ever try to configure that.

Everywhere, this is accomplished with the help of tools and plugins. This not only saves you a lot of time, but it also reduces the errors.





{{< articlead >}}

## Project

The initial project used in this article is [available here][4].

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
   <artifactId>maven-generate-code-wsdl-to-java</artifactId>
   <version>0.0.1-SNAPSHOT</version>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <java.version>11</java.version>
      <maven.compiler.source>${java.version}</maven.compiler.source>
      <maven.compiler.target>${java.version}</maven.compiler.target>
   </properties>
</project>
```

Now, we need a WSDL that we can import in our project and generate classes for it. We will be using [NumberConversion web service][5]. This service has two methods as described below:

1. **NumberToWords** - Returns the word corresponding to the positive number passed as parameter. Limited to quadrillions.
2. **NumberToDollars** - Returns the non-zero dollar amount of the passed number.

The actual wsdl file can be [obtained from here][6]. You can download this file and copy it at `src/main/resources/dataaccess-numberconversion.wsdl`.

{{< rawhtml >}}
<pre class="code-output">
-> curl https://www.dataaccess.com/webservicesserver/NumberConversion.wso?WSDL \
        -o src/main/resources/dataaccess-numberconversion.wsdl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4645  100  4645    0     0   1436      0  0:00:03  0:00:03 --:--:--  1435
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Maven CXF Plugin

[Apache CXF][7] came into existence when two open-source projects - Celtix and XFire were merged back in 2006-2007, hence the name CXF. It is dedicated project for providing tooling for Web Services, but majorly people use it for XML based SOAP Web Services.

CXF includes [apache-cxf-plugin][8] which can generate java artifacts from [WSDL][2]. The plugin provides `wsdl2java` goal that is used to generated java classes from wsdl.

Let's start configuring the plugin in `pom.xml`.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-codegen-plugin</artifactId>
      <version>3.4.2</version>
      <executions>
        <execution>
          <id>generate-sources</id>
          <phase>generate-sources</phase>
          <goals>
            <goal>wsdl2java</goal>
          </goals>
          <configuration>
            <wsdlOptions>
              <wsdlOption>
                <wsdl>${basedir}/src/main/resources/dataaccess-numberconversion.wsdl</wsdl>
                <packagenames>
                  <packagename>io.codejournal.maven.wsdl2java</packagename>
                </packagenames>
              </wsdlOption>
            </wsdlOptions>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Starting off, we configure the plugin co-ordinate, execution section which maps the phase(`generate-sources`) with goal(`wsdl2java`).

With the configuration, we specify the location of wsdl with `<wsdl>` and we define the package in which the classes are generated in `<packagenames>` section.

Now, if you run `mvn clean generate-sources`, you will get the Java classes generated out of the WSDL.

{{< rawhtml >}}
<pre class="code-output">
-> mvn clean generate-resources
[INFO] Scanning for projects...
[INFO] 
[INFO] -------< io.codejournal.maven:maven-generate-code-wsdl-to-java >--------
[INFO] Building maven-generate-code-wsdl-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-wsdl-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-wsdl-to-java/target
[INFO] 
[INFO] --- cxf-codegen-plugin:3.4.2:wsdl2java (generate-sources) @ maven-generate-code-wsdl-to-java ---
[INFO] Running code generation in fork mode...
[INFO] The java executable is /usr/lib/jvm/java-11-openjdk-amd64/bin/java
[INFO] Building jar: /tmp/cxf-tmp-5499391244447620404/cxf-codegen2918941894437827096.jar
[WARNING] SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
[WARNING] SLF4J: Defaulting to no-operation (NOP) logger implementation
[WARNING] SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.325 s
[INFO] Finished at: 2021-01-27T16:45:58+05:30
[INFO] ------------------------------------------------------------------------


-> tree target/generated-sources/cxf/
target/generated-sources/cxf/
└── io
    └── codejournal
        └── maven
            └── wsdl2java
                ├── NumberConversion.java
                ├── NumberConversionSoapType.java
                ├── NumberToDollars.java
                ├── NumberToDollarsResponse.java
                ├── NumberToWords.java
                ├── NumberToWordsResponse.java
                ├── ObjectFactory.java
                └── package-info.java
</pre>
{{< /rawhtml >}}

{{< rawhtml >}}
<div class="notification">
    If you are building this project with Java 10 and below, you don't need to import any other dependencies and compilation should work. However, Java 11 onwards, you need additional dependency, as configured below.
</div>
{{< /rawhtml >}}





{{< articlead >}}

## Java-EE to Jakarta-EE

[Java-EE][8] has been a prominent framework in developing enterprise applications. However, Oracle decided to move Java-EE to open source foundation, and Eclipse put up its hand to continue supporting it. Under the Eclipse foundation umbrella, this will be known as [Jakarta-EE][9]. Thus, along with other changes, this makes the `javax.xml.*, javax.jws.*` packages to be set to phase out and replaced with `jakarta.xml.*, jakarta.jws.*` packages. Here is the [transition notes][10].

With [Java 11][11], the packages `javax.xml.*, javax.jws.*` were removed.

With Apache CXF plugin - 3.4.x version, code generated still uses `javax.xml.*` and `javax.jws.*` packages. This will change with Apache CXF plugin version 3.5+, and this is [the change on cxf side][12] to support [jakarta-ee][9] apis.

So, with Java 11, we need to additionally import [`jaxws-rt`][14] dependency. We are NOT using the latest 3.0.0(at the time of writing), since it has `jakarta.*` package structure. So, we use the next latest one - 2.3.3, which still uses `javax.*` package structure.

```xml
<dependencies>
  <dependency>
    <groupId>com.sun.xml.ws</groupId>
    <artifactId>jaxws-rt</artifactId>
    <version>2.3.3</version>
  </dependency>
</dependencies>
```

And now if you compile your project now, it should be compiling.


{{< rawhtml >}}
<pre class="code-output">
-> mvn clean compile
[INFO] Scanning for projects...
[INFO] 
[INFO] -------< io.codejournal.maven:maven-generate-code-wsdl-to-java >--------
[INFO] Building maven-generate-code-wsdl-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-wsdl-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-wsdl-to-java/target
[INFO] 
[INFO] --- cxf-codegen-plugin:3.4.2:wsdl2java (generate-sources) @ maven-generate-code-wsdl-to-java ---
[INFO] Running code generation in fork mode...
[INFO] The java executable is /usr/lib/jvm/java-11-openjdk-amd64/bin/java
[INFO] Building jar: /tmp/cxf-tmp-15185450929649547269/cxf-codegen2117855940532136684.jar
[WARNING] SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
[WARNING] SLF4J: Defaulting to no-operation (NOP) logger implementation
[WARNING] SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-wsdl-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-wsdl-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 9 source files to /home/codejournal/workspace/maven-generate-code-wsdl-to-java/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.732 s
[INFO] Finished at: 2021-01-27T16:58:17+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}





{{< articlead >}}

## Making SOAP Calls

Now, that our classes are generating and the build is compiling, let's make calls to the service itself.

Let's understand the classes that are generated.

- `NumberConversion.java` - Service class which abstracts away how calls are made to the service
- `NumberConversionSoapType.java` - Interface for defines the SOAP service methods
- `NumberToDollars.java` - Request class for numberToDollars operation
- `NumberToDollarsResponse.java` - Response class for numberToDollars operation
- `NumberToWords.java` - Request class for numberToWords operation
- `NumberToWordsResponse.java` - Response class for numberToWords response
- `ObjectFactory.java` - Object Factory class to create request and response classes.

With these in mind, here is the code for Runner.java that makes call for numberToWords operation.

```java
package io.codejournal.maven.wsdl2java;

import static java.lang.String.format;

import java.math.BigDecimal;
import java.math.BigInteger;
import java.net.URI;
import java.net.URL;

public class Runner {

    public static void main(final String[] args) throws Exception {

        final String endpoint = "https://www.dataaccess.com/webservicesserver/NumberConversion.wso";

        final URL url = URI.create(endpoint).toURL();

        final NumberConversion service = new NumberConversion(url);

        final NumberConversionSoapType soap = service.getNumberConversionSoap();

        final BigInteger number1 = BigInteger.valueOf(111);
        final BigDecimal number2 = BigDecimal.valueOf(15.99);

        final String words = soap.numberToWords(number1);
        final String dollars = soap.numberToDollars(number2);

        System.out.println(format("\n%s => %s", number1, words));
        System.out.println(format("%s => %s\n", number2, dollars));
    }
}
```

Looking at the code above, you can see that you don't have to worry about any code that makes call and parse the response. All those details are safely hidden away and also these are generated by the CXF plugin which is open-source. So, any bug in the implementation, will be reported, fixed and reviewed by the community.

Let's run to see the SOAP call working and we will have the output below

```
111 => one hundred and eleven
15.99 => fifteen dollars and ninety nine cents
```

Here is the full output from maven build.

{{< rawhtml >}}
<pre class="code-output">

-> mvn clean compile exec:java -Dexec.mainClass="io.codejournal.maven.wsdl2java.Runner"

[INFO] Scanning for projects...
[INFO] 
[INFO] -------< io.codejournal.maven:maven-generate-code-wsdl-to-java >--------
[INFO] Building maven-generate-code-wsdl-to-java 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-generate-code-wsdl-to-java ---
[INFO] Deleting /home/codejournal/workspace/maven-generate-code-wsdl-to-java/target
[INFO] 
[INFO] --- cxf-codegen-plugin:3.4.2:wsdl2java (generate-sources) @ maven-generate-code-wsdl-to-java ---
[INFO] Running code generation in fork mode...
[INFO] The java executable is /usr/lib/jvm/java-11-openjdk-amd64/bin/java
[INFO] Building jar: /tmp/cxf-tmp-4414518037936577194/cxf-codegen10655173327932969832.jar
[WARNING] SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
[WARNING] SLF4J: Defaulting to no-operation (NOP) logger implementation
[WARNING] SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-generate-code-wsdl-to-java ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven-generate-code-wsdl-to-java ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 9 source files to /home/codejournal/workspace/maven-generate-code-wsdl-to-java/target/classes
[INFO] 
[INFO] --- exec-maven-plugin:3.0.0:java (default-cli) @ maven-generate-code-wsdl-to-java ---

111 => one hundred and eleven
15.99 => fifteen dollars and ninety nine cents

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.424 s
[INFO] Finished at: 2021-03-27T17:03:46+05:30
[INFO] ------------------------------------------------------------------------
</pre>
{{< /rawhtml >}}

The [working code][13] is available on github for reference.


  [1]: https://en.wikipedia.org/wiki/SOAP
  [2]: https://en.wikipedia.org/wiki/Web_Services_Description_Language
  [3]: https://javaee.github.io/jaxb-v2/
  [4]: https://github.com/the-code-journal/maven-for-beginners/raw/main/007-generate-code-wsdl-to-java/maven-generate-code-wsdl-to-java.zip
  [5]: https://www.dataaccess.com/webservicesserver/NumberConversion.wso
  [6]: https://www.dataaccess.com/webservicesserver/NumberConversion.wso?WSDL
  [7]: https://cxf.apache.org/index.html
  [7]: https://cxf.apache.org/docs/maven-cxf-codegen-plugin-wsdl-to-java.html
  [8]: https://javaee.github.io/
  [9]: https://jakarta.ee/
 [10]: https://blogs.oracle.com/javamagazine/transition-from-java-ee-to-jakarta-ee
 [11]: https://www.oracle.com/java/technologies/javase/11-relnote-issues.html#JDK-8190378
 [12]: https://issues.apache.org/jira/browse/CXF-8371
 [13]: https://github.com/the-code-journal/maven-for-beginners/tree/main/007-generate-code-wsdl-to-java/final
 [14]: https://mvnrepository.com/artifact/com.sun.xml.ws/jaxws-rt
