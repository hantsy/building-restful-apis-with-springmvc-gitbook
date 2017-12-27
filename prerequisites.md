# Prerequisites

Before writing any codes, please install the latest JDK 8, Apache Maven, and your favorate IDE.

## Java 8

Oracle Java 8 is recommended. For Windows user, just go to [Oracle Java website](http://java.oracle.com) to download it and install into your system. Redhat has just released a OpenJDK 8 for Windows user at DevNation 2016, if you are stick on the OpenJDK, go to [Redhat Developers website](https://developers.redhat.com) and get it.
	
Most of the Linux distributions includes the OpenJDK, install it via the Linux package manager.

Optionally, you can set **JAVA\_HOME** environment variable and add *&lt;JDK installation dir>/bin* in your **PATH** environment variable.

Type this command in system terminal to verify your Java environment installed correctly.

```
#java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

## Apache Maven
   
Download the latest Apache Maven from [http://maven.apache.org](http://maven.apache.org), and uncompress it into your local system. 

Optionally, you can set **M2\_HOME** environment varible, and also do not forget to append *&lt;Maven Installation dir>/bin* your **PATH** environment variable.  

Type the following command to verify Apache Maven is working.

```
#mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: D:\build\maven
Java version: 1.8.0_102, vendor: Oracle Corporation
Java home: D:\jdk8\jre
Default locale: en_US, platform encoding: Cp1252
OS name: "windows 10", version: "10.0", arch: "amd64", family: "dos"
```	
	
If you are a Gradle fan, you can use Gradle as build tool. Gradle could be an alternative of Apache Maven.

## Lombok

I would like use Lombok to simply codes and make the codes clean. Go to [Lombok project](https://projectlombok.org/) to get know with Lombok.

There is a good [introduction to Lombok](http://www.baeldung.com/intro-to-project-lombok) from baeldung's blog.

## IDE 

The source codes are Maven based, it is IDE independent, so you can choose your favorite IDE. Nowadays the popular IDEs includes Eclipse, IDEA, NetBeans.

We will use JPA criteria metadata to provide type safe query, and use Lombok to simplify the codes, you have to enable **Annotation Processing** feature in your IDEs. 

### Spring ToolSuite

Spring Tool Suite is an Eclipse based IDE, and provides a lot of built-in Spring supports, it is highly recommended for new Spring users.

Go to [Spring official site](https://spring.io), download a copy of [Spring Tool Suite](https://spring.io/tools/sts). At the moment, the latest version is 3.8.

Alternatively, you can download a copy of Eclipse Java EE bundle from [Eclise official website](https://www.eclipse.org), and install the STS plugin from Eclipse Marketplace.
	
Extract the files into your local disk. Go to root folder, there is **STS.exe** file, double click it and starts up Spring Tool Suite.

1. Go to Windows/Preference menu, and open Preference dialog
2. Search `Annotation`...
3. Expand *Compiler/ Annotation Processing* , enable **Annotation Processing**.
4. Expand *Maven/Annotation Processing*, enable **Annotation Processing**. If it does not exists, install **m2e-apt** in **Maven/Discovery** firstly.
5. Apply all changes.

Go to Lombok project website, and follow the official [the installation guideline](https://projectlombok.org/setup/eclipse)) to install Lombok plugin into your Eclipse IDE.

### Intellij IDEA	

No doubt, [Intellij IDEA](https://www.jetbrains.com/idea) is the most productive Java IDE. It includes free and open source community version and enterprise version.

1. Go to File / Settings 
2. Search `annotation processor`
3. Enable Annotation processing

You can install Lombok plugin from IDEA plugin manager to get Lombok support in your IDEA.

1. Go to File / Settings / Plugins
2. Click on Browse repositories...
3. Search for `Lombok Plugin`
4. Click on Install plugin
5. Restart IDE 

### NetBeans

[NetBeans](http://www.netbeans.org) is the simplest IDE for Java development, which was originally brought by Sun microsystem(and later maintained by Oracle), it is free and open source. 

Now Oracle denoted it as [an incubator project under Apache Foundation](http://netbeans.apache.org).

Download a copy of NetBeans from [NetBeans website](https://netbeans.org)(it is still working before Apache hand over it).

For NetBeans users, there is no need to setup Annotation Processing and Lombok, NetBeans has activated Annotation processing capability by default.

In the next posts, let's try to create a project skeleton for our blog sample application.