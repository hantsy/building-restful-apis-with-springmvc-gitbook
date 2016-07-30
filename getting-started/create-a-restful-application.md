# Create a Spring MVC application

In these days, more and more poeples are using Spring Boot to get autoconfiguration support and quicker build lifecycle.

For those new to Spring, the regular approache\(none Sprinb Boot\) is more easy to understand Spring essential configurations.

Let's us create a Maven based Java EE 7 web application to introduce how to configure a Spring MVC web application in details, then switch to Spring Boot, thus you can compare these two approaches, and understand how Spring Boot simplifies configurations.

## Create a regular web project

1. Create a Maven web project.

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

2. Add Spring IO platform `platform-bom` to the _dependencyManagement_.

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

3. Add essential dependencies for Spring MVC.

  ```
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-core</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-orm</artifactId>
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

   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-tx</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-web</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-websocket</artifactId>
   </dependency>
   <!-- Spring security -->
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-core</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-config</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-web</artifactId>
   </dependency>

   <!-- Spring Data -->
   <dependency>
       <groupId>org.springframework.data</groupId>
       <artifactId>spring-data-commons</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework.data</groupId>
       <artifactId>spring-data-jpa</artifactId>
   </dependency>    

   <dependency>
       <groupId>org.hibernate</groupId>
       <artifactId>hibernate-entitymanager</artifactId>
   </dependency>

   <!--bean validation -->
   <dependency>
       <groupId>org.hibernate</groupId>
       <artifactId>hibernate-validator</artifactId>
   </dependency>
  ```

  Besides Spring core dependencies, Spring MVC, Spring Security , Hibernate, JPA\/Spring Data JPA, Bean validation are also required.

  If you would like use `@Inject` instead of `@Autowire` in codes, add _inject_ dependency.

  ```
   <dependency>
       <groupId>javax.inject</groupId>
       <artifactId>javax.inject</artifactId>
       <version>1</version>
   </dependency>    
  ```

4. Enable Spring MVC `DisptachServlet`.

  Declare a `AbstractAnnotationConfigDispatcherServletInitializer` bean.

  ```
   @Order(0)
   public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

       @Override
       protected Class<?>[] getRootConfigClasses() {
           return new Class[] {
                   AppConfig.class, //
                   DataSourceConfig.class, //            
                   JpaConfig.class, //
                   DataJpaConfig.class,//
                   SecurityConfig.class,//   
                   Jackson2ObjectMapperConfig.class,//
                   MessageSourceConfig.class
           };
       }

       @Override
       protected Class<?>[] getServletConfigClasses() {
           return new Class[] {
               WebConfig.class, //
               SwaggerConfig.class //
           };
       }

       @Override
       protected String[] getServletMappings() {
           return new String[] { "/" };
       }

       @Override
       protected Filter[] getServletFilters() {
           CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
           encodingFilter.setEncoding("UTF-8");
           encodingFilter.setForceEncoding(true);

           return new Filter[] { encodingFilter };
       }

   }
  ```

  Serlvet 3.0 provides a new feature _ServletInitializer_ to configure web applciation without _web.xml_.

  Spring has its _WebApplicationInitializer_ interface, there are a few classes implement this interface, `AbstractAnnotationConfigDispatcherServletInitializer` includes configuration of Spring Dispatch Servlet, and leave some room to customize `DispatchServlet`.

  `getRootConfigClasses` specifies the configuration classes should be loaded for the Spring infrastrucuture.

  `getServletConfigClasses` specifies the configurations depend on Servlet specification, esp, web mvc related configurations.

  `getServletMappings` is the Spring `DispatchServlet` mapping URL pattern.

  `getServletFilters` are the filters will be applied on the Spring `DispatchServlet`.

  Spring MVC `DispatchServlet` is configured in the super classes, explore the details if you are interested in it.

