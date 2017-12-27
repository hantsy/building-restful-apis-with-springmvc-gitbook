# Configure DataSource

Instead of configuring `DataSource` in Java code.

Spring Boot provides built-in `application.properties`(or `application.yml` for YAML format) to configure `DataSource` in project specific file.

Create a *application.yml* in *src/main/resources* folder. It will override the default configuration. 

```yaml
server:
	port: 9000
	contextPath: 

spring: 
	profiles:
		active: dev
	devtools.restart.exclude: static/**,public/**
	datasource:
		dataSourceClassName: org.h2.jdbcx.JdbcDataSource
		url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
		databaseName: 
		serverName: 
		username: sa
		password: 

	jpa:
		database-platform: org.hibernate.dialect.H2Dialect
		database: H2
		openInView: false
		show_sql: true
		generate-ddl: true
		hibernate:
			ddl-auto: 
			naming-strategy: org.hibernate.cfg.EJB3NamingStrategy
		properties:
			hibernate.cache.use_second_level_cache: true
			hibernate.cache.use_query_cache: false
			hibernate.generate_statistics: true
			hibernate.cache.region.factory_class: org.hibernate.cache.internal.NoCachingRegionFactory

	data:
		jpa.repositories.enabled: true 
		
	freemarker:
		check-template-location: false
		
	messages:
		basename: messages

logging:
	file: app.log
	level:
		root: INFO
		org.springframework.web: INFO
		com.hantsylabs.restexample.springmvc: DEBUG
```
			
This is a classic application configuration file in *YAML* format.    

It is easy to understand.

**server.port** specifies the port number this application will serve at start up.

Under **spring** defines *DataSource*, *JPA*, *Spring Data JPA* etc.

**logging** configures logging level for packages.

**NOTE**: Spring Boot also supports *properties*, *groovy DSL* format for application configuration.