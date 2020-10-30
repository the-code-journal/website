---
date: 2020-09-10
linktitle: Maven Dependency Scopes
next: /2020/08/install-apache-maven-on-linux/
prev: /2020/09/maven-project-structure/
title: Maven Project Object Model
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



POM is where a project's identity and structure are declared, builds are configured, and project relationships are defined. We will see what all can be configured in Maven configuration file - pom.xml, and when to use them.

Here is the video explaining these.

{{< yt "https://www.youtube.com/embed/R8K0FuHBccc" >}}





## Project Object Model - pom.xml

Maven's project object model defines the conceptual model of the project. It tells Maven what sort of project you have, and all related information to the project.

The POM is independent of any language specifics and that is why, Maven can be used to build projects for other languages like `.Net` or `C++` as well.

As long as you have a plugin that Maven can call to perform the operation, Maven can build any non-Java project. That's the beauty of Maven.

The configuration options in `pom.xml` can be categorized into 4 types.

- Configurations that define project's relation to other projects
- General Project Information
- Build Settings
- Build Environment





{{< articlead >}}

## Relationships

Relationships section groups a set of configurations to define the relation of your project with other projects. Hardly you will have a Maven project that is completely standalone.



### Relationships - Project Co-ordinates

The first set of configuration is the Maven coordinates. We have already talked about Maven coordinates and these are your - groupId, artifactId and version. This combination defines a unique identity of your project that maven need to reference your project in the sea of dependencies.

Every project needs to have these and you define these during creation of the project.



### Relationships - Modules

The next section within relationships is configuration to define a multi-module project. A multi-module project is a project where a project gets splitted into various sub-modules, but they are still part of the main project.

This way, you just need to build the main project, and the modules get build automatically. You don't even need to tell Maven which modules to build first. Maven decides that automatically.

These are configured using the `<modules>` tag in the parent pom and in the child pom uses `<parent>` tag to specify which main project they are part of.

Here is an example, wherein we are configuring 2 modules in our parent pom - `submodule1` and `submodule2`.


```xml
<!-- Parent POM -->
...
<modules>
  <module>submodule1</module>
  <module>submodule2</module>
</modules>
...


<!-- Child POM -->
...
<parent>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
</parent>
<artifactId>submodule1</artifactId>
...
```



### Relationships - inheritance

The next section is Inheritance. If your project is a multi module project, you automatically inherit the configurations from the parent pom in your submodules.

But that is not required if you don't have any submodules. You can still use the `parent` tag to import a parent pom from another location. All, you need, is to configure the `relativePath` in the parent tag where Maven will look for importing the maven configurations.

If the `relativePath` is not provided, Maven defaults to looking the `pom.xml` in parent directory, and if not found, Maven uses the co-ordinate to first search the local repository and then the remote repository.

Here is an example how to configure this.

```xml
...
<parent>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <relativePath>../parent</relativePath>
</parent>
...
```



### Relationships - dependencies

And the last section of relationships - dependencies. This section allows you to configure your project's dependencies.

The wrapping tag is `dependencies` and then every dependency is configured inside this. Each dependency is configured by its coordinate and you define the [dependency scope](/2020/08/maven-dependency-scopes/) appropriately.

And those are all the configurations related to project relationships in Maven.





{{< articlead >}}

## Project Info

Maven is mostly about build configurations, but Maven also provides a whole lot of options to provide project information.



### Project Info - General information

General Project information section gives the overall project information. The first section in this has a set of configurations to define the name of the project, description and url.

```xml
<name>Project Name</name>
<description>Description</description>
<url>https://www.project.url</url>
<inceptionYear>2020</inceptionYear>
```



### Project Info - Licenses

License section defines the License associated to the project. It marks when and how the project can be used. This is a very useful thing because projects can be published to maven central repositories and anybody using your project, will know the license implications.

You can define one or more licenses if they are applicable. Here is an example.

```xml
...
<licenses>
  <license>
    <name>The MIT License</name>
    <url>https://opensource.org/licenses/MIT</url>
    <distribution><!-- repo|manual --> </distribution>
    <comments>An OSS license</comments>
  </license>
</licenses>
...
```

One thing that is important here is the `distribution`. You have option to specify either `repo` or `manual`. A value of `repo` means that your project artifact can be downloaded from the maven's central repository. A value of `manual` signifies that project artifact has to be manually obtained and installed for usage.



### Project Info - Organization

You can also configure your organization `name` and `url` in the poml like this.

```xml
...
<organization>
  <name>Codehaus Mojo</name>
  <url>http://mojo.codehaus.org</url>
</organization>
...
```



### Project Info - Developers

