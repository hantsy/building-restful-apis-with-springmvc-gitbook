# Configure JPA

JPA was proved a greate success in Java community, and it is wildly used in Java applications, including some desktop applications.

Spring embraces JPA specification in the first instance.

We have configured a DataSource, to support JPA in Spring, we need to configure a JPA specific `EntityManagerFactoryBean` and a `PlatformTransactionManager`.

Add `spring-orm` into *pom.xml*.

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-orm</artifactId>
</dependency>
```

Use Hibernate as the JPA provider.

```
<!--java persistence API 2.1 -->
<dependency>
	<groupId>org.hibernate.javax.persistence</groupId>
	<artifactId>hibernate-jpa-2.1-api</artifactId>
	<version>1.0.0.Final</version>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-core</artifactId>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-entitymanager</artifactId>
	<exclusions>
		<exclusion>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
		</exclusion>
		<exclusion>
			<groupId>dom4j</groupId>
			<artifactId>dom4j</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

Cooperate with Hibernate Core, we also use Bean Validation and `hibernate-validator` to generate database schema constraints.

```xml
<!--bean validation -->
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-validator</artifactId>
</dependency>
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>1.1.0.Final</version>
</dependency>
```

The following is the complete codes of `JpaConfig`.

```java
@Configuration
@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)
public class JpaConfig {

    private static final Logger log = LoggerFactory.getLogger(JpaConfig.class);

    private static final String ENV_HIBERNATE_DIALECT = "hibernate.dialect";
    private static final String ENV_HIBERNATE_HBM2DDL_AUTO = "hibernate.hbm2ddl.auto";
    private static final String ENV_HIBERNATE_SHOW_SQL = "hibernate.show_sql";
    private static final String ENV_HIBERNATE_FORMAT_SQL = "hibernate.format_sql";

    @Inject
    private Environment env;

    @Inject
    private DataSource dataSource;

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
        emf.setDataSource(dataSource);
        emf.setPackagesToScan("com.hantsylabs.restexample.springmvc");
        emf.setPersistenceProvider(new HibernatePersistenceProvider());
        emf.setJpaProperties(jpaProperties());
        return emf;
    }

    private Properties jpaProperties() {
        Properties extraProperties = new Properties();
        extraProperties.put(ENV_HIBERNATE_FORMAT_SQL, env.getProperty(ENV_HIBERNATE_FORMAT_SQL));
        extraProperties.put(ENV_HIBERNATE_SHOW_SQL, env.getProperty(ENV_HIBERNATE_SHOW_SQL));
        extraProperties.put(ENV_HIBERNATE_HBM2DDL_AUTO, env.getProperty(ENV_HIBERNATE_HBM2DDL_AUTO));
        if (log.isDebugEnabled()) {
            log.debug(" hibernate.dialect @" + env.getProperty(ENV_HIBERNATE_DIALECT));
        }
        if (env.getProperty(ENV_HIBERNATE_DIALECT) != null) {
            extraProperties.put(ENV_HIBERNATE_DIALECT, env.getProperty(ENV_HIBERNATE_DIALECT));
        }
        return extraProperties;
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new JpaTransactionManager(entityManagerFactory().getObject());
    }
}
```

Generally, in a Java EE web application, JPA is activated by *META-INF/persistence.xml* file in the application archive. 

The following is a classic JPA *persistence.xml*. 

```xml
<persistence version="2.1"
   xmlns="http://xmlns.jcp.org/xml/ns/persistence" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
		http://xmlns.jcp.org/xml/ns/persistence
		http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
   <persistence-unit name="primary" transaction-type="RESOUECE_LOCAL">

	  <class>...Post</class>
	  <none-jta-data-source/>
	  <properties>
		 <!-- Properties for Hibernate -->
		 <property name="hibernate.hbm2ddl.auto" value="create-drop" />
		 <property name="hibernate.show_sql" value="false" />
	  </properties>
   </persistence-unit>
</persistence>
```

The transaction-type is `RESOUECE_LOCAL`, another available option is `JTA`. Most of the time, Spring applications is running in a Servlet container which does not support `JTA` by default. 

You have to specify entity classes will be loaded and datasource here. Spring provides an alternative, `LocalEntityManagerFactoryBean` to simplify the configuration, there is a `setPackagesToScan` method provided to specify which packages will be scanned, and another `setDataSource` method to setup Spring DataSource configuration and no need to use database connection defined in the *persistence.xml* file at all.

More simply, `LocalContainerEntityManagerFactoryBean` does not need a *persistence.xml* file any more, and it builds JPA environment from ground. In above codes, we use `LocalContainerEntityManagerFactoryBean` to shrink JPA configuration.

Next, configure Spring Data JPA, add `spring-data-jpa` into pom.xml.

```xml
<!-- Spring Data -->
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-commons</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-jpa</artifactId>
</dependency>
```

Enable Spring Data JPA, use a standalone configuration class.

```java
@Configuration
@EnableJpaRepositories(basePackages = {"com.hantsylabs.restexample.springmvc"})
public class DataJpaConfig {

}
```

Spring Data Commons provides a series of `Repostory` APIs. `basePackages` in `@EnableJpaRepositories` will tell Spring Data JPA to scan `Repository` classes in these packages.

Do not forget to add `JpaConfig` and `DataJpaConfig` configuration classes into `getRootConfigClasses` method of `AppInitializer`.


