#Maven Profiles and Spring Profiles

Similar with the former vanilla version, we can define some Maven profiles to inject configuration for different project stages, such as *dev*, *staging*, *prod*. etc.

Open the project `pom.xml` file.

```xml
<profile>
	<id>dev</id>
	<activation>
		<activeByDefault>true</activeByDefault>
	</activation>
	<properties>
		<spring.profiles.active>dev</spring.profiles.active>
		<log4j.level>DEBUG</log4j.level>
	</properties>
	<build>
		<resources>
			<resource>
				<directory>src/main/resources-dev</directory>
				<filtering>true</filtering>
			</resource>
		</resources>
	</build>
</profile>
```

The above Maven profile(*dev*) will be activated by default, and it will add */src/main/reources-dev* as resource folder.

The *application.yml* under */src/main/reources-dev* will be packaged into the final package(jar or war).

We can define some other profile based *application.yml* for different stages. Every *application.yml* will add different configuration, such as in development stage, use a H2 database for easy testing, and in production profile, the *application.yml* will use a pool `DataSource` for better performance at runtime.

The above approach combine Maven profiles and Spring profile to get clean and simple configuration for applications.

Alternatively, Spring Boot provides more powerful capability to switch profile via environment variables. You can package all profile based configuration in the same application package, and add a parameter to select a profile when the application is bootstrapping.

```
java -jar ./app.jar -Dspring.profiles.active=staging
```


