# Configure Spring MVC 

Firstly add Spring WebMvc related dependenices into *pom.xml*.

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
</dependency>
```

To bootstrap a Spring MVC application, you have to enable Spring built-in `DisptachServlet`.

Serlvet 3.0 provides a new feature _ServletInitializer_ to configure web applciation without _web.xml_.

Spring has its _WebApplicationInitializer_ interface, there are a few classes implement this interface, `AbstractAnnotationConfigDispatcherServletInitializer` includes configuration of Spring Dispatch Servlet, and leaves some room to customize `DispatchServlet`.

Declare a `AbstractAnnotationConfigDispatcherServletInitializer` bean.

```java
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

`getRootConfigClasses` specifies the configuration classes should be loaded for the Spring infrastrucuture.

`getServletConfigClasses` specifies the configurations depend on Servlet specification, esp, web mvc related configurations.

`getServletMappings` is the Spring `DispatchServlet` mapping URL pattern.

`getServletFilters` are those Web Filters will be applied on the Spring `DispatchServlet`.

Spring MVC `DispatchServlet` is configured in the super classes, explore the details if you are interested in it.
 
In `getServletConfigClasses`, we specify a `WebConfig` will be loaded, which is responsible for configuring Spring MVC in details, including resource handling, view, view resolvers etc.

A classic `WebConfig` looks like.

```java
public class WebConfig extends WebMvcConfigurerAdapter {

}
```

Generally, `WebMvcConfigurerAdapter` is the extension point left for users to use for customizing MVC configurations. 

Spring Data project provides a `SpringDataWebConfiguration` which is a subclass of `WebMvcConfigurerAdapter` and adds pagination, sort, and domain object conversion support. Open the source code of `SpringDataWebConfiguration` and research yourself.

The following is the full source code of `WebConfig`.

```java
@Configuration
@EnableWebMvc
@ComponentScan(
        basePackageClasses = {Constants.class},
        useDefaultFilters = false,
        includeFilters = {
            @Filter(
                    type = FilterType.ANNOTATION,
                    value = {
                        Controller.class,
                        RestController.class,
                        ControllerAdvice.class
                    }
            )
        }
)
public class WebConfig extends SpringDataWebConfiguration {

    private static final Logger logger = LoggerFactory.getLogger(WebConfig.class);

    @Inject
    private ObjectMapper objectMapper;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

        registry.addResourceHandler("/swagger-ui.html")
                .addResourceLocations("classpath:META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:META-INF/resources/webjars/");
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {

    }

    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers) {
        exceptionResolvers.add(exceptionHandlerExceptionResolver());
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorParameter(false);
        configurer.favorPathExtension(false);
    }

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        List<HttpMessageConverter<?>> messageConverters = messageConverters();
        converters.addAll(messageConverters);
    }

    @Bean
    public ExceptionHandlerExceptionResolver exceptionHandlerExceptionResolver() {
        ExceptionHandlerExceptionResolver exceptionHandlerExceptionResolver = new ExceptionHandlerExceptionResolver();
        exceptionHandlerExceptionResolver.setMessageConverters(messageConverters());

        return exceptionHandlerExceptionResolver;
    }

    private List<HttpMessageConverter<?>> messageConverters() {
        List<HttpMessageConverter<?>> messageConverters = new ArrayList<>();

        MappingJackson2HttpMessageConverter jackson2Converter = new MappingJackson2HttpMessageConverter();
        jackson2Converter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON));
        jackson2Converter.setObjectMapper(objectMapper);

        messageConverters.add(jackson2Converter);
        return messageConverters;
    }

}
```

* `@EnableWebMvc` tells Spring to enable Spring MVC.
* `@ComponentScan` uses a filter to select Spring MVC related classes to be activated.
* `addResourceHandlers` configure how to handle static resources.
* `addViewControllers` leave places to configure view resolver to render specific views.
* `configureHandlerExceptionResolvers` specifies exception handling stretagy.
* `configureMessageConverters` configure `HttpMessageConverter` will be used for serialization and deserialization.

I would like use `application/json` as default content type, and uses Jackson to serialize and deserialize messages.

Configure a Jackson ObjectMapper as you need.

```java
@Configuration
public class Jackson2ObjectMapperConfig {

    @Bean
    public ObjectMapper objectMapper() {

        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
        builder.serializationInclusion(Include.NON_EMPTY);
        builder.featuresToDisable(
               // SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,
                DeserializationFeature.FAIL_ON_IGNORED_PROPERTIES,
                DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        builder.featuresToEnable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY);

        return builder.build();
    }
}
```

`Jackson2ObjectMapperBuilder` provides fluent APIs to configure which features will be enabled or disabled for serialization and deserialization. For the complete configurable options, check the javadocs of `SerializationFeature` and `DeserializationFeature`.



