---
date: 2020-11-18
linktitle: Maven Lifecycles
next: /2020/08/install-apache-maven-on-linux/
prev: /2020/09/maven-project-object-model/
title: Maven Lifecycles
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---



[Maven Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) is the core of how Maven build the projects. If you are building a project, you are essentially running a Maven lifecycle. In this article, we will understand the lifecycles, phases and plugin goals, and how this combination enables Maven to build your projects with enormous flexibility.

Here is the video explaining these.

{{< yt "https://www.youtube.com/embed/gOGrKEGBm0I" >}}





## Lifecycle

What is a Lifecycle?

{{< rawhtml >}}
<div class="notification">It is a series of stages through which something passes during its lifetime</div>
{{< /rawhtml >}}

In elementary school, we learnt about water cycle wherein water evaporates through ponds, rivers and oceans and goes into the atmosphere. There it condenses into water droplets which eventually fall back to the surface in the form of rain, thereby completing the cycle.

{{< rawhtml >}}
<div class="image">
    <a title="The Odd Git, Public domain, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Simple_Water_Cycle.JPG"><img width="512" alt="Simple Water Cycle" src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fd/Simple_Water_Cycle.JPG/512px-Simple_Water_Cycle.JPG"></a>
</div>
{{< /rawhtml >}}

A similar lifecycle in Software world is the Product Lifecycle. We start off with planning and designing the product, we develop it in programming language of our choice and test for all scenarios. We then put it out in the real world where the end-users use the product and provide us with feedback in terms of new features and bugs, which make up into new product planning and designing, completing the Software Product lifecycle.

{{< rawhtml >}}
<div class="image">
    <img src="/images/009-maven-lifecycles/software-product-lifecycle.png" alt="Software Product Lifecycle" />
</div>
{{< /rawhtml >}}

Similarly, Maven builds also have a lifecycle. There are pre-defined stages in which every build goes through, and Maven allows you to customize what happens during each of those stages.





{{< articlead >}}

## Lifecycle Phases

A Maven build life-cycle consists of predefined stages called `phases`. Their order of execution is also pre-defined. That way, you know which phase is executed first and which one is executed last. Each phase has clearly defined purpose.

