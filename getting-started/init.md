# Create project skeleton

In these days, more and more poeples are using Spring Boot to get autoconfiguration support and quicker build lifecycle.

For those new to Spring, the regular approache\(none Spring Boot\) is more easy to understand Spring essential configurations.

Let's us create a Maven based Java EE 7 web application to introduce how to configure a Spring MVC web application in details, then switch to Spring Boot, thus you can compare these two approaches, and understand how Spring Boot simplifies configurations.

## Create a Maven based web project

 Use the official Java EE 7 web archetype to generate the project skeleton, replace the value of _archetypeId_ and _package_ to yours.

```
mvn -DarchetypeGroupId=org.codehaus.mojo.archetypes 
-DarchetypeArtifactId=webapp-javaee7 
-DarchetypeVersion=1.1 
-DgroupId=your_group_id 
-DartifactId=angularjs-springmvc-sample 
-Dversion=1.0.0-SNAPSHOT
-Dpackage=com.hantsylabs.restsampels
-Darchetype.interactive=false 
--batch-mode  
archetype:generate
```

## Add Spring dependencies.

Add Spring IO platform `platform-bom` to the _dependencyManagement_ section in *pom.xml*.

  ```
   <dependencyManagement>
       <dependencies>
           <!-- Spring BOM -->
           <dependency>
               <groupId>io.spring.platform</groupId>
               <artifactId>platform-bom</artifactId>
               <version>2.0.6.RELEASE</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
  ```

You can also use _platform-bom_ as parent of this project.

Read the Apache Maven docs to understand [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html).

`platform-bom` manages all dependencies of Spring projects, and you can add dependency declaration directly without specifying a version. `platform-bom` manages the versions, and resolved potential conflicts for you.

In order to get IOC container support, you have to add the several core dependencies into *pom.xml*.

```xml
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-core</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-context</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-beans</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-context-support</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aop</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aspects</artifactId>
</dependency>
```

If you would like use `@Inject` instead of `@Autowire` in codes, add _inject_ dependency.

```xml
<dependency>
   <groupId>javax.inject</groupId>
   <artifactId>javax.inject</artifactId>
   <version>1</version>
</dependency>    
```

Get the [codes](https://github.com/hantsy/angularjs-springmvc-sample) from my github account to explore all configuration classes.

