---
date: 2020-08-01
linktitle: Install Apache Maven on Linux
next: /2020/08/install-apache-maven-on-mac/
title: Install Apache Maven on Linux
weight: 10
tags: [ "maven", "devops", "tools"]
categories: [ "DevOps", "Maven", "Tools" ]
---

This article gives you a step by step walkthrough for installing [Apache Maven](https://maven.apache.org/) on Linux. 

You can watch the entire installation demo in this video.

{{< yt "https://www.youtube.com/embed/JgFXdAmWoxQ" >}}





## Introduction

[Apache Maven](https://maven.apache.org/) is the most popular build and project management tool for Java projects. Maven is built on top of Java, so if you plan to install Maven on your system, you need to have Java installed on your system.

Installation of Maven on any system consists of following 3 steps.

1. Download the Apache Maven binary zip from the offical website
2. Extracting the binary zip into a specific folder
3. Configure the environment variables of the System.

Let me walk you through these steps in detail for a Linux system.





### 1. Downloading the Apache Maven

The first step is to download the [Apache Maven](https://maven.apache.org/) from the official website. The downloads page for Apache Maven is this - https://maven.apache.org/download.cgi. On this page, you have two binary options to download - either a zip file or a tar.gz file(marked in blue boxes). Here is the screenshot below.

![Apache Maven Download Files](/images/001-maven-install-linux-mac/maven-downloads-binary.png)

You can chose either of these. If you are not sure, chose the zip file. Also, download the corresponding checksum file(marked in orange boxes) so that you can validate the zip file download.

You can use your browser to download these, or you can use the `wget` or `curl` on the command prompt to download the files.

Here is the command using the `wget` that download the zip file and the corresponding checksum file.

```
-> wget -q \
     --show-progress https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip

apache-maven-3.6.3-bin.zip            100%[============================>]   9.16M  13.5MB/s    in 0.7s    

-> wget -q \
     --show-progress https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip.sha512
apache-maven-3.6.3-bin.zip.sha512     100%[============================>]     128  --.-KB/s    in 0s

-> ls -l
total 9384
-rw-rw-r-- 1 codejournal codejournal 9602303 Nov 19  2019 apache-maven-3.6.3-bin.zip
-rw-rw-r-- 1 codejournal codejournal     128 Nov 19  2019 apache-maven-3.6.3-bin.zip.sha512
```




### 2. Extracting the Binary archive File

#### 2.1. Verifying the Checksum
Before you extract the zip or tar.gz file, perform a quick checksum validation of the downloaded file to ensure that there were no issues with the download.

The checksum files mentions which kind of hash is used for zip file. Its sha512, and hence we need [`sha512sum`](https://linux.die.net/man/1/sha512sum) command to calculate the checksum of the zip file. This command is part of `coreutils` package and if you don't have install, you can install it with your package manager. Here is a [list of installation commands](https://command-not-found.com/sha512sum) for installing coreutils on your Linux distribution.

```
-> sha512sum apache-maven-3.6.3-bin.zip
1c095ed556eda06c6d82... (removed from brevity) ...c541af234eded94d  apache-maven-3.6.3-bin.zip

-> cat apache-maven-3.6.3-bin.zip.sha512 
1c095ed556eda06c6d82... (removed from brevity) ...c541af234eded94d
```

A quick glance, you would see that both the checksums are same and that ensures that the download was completed without any errors.



#### 2.2. Installation Directory

Before we extract the binary archive file, we need a directory. It is a good practice to keep all of your installations together, so you can place them in a directory called `installed` in your home(`~`) directory.

You can create this directory if it does not exist using the `mkdir` command. Inside this `installed` directory, we need to create our `maven3` directory where we will extract our binary archive file. You can create these directories like this.

```
-> mkdir ~/installed

-> mkdir ~/installed/maven3
```

Afterwards, we need to move our binary archive file in this `maven3` directory, and then we `cd` into this directory so that we can extract the archive file here.

```
-> mv apache-maven-3.6.3-bin.zip ~/installed/maven3/

-> cd ~/installed/maven3
```



#### 2.3. Extracting the Binary file

If you have downloaded the zip file, you will need `unzip` command to extract the zip file. After extracting the zip file, you wont need it, so you can delete that.

```
-> unzip -q apache-maven-3.6.3-bin.zip

-> rm apache-maven-3.6.3-bin.zip -f
```

And if you have downloaded the tar.gz file, you will need `tar` command.

```
-> tar -xf apache-maven-3.6.3-bin.tar.gz

-> rm apache-maven-3.6.3-bin.tar.gz -f
```



#### Creating a Symbolic Link

You don't need to create a symbolic link to the newly extracted folder, but it will make your life easier if a symbolic link always pointed to latest Maven installation.

So, if next time, you upgrade the Maven installation from `3.6.3` to `3.6.4`, you can just update the symbolic link and everything should continue to work as it is. You won't have to update any environment variables(discussed in the next section)

Here is how you will create a symbolic link.

```
-> ln -s apache-maven-3.6.3 latest
```





### Configure the Environment Variables

With the extraction of binary archive file in the `installed/maven3` directory, Maven is pretty much installed in your system. The only gotcha is that everytime you need to run the `mvn` command, you will have to traverse the entire path - `~/installed/maven3/latest/bin/mvn`. It will be much better, if we could just run the `mvn` command from any directory without specifying the entire path to the `mvn` executable.

A lot of Java tools look for `M2_HOME` environment variable to locate the Maven installation directory in the system. This environment variable can be configured in any of your `.bash_profile, .bashrc, .profile` file. This is the entry that you will need to put.

```
export M2_HOME="~/installed/maven3/latest"
```

Once, this is configured, we need to configure the `PATH` variable, that will enable running of the `mvn` command from any directory. Here is how, it will be configured in your `.bashrc or .bash_profile` file.

```
export PATH="$PATH:$M2_HOME/bin"
```

Once this is configured, you need to re-import the configurations. Simple close the current terminal window and open a new one.

And now if you run `mvn -version`, you should see the Maven installation details confirming that you have Apache Maven installed successfully.

```
-> mvn -version

Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /home/codejournal/installed/maven3/latest
Java version: 1.8.0_265, vendor: Private Build, runtime: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.15.0-117-generic", arch: "amd64", family: "unix"
```

Now, you can start to run your builds locally with Maven on your Linux system.