`developers` section allows you to configure the core team members of the project. You can have a lot of people working on the project, but this section should list the developers who are responsible for the project, and take key decisions regarding the project.

As a general rule of thumb, if somebody shouldn't be contacted about the project, they should not be part of this list.

Here is an example, and most of the them are self explanatory with information like `id`, `name`, `email`, `roles`, user properties etc. You can look up [the documentation](https://maven.apache.org/pom.html#Developers) for more details if you are configuring this.

```xml
...
<developers>
  <developer>
    <id>jdoe</id>
    <name>John Doe</name>
    <email>jdoe@example.com</email>
    <url>http://www.example.com/jdoe</url>
    <organization>ACME</organization>
    <organizationUrl>http://www.abc.com</organizationUrl>
    <roles>
      <role>architect</role>
      <role>developer</role>
    </roles>
    <timezone>America/New_York</timezone>
    <properties>
      <picUrl>http://www.example.com/jdoe/pic</picUrl>
    </properties>
  </developer>
</developers>
...
```



### Project Info - Contributors

For anybody who is part of the project, but isn't part of the `developers` section, you can list them in `contributors` section. It is similar to `developers` section. Any healthy project will have more contributors than developers.

Here is an example.

```xml
...
<contributors>
  <contributor>
    <name>Noelle</name>
    <email>some.name@gmail.com</email>
    <url>http://noellemarie.com</url>
    <organization>Noelle Marie</organization>
    <organizationUrl>http://abc.com</organizationUrl>
    <roles>
      <role>tester</role>
    </roles>
    <timezone>America/Vancouver</timezone>
    <properties>
      <gtalk>some.name@gmail.com</gtalk>
    </properties>
  </contributor>
</contributors>
...
```

And those are all the various configurations related to General Project Information.





{{< articlead >}}

## Build Settings

We now come to `build` configurations. Since, maven uses convention over configuration, you might not need to configure most of these, but knowing what configuration options are available is always good.

Any build related configuration is placed inside the `<build>` tags, which marks the section for maven build.



### Build Settings - directories

Configurations related to directories allows you to configure the directories for your source and test files, as well as the output directories for these.

Here are the default values of these configurations and more often than not, you won't be required to configure these.

This makes your life easy by adhering to Maven's convention and getting your builds running in no time.

```xml
...
<build>
  ...
  <sourceDirectory>${basedir}/src/main/java</sourceDirectory>

  <scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory>

  <testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>

  <outputDirectory>${basedir}/target/classes</outputDirectory>

  <testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>
  ...
</build>
...
```



### Build Settings - extensions

`extensions` can be used to enable extensions to the build process. They are also used to configure wagor providers that are used for the transport of artifact between repositories.

Maven, by default uses [`HttpURLConnection`](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html) java classes to download the artifacts, but there can be situations when you want to use a different way of fetching the artifacts from repositories. You might never need this, but this is an option available for you, in case you need these.

Here is an example that configures `wagon-ftp` as your primary transport mechanism to exchange artifacts between repositories.

```xml
...
<build>
  ...
  <extensions>
    <extension>
      <groupId>org.apache.maven.wagon</groupId>
      <artifactId>wagon-ftp</artifactId>
      <version>2.10</version>
    </extension>
  </extensions>
  ...
</build>
...
```





{{< articlead >}}

### Build Settings - resources

Resources are files that your application need while running. These are not source code files, but are still important, like application properties, yamls, db configs, web xmls, etc.

Maven already looks for resource files under `src/main/resources` and `src/test/resources`, but if you have any other file locations to be configured, you can use the `resources` section to configure these.

Here is an example.

```xml
...
<build>
  ...
  <resources>
    <resource>
      <targetPath>META-INF/plexus</targetPath>
      <filtering>false</filtering>
      <directory>${basedir}/src/main/plexus</directory>
      <includes>
        <include>configuration.xml</include>
      </includes>
      <excludes>
        <exclude>**/*.properties</exclude>
      </excludes>
    </resource>
  </resources>
  <testResources>
    ...
  </testResources>
  ...
</build>
...
```

You can also configure filtering of the resources files if you need it using the `includes` and `excludes` that allow you to include specific set of files or exclude a specific set of files during the build process.



### Build Settings - plugins

And then we have the `plugins` section that allows you to configure the plugins to be used during the build.

Maven already provides a default plugins for its build lifecycle, but if you want to have more control on them, you can configure them in the `plugins` sections. Here is an example.

```xml
...
<build>
  ...
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <version>2.6</version>
      <extensions>false</extensions>
      <inherited>true</inherited>
      <configuration>
        <classifier>test</classifier>
      </configuration>
      <dependencies>...</dependencies>
      <executions>...</executions>
    </plugin>
  </plugins>
  ...
</build>
...
```



### Build Settings - reporting

And final section in build settings is reporting. This section contains elements that correspond specifically for the site generation phase. For example, you can have configuration to generate Javadocs for your application, in this section.

```xml
...
<reporting>
  <outputDirectory>
    ${basedir}/target/site
  </outputDirectory>
  <plugins>
    <plugin>
      <artifactId>maven-project-info-reports-plugin</artifactId>
      <version>2.0.1</version>
      <reportSets>
        <reportSet>
          ...
        </reportSet>
      </reportSets>
    </plugin>
  </plugins>
</reporting>
...
```





{{< articlead >}}

## Build Environment

And final category of configurations is Environment configurations. These configurations allow you to control maven build execution environment.

For example, if you are building for a development environment, you might be deploying to development server, where as if you are building for production, you might be deploying to production. The build settings for these, will be different, and this is the section where you will need to configure this behaviour.

Let’s look what configurations we have here, and we start with environment related configurations.



### Build Environment - issueManagement

The first set of configuration under environment is Issue Management. It defines the bug tracking system, for example, BugZilla, or JIRA. Here is the example.

```xml
...
<issueManagement>
  <system>Bugzilla</system>
  <url>http://127.0.0.1/bugzilla/</url>
</issueManagement>
...
```

Primarily, without any additional configuration, this information is used for project documentation, but there is nothing stopping you from using this information for other purposes as well.

For example, you could have a maven plugin that could use this to automatically close the bugs/issues once the build is deployed to production.



### Build Environment - ciManagement

CI management or Continuous Integration management section allows to integrate with the Continuous Integration system to trigger notifications related to the build status.

So, if your build is successful or fails during the continuous integration build, appropriate notifications can be sent to team or developers.

Here is an example.

```xml
...
<ciManagement>
  <system>continuum</system>
  <url>http://127.0.0.1:8080/continuum</url>
  <notifiers>
    <notifier>
      <type>mail</type>
      <sendOnError>true</sendOnError>
      <sendOnFailure>true</sendOnFailure>
      <sendOnSuccess>false</sendOnSuccess>
      <sendOnWarning>false</sendOnWarning>
      <configuration>
        <address>continuum@127.0.0.1</address>
      </configuration>
    </notifier>
  </notifiers>
</ciManagement>
...
```



### Build Environment - mailingLists

If your project uses mailing lists, then you also get a section to configure your mailing list settings. Again, this information is primarily used for documentation, but it can be used for other purposes as well.

Here is an example that configures a mailing list with its address where it can be subscribed to and also be unsubscribed from, and also, address to use while posting to the mailing list.

There is a config for archive url where old topics can be archived and topics can also be mirrored using otherArchives section.

```xml
...
<mailingLists>
  <mailingList>
    <name>User List</name>
    <subscribe>subscribe@127.0.0.1</subscribe>
    <unsubscribe>unsubscribe@127.0.0.1</unsubscribe>
    <post>user@127.0.0.1</post>
    <archive>http://127.0.0.1/user/</archive>
    <otherArchives>
      <otherArchive>http://example.com</otherArchive>
    </otherArchives>
  </mailingList>
</mailingLists>
...
```



### Build Environment - scm

SCM or Source Code Magement section allows you to configure the source code system. Here is an example.

```xml
...
<scm>
  <connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
  <developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
  <tag>HEAD</tag>
  <url>http://127.0.0.1/websvn/my-project</url>
</scm>
...
```

The `connection` value specifies the read only connection to SCM system, while the `developerConnection` specifies the write access connection to SCM system. `Tag` specifies the tag under which this project exists. And the `url` specifies the public url of the SCM system, where you can publicly view the files of the project.





{{< articlead >}}

### Build Environment - prerequisites

`prerequisites` section allows you to configure certain conditions so that if they are not met, the build will fail.

For example, if you have a specific feature from maven used in the project that is not available in earlier versions, you can configure the maven version as a pre-requisite.

```xml
...
<prerequisites>
  <maven>3.0.6</maven>
</prerequisites>
...
```

That way, if somebody has a lower version of maven and they try to build, they will get a build failure.



### Build Environment - repositories

Repositories store the Maven artifacts and whenever your project needs any dependency, it will be made available from the repository.

Every artifact is first looked up in your local repository in your local system, and when it is not available there, Maven goes out to the internet and downloads it. But from where does it download the artifact, that is exactly configured in this section.

Here is an example.

```xml
...
<repositories>
  <repository>
    <name>Nexus Snapshots</name>
    <id>snapshots-repo</id>
    <url>https://oss.sonatype.org/snapshots</url>
    <layout>default</layout>
    <releases>
        <enabled>false</enabled>
        <updatePolicy>always</updatePolicy>
        <checksumPolicy>warn</checksumPolicy>
    </releases>
    <snapshots>
      ...
    </snapshots>
  </repository>
</repositories>
<pluginRepositories>
  ...
</pluginRepositories>
...
```

You have the `id`, `name` and `url` of the repository. Two particular configurations to note here are the - `releases` and `snapshots`. Releases section configures downloading of the release artifacts from the repository, while the snapshots section configures downloading of the snapshot artifacts.

Repository section is optional, and by default, Maven uses its central repository to download the artifacts. There are two cases when you might need to configure these.

1. When your dependency artifact is not available in Maven’s central repository. That’s obvious, of course.
2. You might have a central repository available in your organization. This will store artifacts from your organization that are not publicly available, and also since its available in your organization, the download speeds are much faster compared to Maven’s central repository.



### Build Environment - distributionManagement

Distribution Management manages the distribution of the artifact and other related files. There are multiple options here, so let’s see an example.

```xml
...
<distributionManagement>
  <repository>...</repository>
  <snapshotRepository>...</snapshotRepository>
  <site>...</site>
  <relocation>...</relocation>
  <downloadUrl>http://mojo.codehaus.org/my-project</downloadUrl>
  <status>deployed</status>
</distributionManagement>
...
```

The `repository` section configures where the artifact will be put when the project is deployed. The `snapshotRepository` section configures where the snapshot artifacts will be put when the snapshot is deployed. `site` section configures where the Project site will be deployed. `relocation` configures the details if your project is being moved to a new location, you can configure that here. `downloadUrl` is the public url from where the project artifact can be downloaded.

And `status` specifies the status of the project. Ideally, you should not be setting this as Maven will automatically set this when deploying the artifact.





{{< articlead >}}

## Build Environment - Profiles

Maven profiles are very very useful feature and it allows you to segregate your builds for various environments. Usually, your build steps are going to be different when you are running your builds locally just to test your application vs a full fledged build that deploys to production environment that is run by a Continuous Integration system.

This separation of build variations can be configured using Profiles. Here is an example.

```xml
...
<profiles>
  <profile>
    <id>test</id>
    <activation>...</activation>
    <build>...</build>
    <modules>...</modules>
    <repositories>...</repositories>
    <pluginRepositories>...</pluginRepositories>
    <dependencies>...</dependencies>
    <reporting>...</reporting>
    <dependencyManagement>...</dependencyManagement>
    <distributionManagement>...</distributionManagement>
  </profile>
</profiles>
...
```

So, you can configure a profile for your local system builds and a different profile for Continuous Integration build.

You define these using the profiles section, and this section has a subset of build settings and build environment in the configuration.

Yet again, we will be discussing Maven profile in great details in our future videos, but just understand that maven profiles is what you will need when you have to configure different build behaviours.





{{< articlead >}}

## Maven Properties

One Last section that is very important is `properties`. Properties are value placeholders, just like variables in any language. These can be referenced using the $ sign and curly braces, and within the curly braces, you have the name of the property. For, example, like this - `${property}`

All Environment variables are available in maven within `env` prefix. So, if you have to access the JAVA_HOME environment variable of a system, you would configure it in pom as `${env.JAVA_HOME}`.

You can also reference any configuration with the current pom with the `project` prefix. For example, if you have to access of the `groupId` of the pom, you can specify - `${project.groupId}`.

You can also reference the maven global configurations from the `settings.xml` with prefix `settings`. For example, `${settings.offline}` to check if the repository is offline or online.

And finally, all properties accessible by `System.getProperties()` are accessible directly by their names. For example, `${user.name}`.

You also have option to create some new ones within `properties` section.

```xml
...
<properties>
  <junit.version>4.13</junit.version>
</properties>

<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
  </dependency>
</dependencies>
...
```

In this example here, I have created a property as `junit.version` and the value is `4.13`. And then I am using this property when configuring the dependency for junit.

This is a very common use case of custom property. This allows you to have all the dependency version as properties.

And when you have to update the version, you just update the corresponding property. You won’t have to scan the entire pom to locate the dependency and update the version there

This is also very useful when defining versions in parent poms that all child poms use.





{{< articlead >}}

And that is all about Maven Project Object Model. As you can see, Maven’s POM is very exhaustive and has pretty much every config required by your project for any use case.

Obviously, you wont need all of these, and it will really depend on the size of the project you are dealing with.

A general rule is to use Maven convention all the time unless you really have to customize it. Start with minimum configuration at first and build on top of that.
