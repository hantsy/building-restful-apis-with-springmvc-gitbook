#Maven Profiles and Spring Profiles

Spring provides `@Profile` annotation to declare beans for specific profiles. You can specify a **spring.profiles.active** environment varible to activiate which beans will be used in Spring applications.

In this sample application, I would like use Maven profile to specify the **spring.profiles.active** for different development stage.

* For development stage, I would like an embeded database, such as H2. It is easy to user, and start up with a clean environment for test, I also like enable logging debug and get more log info.

* For staging stage, I would like use similar environment with produciton to run the application with a CI server, such as Jenkins, Travis CI, Circle CI etc.

* For produciton, enable all cache and performance optimization, use application server container managed datasource, only enable essential logging tracking etc.

We could plan some other profiles for UAT, etc.  

In my [angularjs-springmvc-sample](https://github.com/hantsy/angularjs-springmvc-sample), I defined 3 Maven profile for different purposes as described above.

```
<profiles>
	<profile>
		<id>dev</id>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
		<properties>

			<log4j.level>DEBUG</log4j.level>

			<spring.profiles.active>dev</spring.profiles.active>
			<!-- hibernate -->
			<hibernate.hbm2ddl.auto>create</hibernate.hbm2ddl.auto>
			<hibernate.show_sql>true</hibernate.show_sql>
			<hibernate.format_sql>true</hibernate.format_sql>

			<!-- mail config -->
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
	<profile>
		<id>staging</id>
		<properties>
			<jdbc.url><![CDATA[jdbc:mysql://localhost:3306/app]]>
			</jdbc.url>
			<jdbc.username>root</jdbc.username>
			<jdbc.password></jdbc.password>

			<log4j.level>INFO</log4j.level>

			<spring.profiles.active>staging</spring.profiles.active>

			<hibernate.hbm2ddl.auto>update</hibernate.hbm2ddl.auto>
			<hibernate.show_sql>false</hibernate.show_sql>
			<hibernate.format_sql>false</hibernate.format_sql>
			<hibernate.dialect>org.hibernate.dialect.MySQL5Dialect</hibernate.dialect>

		</properties>
		<dependencies>
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
			</dependency>
		</dependencies>
		<build>
			<resources>
				<resource>
					<directory>src/main/resources-staging</directory>
					<filtering>true</filtering>
				</resource>
			</resources>
		</build>
	</profile>
	<profile>
		<id>prod</id>
		<properties>
			<log4j.level>INFO</log4j.level>

			<spring.profiles.active>prod</spring.profiles.active>

			<hibernate.hbm2ddl.auto>none</hibernate.hbm2ddl.auto>
			<hibernate.show_sql>false</hibernate.show_sql>
			<hibernate.format_sql>false</hibernate.format_sql>
			<hibernate.dialect>org.hibernate.dialect.MySQL5Dialect</hibernate.dialect>
		</properties>
		<build>
			<resources>
				<resource>
					<directory>src/main/resources-prod</directory>
					<filtering>true</filtering>
				</resource>
			</resources>
		</build>
	</profile>
</profiles>
```

In every Maven profiles, there is a *spring.profiles.active* property, its value will be filled in *applicaiton.properties* file and replaces the placeholder **@spring.profiles.active@** in this file, the properties files can be categoried in different files for varied purposes, such as database connection, global application settings etc. They are placed in the profile specific resource folder(defined in *resources* element in every profiles section).

Append a **-P** parameter to switch which Maven profile will be applied. eg.

```
mvn clean package -Pprod 
```

The above command will package the applicatin for **prod** profile, it also apply **spring.profiles.active** value(`prod`) for this application when it is running.

For exmaple, for the DataSource configuration in `DataSourceConfig`, the `@Profile("prod")` annotated bean will be activated.

```
@Bean
@Profile("prod")
public DataSource prodDataSource() {
	JndiObjectFactoryBean ds = new JndiObjectFactoryBean();
	ds.setLookupOnStartup(true);
	ds.setJndiName("jdbc/postDS");
	ds.setCache(true);

	return (DataSource) ds.getObject();
}
```

It uses the application server container managed DataSource for better performance and also easy to be monitored by application server console. Tomcat, Glassfish, Weblogic all provide friendly UI for administration.