5. Enable Spring Security

  ```
   @Order(1)
   public class SecurityInitializer extends AbstractSecurityWebApplicationInitializer {

   }    
  ```

  Similiar with `AbstractAnnotationConfigDispatcherServletInitializer`, it is a `WebApplicationInitializer` implementation, and aleady configured  Spring Security filter chain for you.

6. Configuration classes.

  Therea are a few configuration classes already created for this sample application, as you saw in above `AppInitializer`.

  Every configuration should be annotated with `@Configuration`.

  It is recommended to slipt the configurations into different classes by categories. eg. In this sample the configuration classes includes:

  * `AppConfig` is the base configuration.
  * `DataSourceConfig` is the configuration for `DataSource`.
  * `JpaConfig` is the configuration for JPA and transacation.
  * `DataJpaConfig` is the configuration for Spring Data JPA extension support.
  * `SecurityConfig` is the Spring Security configuration.
  * `Jackson2ObjectMapperConfig` is the customized Jackson `ObjectMapper` bean.
  * `MessageSourceConfig` is the configuration for `MessageSource`.
  * `WebConfig` is the configuration for Spring MVC.
  * `SwaggerConfig` is the configuration for Swagger schema APIs.


    `@ComponentScan` defines the component scanning strategy for loading Spring `Component` at the application starts up. 

    The **excludeFilters** property define rules to exclude some components. `AppConfig` only scans none web beans, and `WebConfig` only scan web relates beans.

    **@PropertySource** configures the properties file will be loadded by Spring, later you can read the properties by `Environment`. Check the usage in `DataSourceConfig` class.

    Have a look at the content of `WebConfig` which is responsible for configuring Spring MVC in details, including resource handling, view, view resolvers etc.

       public class WebConfig extends SpringDataWebConfiguration {

       }

    Generally, `WebMvcConfigurerAdapter` is use for customizing MVC details. `SpringDataWebConfiguration` is a subclass of `WebMvcConfigurerAdapter`, it is from Spring data project, and add pagination, sort, and domain object conversion support. Open the source code of `SpringDataWebConfiguration` and research yourself.

    `SecurityConfig` is the Spring security configuration details.

        @Configuration
        @EnableWebSecurity
        public class SecurityConfig extends WebSecurityConfigurerAdapter {
            @Override
            public void configure(WebSecurity web) throws Exception {
                web
                    .ignoring()
                    .antMatchers("/**/*.html", //
                                 "/css/**", //
                                 "/js/**", //
                                 "/i18n/**",// 
                                 "/libs/**",//
                                 "/img/**", //
                                 "/webjars/**",//
                                 "/ico/**");
            }

            @Override
            protected void configure(HttpSecurity http) throws Exception {
                    http   
                        .authorizeRequests()   
                        .antMatchers("/api/**")
                        .authenticated()
                    .and()
                        .authorizeRequests()   
                        .anyRequest()
                        .permitAll()
                        .and()
                            .sessionManagement()
                            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                        .and()
                            .httpBasic()
                        .and()
                            .csrf()
                            .disable();
            }

            @Override
            protected void configure(AuthenticationManagerBuilder auth)
                    throws Exception {
                auth.inMemoryAuthentication()
                            .passwordEncoder(passwordEncoder())
                            .withUser("admin").password("test123").authorities("ROLE_ADMIN")
                            .and()
                                .withUser("test").password("test123").authorities("ROLE_USER");
            }


            @Bean
            @Override
            public AuthenticationManager authenticationManagerBean() throws Exception {
                return super.authenticationManagerBean();
            }

            @Bean
            @Override
            public UserDetailsService userDetailsServiceBean() throws Exception {
                return super.userDetailsServiceBean();
            }
        }

    `AuthenticationManagerBuilder` is the simplest entry to configure the essential security requirements. *InMemory* authentication is frequently used for demonstration. In a real world project, it is better to implement a `UserDetailsService` to load users from database. 

    Get the [codes](https://github.com/hantsy/angularjs-springmvc-sample) from my github account to explore all configuration classes.

