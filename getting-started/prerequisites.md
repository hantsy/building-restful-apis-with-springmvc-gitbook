#Prerequisites

## Java 8

Oracle Java 8 is recommended. For Windows user, just go to [Oracle Java website](http://java.oracle.com) to download it and install into your system. Redhat has just released a OpenJDK 8 for Windows user at DevNation 2016, if you are stick on the OpenJDK, go to [Redhat Developers website](https://developers.redhat.com) and get it.
	
Most of the Linux distributions includes the OpenJDK, install it via the Linux package manager.

Optionally, you can set **JAVA\_HOME** environment variable and add *&lt;JDK installation dir>/bin* in your **PATH** environment variable.

##Apache Maven
   
Download the latest Apache Maven from [http://maven.apache.org](http://maven.apache.org), and uncompress it into your local system. 

Optionally, you can set **M2\_HOME** environment varible, and also do not forget to append *&lt;Maven Installation dir>/bin* your **PATH** environment variable.  
	
If you are a Gradle fan, you can use Gradle as build tool. Gradle could be an alternative of Apache Maven.

##Spring ToolSuite

STS is an Eclipse based IDE, and built-in supports for many Spring features, it is highly recommended for new Spring users.

Go to Spring official site, download a copy of [Spring Tool Suite](https://spring.io/tools/sts). At the moment, the latest version is 3.8.
	
Extract the files into your local disk.
	
You can select your favorate IDEs, such as NetBeans, Intellij IDEA etc. All of these have good Maven and Gradle support.