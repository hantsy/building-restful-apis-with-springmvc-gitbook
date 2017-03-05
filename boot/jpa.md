# Configure JPA

Make sure the following dependencies are added in your `dependency` section of the project `pom.xml` file.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

It will add managed JPA, Spring Data JPA and Hibernate into your project.

We have already configured `DataSource`, and JPA and Spring Data JPA via `application.yml`.

You can also use Java code configuration to add some extra features.

```java
@Configuration
@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)
@EntityScan(basePackageClasses = {User.class, Jsr310JpaConverters.class})
@EnableJpaAuditing(auditorAwareRef = "auditor")
public class JpaConfig {

    @Bean
    public AuditorAware<User> auditor() {
        return () -> SecurityUtil.currentUser();
    }

}
```

In the above codes we add `@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)` to enable transaction management and use `AspectJ` to weave transaction aspect into your business logic.

And `@EnableJpaAuditing(auditorAwareRef = "auditor")` enables the simple auditing features provided in Spring Data JPA. It requires a `AuditorAware` bean.

`@EntityScan(basePackageClasses = {User.class, Jsr310JpaConverters.class})` add the JPA entity scan scope, Java 8 DateTime support is added in Spring Data JPA via JPA 2.1 `AttributeConvertor` feature.

 