We already learnt in our [first video](https://www.youtube.com/embed/JgFXdAmWoxQ), that whatever maven executes, it executes it from a plugin. To be more specific, when we say maven is executing a plugin, what maven is actually executing is a `goal` from that plugin.

A plugin goal represents a specific task which contributes to the building and managing of a project. In other words, a goal is simply a set of commands or instructions that maven can run to perform a task. A maven plugin can have one or more goals based on how much functionality a maven plugin provides. Maven plugins are very targeted, that means they strive to perform one and only one specific function for which they are written.

{{< rawhtml >}}
<div class="image">
    <img src="/images/009-maven-lifecycles/maven-lifecycle-phases.png" alt="Maven Lifecycle Phases" />
</div>
{{< /rawhtml >}}

Here we have 3 example plugins - `plugin-x, plugin-y` and `plugin-z`. They all have two goals in them.

Now, to tell Maven which plugin and goal to execute and when, we link the lifecycle phases to their corresponding plugin goals. In our example here, I am linking `phase-a` to `goal-x1` of `plugin-x`, `phase-b` to `goal-y1` of `plugin-y` and `goal-z1` to `plugin-z`.

When Maven starts to run this example lifecycle, it first executes `phase-a`. Maven looks for all plugin goals that are linked to this phase and then execute them one by one. So, during `phase-a`, Maven executes `goal-x1` of `plugin-x`. Since, that’s the only goal attached to this phase, `phase-a` is completed and Maven moves onto executing the next phase, which is `phase-b`.

For `phase-b`, `goal-y1` of `plugin-y` is linked, and thus Maven executes this goal. Again, no more goals are linked to `phase-b` apart from this, Maven marks this phase as completed and moves onto executing the next phase, and so on and so forth.

When all the phases are completed, Maven has executed the lifecycle and the build is complete.






{{< articlead >}}

## Maven Lifecycles

Maven has just [three lifecycles](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference) defined - `clean`, `default` and `site`. You can add your own custom one, but that would be hardly necessary. These three should suffice for all your build requirements.

`clean` lifecycle is responsible for cleaning up your project for a new build. Any files that were generated in previous build are removed during this lifecycle. Maven, by default, has plugin mapping for this lifecycle with `maven-clean-plugin`. So, you don’t need to configure anything from your side; another example of Maven’s convention over configuration. When you run this lifecycle, maven will automatically use `maven-compile plugin` to perform the cleanup of your project. This lifecycle is also independent of your project type - pom, jar, war, etc.

The next lifecycle is the `default` lifecycle. If you don’t specify which lifecycle to run, Maven runs the `default` lifecycle. This lifecycle is responsible for building your project. And that is why, this is the most commonly used lifecycle. It is also the most elaborate and has various phases to cater to all possible build situations. Since, every project is built differently, plugin bindings for the phases of this lifecycle varies as per the project type. In other words, if your project is of type jar, then default plugins used to build the project will differ from that of a project which is of type war or ejb or pom.

And finally, the `site` lifecycle which is responsible for generating the project documentation site. This is the least used one since not all projects generate documentation sites from their source code and these days, people prefer dedicated documentation site tools for such purposes. Like `clean` lifecycle, plugin bindings for `site` lifecycle are also configured out of the box regardless of the type of project that you might have.

Let’s look at the various phases in each of these lifecycles.





{{< articlead >}}

## Lifecycle - clean

The `clean` lifecycle is comprised of just three phase - `pre-clean, clean` and `post-clean` and they are executed in the same order.

Of the three lifecycle phases, the `clean` phase has binding to `clean` goal of [`maven-clean-plugin`](https://maven.apache.org/plugins/maven-clean-plugin/). This goal deletes your previous build directory, essentially cleaning it, so that you are ready to build your project from scratch. 

If you see there are a lot of clean keywords here, so a lot of times clean lifecycle, clean phase and clean goal are used interchangeably quite a lot. And that is ok as long as you are not getting into nitty gritty details. Just know that clean lifecycle, clean phase and clean goal are all different.



### Executing Lifecycle Phases

The way Maven executes the phases are by executing all phases that occur prior to the provided phase in the command. So when you say - `mvn pre-clean`, it executes only the `pre-clean` phase.

When you say `mvn clean`, Maven will execute both `pre-clean` and `clean` phases. So, whatever phase you give, all phases before that in the lifecycle are executed. This applies to all the lifecycles - `clean, default` and `site`. 

Similarly, if you want to execute all the phases of `clean` lifecycle, you will say - `mvn post-clean`, and all the phases will be executed.





{{< articlead >}}

## Lifecycle - default

The most familiar and elaborate lifecycle is `default`. It is the build lifecycle of the project. Any build tasks that Maven will execute to build your project, will be executed in one of the phases of `default` lifecycle.

The phases in this lifecycle can be categorized as -
- **Setup** - phases to let you set up the project for building
- **Compilation** - phases related to generation, processing and compilation of application class files
- **Testing** - phases related to generation, processing, compilation and unit testing of application
- **Packaging** - phases related packaging the application
- **Integration Test** - phases that allows you to manage and execute integration tests
- **Release** - phases that allows you to publish your packaged artifacts



### Lifecycle - default - Validation Phases

The first phase of default lifecycle is `validate`. This phase is intended for validation of the project, that it is correct and all necessary information is available to complete a build.

This phase is not linked to any plugin by default since projects can have very varied validation requirements. You could, for example, do some style checks during validation phase by configuring a style check plugin in this phase.

The next phase in the setup category is `initialize`. This phase is intended to initialize your build. A lot of times people configure `maven-clean-plugin` to this phase so that they don’t have to explicitly run the clean lifecycle. You can also initialize some environment variables if you like or any similar task, that is essential for the build in later phases.



### Lifecycle - default - Compilation Phases

The next set of phases in order relates to compilation of application class files. Here are all of them in their execution order.

- `generate-sources`
- `process-sources`
- `generate-resources`
- `process-resources`
- `compile`
- `post-compile`

The first phase in this is `generate-sources` and it is intended for generation of application code that is included during the compilation. For example, generation of class files from a SOAP WSDL using [cxf-plugin](https://cxf.apache.org/docs/maven-cxf-codegen-plugin-wsdl-to-java.html).

`process-sources` is intended for additional processing of generated source code. This phase usually complements with `generate-sources` phase.

`generate-resources` is intended for generating of application resource files and `process-resources` allows you to do any additional processing on those files.

By this time, project is ready for compilation and that is handled in the `compile` phase.

And finally, any final post compilation steps like bytecode enhancements, should be done in `process-compile` phase.





{{< articlead >}}

### Lifecycle - default - Testing Phases

The next set of phases in order relates to testing of the application. Here are all of them in their execution order.

- `generate-test-sources`
- `process-test-sources`
- `generate-test-resources`
- `process-test-resources`
- `test-compile`
- `test`

The phases here are very similar to compilation phases, only that everything happens with respect to test classes.

`generate-test-sources` is intended to generate the test source files and `process-test-sources` does additional processing on them.

`generate-test-resources` and `process-test-resources`, similarly does the generation of test resources and processing them.

Before we can run our tests, we need to compile the test source code in the `test-compile` phase and finally, tests are run in the `test` phase.



### Lifecycle - default - Packaging Phases

The next set of phases relates to packaging of the project. There are two phases in here

- `prepare-package`
- `package`

`prepare-package` which is intended for steps that prepare the project files for packaging. This phase usually results in an unpackaged version of project package.

And then `package` phase which allows to package the project as configured in pom.xml. It takes compiled classes, config files and any other files and then creates the final package(jar or war) as intended.





{{< articlead >}}

### Lifecycle - default - Integration-Test Phases

The integration test phases allow you to manage and run integration tests. There are three phases here

- `pre-integration-test`
- `integration-test`
- `post-integration-test`

`pre-integration-test` is intended for setting up the required environment for integration tests, and then the actual integration tests are run in the `integration-test` phase.

Once the integration tests are executed, you can clean up the environment or generate integration test report in the `post-integration-test` phase.



### Lifecycle - default - Release Phases

The last set of phases of default lifecycle relates to releasing of your application artifacts. Here are these

- `verify`
- `install`
- `deploy`

The first phase is `verify` that allows you to run checks to verify the packaged artifact and that it meets quality criterias.

The next phase is `install` which copies the artifact to the local maven repository in your system.

And last phase of the lifecycle is `deploy`, which copies the artifact to remote repositories like Maven Central or your company internal maven repository for sharing with other developers and projects.






## Lifecycle - default - Plugin Mappings

As said earlier, plugin mappings for `default` lifecycle depend on the type of project you have. Let’s see plugin mappings for default lifecycle for a few common Maven project types.



### Lifecycle - default - Plugin Mappings(JAR)

JAR packaging type is the most common type of Maven Project. The project is intended to generate a Java Archive or JAR file that packages all the project files. Maven has pre-configured plugin mappings for the various phases of default lifecycle to facilitate faster builds with minimum configuration.

Here are the default plugin mappings for it.

| Lifecycle Phase          | Plugin                  | Goal                      |
| ------------------------ | ----------------------- | ------------------------- |
| `process-resources`      | maven-resources-plugin  | `resources:resources`     |
| `compile`                | maven-compiler-plugin   | `compiler:compile`        |
| `process-test-resources` | maven-resources-plugin  | `resources:testResources` |
| `test-compile`           | maven-compiler-plugin   | `compiler:testCompile`    |
| `test`                   | maven-surefire-plugin   | `surefire:test`           |
| `package`                | maven-jar-plugin        | `jar:jar`                 |
| `install`                | maven-install-plugin    | `install:install`         |
| `deploy`                 | maven-deploy-plugin     | `deploy:deploy`           |

We have [`maven-resources-plugin`](https://maven.apache.org/plugins/maven-resources-plugin/) which processes the application and test resources, [`maven-compile-plugin`](https://maven.apache.org/plugins/maven-compiler-plugin/) which compiles the application code and test files. Tests are run using the [`maven-surefile-plugin`](https://maven.apache.org/surefire/maven-surefire-plugin/) and packaging to a jar is done by [`maven-jar-plugin`](https://maven.apache.org/plugins/maven-jar-plugin/), and `install` and `deploy` phases are taken care by [`maven-install-plugin`](https://maven.apache.org/plugins/maven-install-plugin/) and [`maven-deploy plugin`](https://maven.apache.org/plugins/maven-deploy-plugin/) respectively.



### Lifecycle - default - Plugin Mappings(POM)

POM is another very common packaging type wherein you intend to have the project’s pom imported in one or more other maven projects. There is no code to compile or test, and also there are no resources. That is why the default plugin mappings are provided only for `package, install` and `deploy`, as shown here.

| Lifecycle Phase | Plugin               | Goal                     |
| ----------------| ---------------------| ------------------------ |
| `package`       | maven-site-plugin    | `site:attach-descriptor` |
| `install`       | maven-install-plugin | `install:install`        |
| `deploy`        | maven-deploy-plugin  | `deploy:deploy`          |

The packaging isnt too complicated and [`maven-site plugin`](https://maven.apache.org/plugins/maven-site-plugin/) takes care of that, and `install` and `deploy` are again taken care byby [`maven-install-plugin`](https://maven.apache.org/plugins/maven-install-plugin/) and [`maven-deploy plugin`](https://maven.apache.org/plugins/maven-deploy-plugin/).





{{< articlead >}}

### Lifecycle - default - Plugin Mappings(Plugin)

If you creating a Maven plugin itself, you will be essentially creating the jar file which will have the code for performing the desired tasks. That is why, the plugin mappings are very similar to that of jar packaging.

| Lifecycle Phase          | Plugin                                       | Goal                                             |
| ------------------------ | -------------------------------------------- | ------------------------------------------------ |
| `generate-resources`     | maven-plugin-plugin                          | `plugin:descriptor`                              |
| `process-resources`      | maven-resources-plugin                       | `resources:resources`                            |
| `compile`                | maven-compiler-plugin                        | `compiler:compile`                               |
| `process-test-resources` | maven-resources-plugin                       | `resources:testResources`                        |
| `test-compile`           | maven-compiler-plugin                        | `compiler:testCompile`                           |
| `test`                   | maven-surefire-plugin                        | `surefire:test`                                  |
| `package`                | maven-jar-plugin<br/>maven-plugin-plugin     | `jar:jar`<br/>`plugin:addPluginArtifactMetadata` |
| `install`                | maven-install-plugin<br/>maven-plugin-plugin | `install:install`<br/>`plugin:updateRegistry`    |
| `deploy`                 | maven-deploy-plugin                          | `deploy:deploy`                                  |

To allow Maven recognize your jar as a Maven plugin, you need to provide some other metadata and Maven has goals mapped by default to generate this metadata. These are part of [`maven-plugin-plugin`](https://maven.apache.org/plugin-tools/maven-plugin-plugin/), to make your life easier.



### Lifecycle - default - Plugin Mappings(EJB)

EJB stands for [Enterprise Java Beans](https://en.wikipedia.org/wiki/Jakarta_Enterprise_Beans). EJB projects are not very popular these days, but if you are working on an old application, it could have the packaging type as EJB. They are a common data access mechanism for model-driven development in Enterprise Java. Here is the full mapping for reference.

| Lifecycle Phase          | Plugin                  | Goal                      |
| ------------------------ | ----------------------- | ------------------------- |
| `process-resources`      | maven-resources-plugin  | `resources:resources`     |
| `compile`                | maven-compiler-plugin   | `compiler:compile`        |
| `process-test-resources` | maven-resources-plugin  | `resources:testResources` |
| `test-compile`           | maven-compiler-plugin   | `compiler:testCompile`    |
| `test`                   | maven-surefire-plugin   | `surefire:test`           |
| `package`                | maven-ejb-plugin        | `ejb:ejb`                 |
| `install`                | maven-install-plugin    | `install:install`         |
| `deploy`                 | maven-deploy-plugin     | `deploy:deploy`           |

EJB plugin mappings are also very similar to JAR, the only difference being that during the `package` phase, it uses [`maven-ejb-plugin`](https://maven.apache.org/plugins/maven-ejb-plugin/) instead of [`maven-jar-plugin`](https://maven.apache.org/plugins/maven-jar-plugin/). Maven supports for both EJB2 and EJB3, and by default it is configured to look for files as per EJB2 specifications.



### Lifecycle - default - Plugin Mappings(WAR)

WAR packaging is again similar to JAR and EJBs. These are Web Applications that can be deployed to a compatible Web Application Server. Here is the full mapping for reference.

| Lifecycle Phase          | Plugin                  | Goal                      |
| ------------------------ | ----------------------- | ------------------------- |
| `process-resources`      | maven-resources-plugin  | `resources:resources`     |
| `compile`                | maven-compiler-plugin   | `compiler:compile`        |
| `process-test-resources` | maven-resources-plugin  | `resources:testResources` |
| `test-compile`           | maven-compiler-plugin   | `compiler:testCompile`    |
| `test`                   | maven-surefire-plugin   | `surefire:test`           |
| `package`                | maven-war-plugin        | `war:war`                 |
| `install`                | maven-install-plugin    | `install:install`         |
| `deploy`                 | maven-deploy-plugin     | `deploy:deploy`           |

Again, the difference here is the use of [`maven-war-plugin`](https://maven.apache.org/plugins/maven-war-plugin/) instead of [`maven-jar-plugin`](https://maven.apache.org/plugins/maven-jar-plugin/) or [`maven-ejb-plugin`](https://maven.apache.org/plugins/maven-ejb-plugin/).





{{< articlead >}}

## Lifecycle - site

You know that Maven also allows you to build a project site for your project. That is why Maven provides `site` as a dedicated lifecycle to achieve this.

It has 4 phases as below:
- `pre-site`
- `site`
- `post-site`
- `site-deploy`

First one is `pre-site`. It allows you to prepare the project files for generating a site. `site` phase allows to generate the project documentation site.

Once the project site is built, you might want to do some post processing(like minify css and javascript) and it can done on `post-site` phase. And finally, you can deploy your site to your web server in the `site-deploy` phase.

By default, [`maven-site-plugin`](https://maven.apache.org/plugins/maven-site-plugin/) has bindings to `site` and `site-deploy` phases, that you can configure to build and deploy your site.



And that is pretty much all about Maven Lifecycles.
