#Configure DataSource

In order to use Hibernate, Jdbc, or JPA similar persistence framework or tools, you have to configure a `java.sql.DataSource` for it.

Spring DataSource support is available in `sring-jdbc`. Added it into your *pom.xml*.

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
</dependency>
```

A simple `DataSouce` configuration looks like.

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource testDataSource() {
        BasicDataSource bds = new BasicDataSource();
        bds.setDriverClassName("com.mysql.jdbc.Driver");
        bds.setUrl("jdbc:mysql://localhost:3306");
        bds.setUsername("jdbc.username");
        bds.setPassword("jdbc.password");
        return bds;
    }

}
```

Here, I uses Apache Commons Dbcp's `BasicDataSource` to build a `DataSource`. It is configured for MySQL database, before use it, do not forget to add mysql driver into *pom.xml*.

```xml
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

Declares this configuration class in `getRootConfigClasses` method of `AppInitializer`.

In above codes, we set username, password etc in hard codes, but in a real application, it is better to externalize these configurations into a property file.

Create another `@configuration` class for this purpose.

```java
@Configuration
@ComponentScan(
    basePackageClasses = {Constants.class},
    excludeFilters = {
        @Filter(
            type = FilterType.ANNOTATION,
            value = {
                RestController.class,
                ControllerAdvice.class,
                Configuration.class
            }
        )
    }
)
@PropertySource("classpath:/app.properties")
@PropertySource(value = "classpath:/database.properties", ignoreResourceNotFound = true)
public class AppConfig {

}
```

`AppConfig` work as an entr configuration for this application. `@ComponentScan` use a fitler to load all none web components.

Use `@PropertySource` to load the external properties files, `app.properties` is use for application properties, and database.properties for holding database datasource properties.

```
jdbc.url=@jdbc.url@
jdbc.username=@jdbc.username@
jdbc.password=@jdbc.password@
hibernate.dialect=@hibernate.dialect@
```

In DataSouce configuration, use `Environment` to fetch these properties.

```java
private static final String ENV_JDBC_PASSWORD = "jdbc.password";
private static final String ENV_JDBC_USERNAME = "jdbc.username";
private static final String ENV_JDBC_URL = "jdbc.url";

@Inject
private Environment env;

@Bean
public DataSource testDataSource() {
	BasicDataSource bds = new BasicDataSource();
	bds.setDriverClassName("com.mysql.jdbc.Driver");
	bds.setUrl(env.getProperty(ENV_JDBC_URL));
	bds.setUsername(env.getProperty(ENV_JDBC_USERNAME));
	bds.setPassword(env.getProperty(ENV_JDBC_PASSWORD));
	return bds;
}
```

Spring Jdbc provides a simple `EmbeddedDatabaseBuilder` to build an embedded datasource on the fly way.

```java
@Bean
public DataSource dataSource() {
	return new EmbeddedDatabaseBuilder()
			.setType(EmbeddedDatabaseType.H2)
			.build();
}
```

Here we build an embedded H2 datasource. 

An embedded datasource is every helpful for development stage, everytime when we run the application, or run the tests, we are getting a fresh runtime environment.

Spring Jdbc also provides other built-in DataSource, such as DriverManagerDataSource, and some application server specific DataSource, eg. for Webphere.

For a production runtime environment, we should use pooled datasource, such as Apache Commons Dbcp, or application server built-in DataSource to get better performance. 

We have discussed the usages of Apache Commons Dbcp earlier, you can add extra pool configuration for this datasource. 

For application server built-in DataSource, Spring can access it via a Jndi proxy. Firstly configure a Jndi DataSource in appliation server GUI, then defines `JndiObjectFactoryBean` to access it via Jndi name.

```java
@Bean
public DataSource prodDataSource() {
	JndiObjectFactoryBean ds = new JndiObjectFactoryBean();
	ds.setLookupOnStartup(true);
	ds.setJndiName("jdbc/postDS");
	ds.setCache(true);

	return (DataSource) ds.getObject();
}
```	

The complete codes of `DataSouceConfig`.

```java
@Configuration
public class DataSourceConfig {

    private static final String ENV_JDBC_PASSWORD = "jdbc.password";
    private static final String ENV_JDBC_USERNAME = "jdbc.username";
    private static final String ENV_JDBC_URL = "jdbc.url";

    @Inject
    private Environment env;

    @Bean
    @Profile("dev")
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }

    @Bean
    @Profile("staging")
    public DataSource testDataSource() {
        BasicDataSource bds = new BasicDataSource();
        bds.setDriverClassName("com.mysql.jdbc.Driver");
        bds.setUrl(env.getProperty(ENV_JDBC_URL));
        bds.setUsername(env.getProperty(ENV_JDBC_USERNAME));
        bds.setPassword(env.getProperty(ENV_JDBC_PASSWORD));
        return bds;
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        JndiObjectFactoryBean ds = new JndiObjectFactoryBean();
        ds.setLookupOnStartup(true);
        ds.setJndiName("jdbc/postDS");
        ds.setCache(true);

        return (DataSource) ds.getObject();
    }

}
```

Three DataSouce beans are configured. Do not worry about the `@Profile` annotation, I will explain it in a *Spring Profile* related section for it.




