# Upgrade to Spring Boot 1.4

Spring Boot 1.4 is a big jump, and introduced lots of new test facilities and aligned with the new technology stack, such as Spring framework 4.3 and Hibernate 5.2 and Spring Security 4.1, etc.

##Spring Boot 1.4


### New test purpose starter

Spring Boot 1.4 brings a new starter for test scope, named `spring-boot-starter-test`.

Use the following:

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
	
Instead of:

	<dependency>
		<groupId>com.jayway.jsonpath</groupId>
		<artifactId>json-path</artifactId>
		<scope>test</scope>
	</dependency>

	<dependency>
		<groupId>org.assertj</groupId>
		<artifactId>assertj-core</artifactId>
		<scope>test</scope>
	</dependency>

	<dependency>
		<groupId>org.hamcrest</groupId>
		<artifactId>hamcrest-core</artifactId>
		<scope>test</scope>
	</dependency>

	<dependency>
		<groupId>org.mockito</groupId>
		<artifactId>mockito-core</artifactId>
		<scope>test</scope>
	</dependency>

`spring-boot-starter-test` includes the essential dependencies for test, such as json-path, assertj, hamcrest, mockito etc.
	
###@SpringBootTest

Spring Boot 1.4 introduced a new annotation `@SpringBootTest` to unite the old `@IntegrationTest`, `@WebIntegrationTest`, `@SpringApplicationConfiguration` etc, in before versions.

A `webEnvironment` property of `@SpringBootTest` is use for deciding if set up a web environment for test.

There are some configuration options of the `webEnvironment`.

* **MOCK** is the default, provides a mock web environment.
* **NONE** does not give a web environment.
* **DEFINED_PORT** provides an embedded web environment and run the application on a defined port.
* **RANDOM_PORT** provides an embedded web environment, but use a random port number.

If **RANDOM_PORT** is used, add `@LocalSeverPort` annotation on an `int` field will inject the port number at runtime.

	@LocalSeverPort
	int port;
	
`@LocalServerPort` replaces the `@Value("${local.server.port}")` of Spring Boot 1.3.

Similarly, **classes** property is similar to the one of `@SpringApplicationConfiguration`. You can specify the configuration classes to be loaded for the test.

	@SpringBootTest(classes = {Application.class, SwaggerConfig.class})

The above code is equivalent to `@SpringApplicationConfiguration(classes={...})` in Spring Boot 1.3.

### SpringRunner

Spring Boot 1.4 introduced a new JUnit Runner, `SpringRunner`, which is an alias for the `SpringJUnit4ClassRunner`.

	@RunWith(SpringRunner.class)

If you have to use other runners instead of `SpringRunner`, and want to use the Spring test context in the tests, declare a `SpringClassRule` and `SpringMethodRule` in the test to fill the gap.

	@RunWith(AnotherRunner.class)
	public class SomeTest{

		@ClassRule
		public static final SpringClassRule SPRING_CLASS_RULE = new SpringClassRule();
		
		@Rule
		public final SpringMethodRule springMethodRule = new SpringMethodRule();
 
	}
	

### Autoconfigure test slice

The most exciting feature provided in Spring Boot 1.4 is it provides capability to test some feature slice, which just pick up essential beans and configuration for the specific purpose based test. 

Currently there is a series of new annotations available for this purpose.

**@JsonTest** provides a simple Jackson environment to test the json serialization and deserialization.

**@WebMvcTest** provides a mock web environment, it can specify the controller class for test and inject the `MockMvc` in the test.

	@WebMvcTest(PostController.class)
    public class PostControllerMvcTest{
	
		@Inject MockMvc mockMvc;
		
	}
	
**@DataJpaTest** will prepare an embedded database and provides basic JPA environment for the test.	
		
**@RestClientTest** provides REST client environment for the test, esp the RestTemplateBuilder etc.

These annotations are not composed with `SpringBootTest`, they are combined with a series of `AutoconfigureXXX` and a `@TypeExcludesFilter` annotations.

Have a look at `@DataJpaTest`.

	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Inherited
	@BootstrapWith(SpringBootTestContextBootstrapper.class)
	@OverrideAutoConfiguration(enabled = false)
	@TypeExcludeFilters(DataJpaTypeExcludeFilter.class)
	@Transactional
	@AutoConfigureCache
	@AutoConfigureDataJpa
	@AutoConfigureTestDatabase
	@AutoConfigureTestEntityManager
	@ImportAutoConfiguration
	public @interface DataJpaTest {}

You can add your `@AutoconfigureXXX` annotation to override the default config.

	@AutoConfigureTestDatabase(replace=NONE)
	@DataJpaTest
	public class TestClass{
	}

###JsonComponent

`@JsonComponent` is a specific `@Component` to register custome Jackson `JsonSerializer` and `JsonDeserializer`. 

For example, custom `JsonSerializer` and `JsonDeserializer` are use for serializing and deserializing `LocalDateTime` instance.

	@JsonComponent
	@Slf4j
	public class LocalDateTimeJsonComponent {

		public static class LocalDateTimeSerializer extends JsonSerializer<LocalDateTime> {

			@Override
			public void serialize(LocalDateTime value, JsonGenerator jgen, SerializerProvider provider) throws IOException {
				jgen.writeString(value.atZone(ZoneId.systemDefault()).toInstant().toString());
			}
		}

		public static class LocalDateTimeDeserializer extends JsonDeserializer<LocalDateTime> {

			@Override
			public LocalDateTime deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
				ObjectCodec codec = p.getCodec();
				JsonNode tree = codec.readTree(p);
				String dateTimeAsString = tree.textValue();
				log.debug("dateTimeString value @" + dateTimeAsString);
				return LocalDateTime.ofInstant(Instant.parse(dateTimeAsString), ZoneId.systemDefault());
			}
		}
	}

If you are using the Spring Boot default Jackson configuration, it will be activated by default when the application starts up.

But if you customized a `ObjectMapper` bean in your configuration, the autoconfiguration of `ObjectMapper` is disabled. You have to install `JsonComponentModule` manually, else the `@JsonComponent` beans will not be scanned at all.

    @Bean
    public Jackson2ObjectMapperBuilder objectMapperBuilder(JsonComponentModule jsonComponentModule) {
   
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
		//...
                .modulesToInstall(jsonComponentModule);
    
        return builder;
    } 

###Mocking and spying Beans

Spring Boot 1.4 integrates Mockito tightly, and provides Spring specific `@MockBean` and `@MockSpy` annotations.
	
	@RunWith(SpringRunner.class)
	public class MockBeanTest {
		
		@MockBean
		private UserRepository userRepository;
		
	}
	
	
	
###TestConfiguration and TestComponent

`TestConfiguration` and `TestComponent` are designated for test purpose, they are similar with `Configuration` and `Component`. Generic `Configuration` and `Component` can not be scanned by default in test.

	public class TestClass{
	
		@TestConfiguration
		static class TestConfig{
		}
		
		@TestComponent
		static class TestBean{}
	
	}

	
##Spring 4.3

There are a few features added in 4.3, the following is impressive.

###Composed annotations

The effort of [Spring Composed](https://github.com/sbrannen/spring-composed) are merged into Spring 4.3. 

A series of new composed annotations are available, but the naming is a little different from Spring Composed.

For example, a RestController can be simplfied by the new annotations, list as the following table.

|Spring 4.2| Spring 4.3|
|----|----|
|@RequestMapping(value = "", method = RequestMethod.GET)|@GetMapping()|
|@RequestMapping(value = "", method = RequestMethod.POST)|@PostMapping()|
|@RequestMapping(value = "/{id}", method = RequestMethod.PUT)|@PutMapping(value = "/{id}")|
|@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)|@DeleteMapping(value = "/{id}")|

A new `@RestControllerAdvice()` is provided for exception handling, it is combination of `@ControllerAdvice` and `@ResponseBody`. You can remove the `@ResponseBody` on the `@ExceptionHandler` method when use this new annotation.

For example, in the old Spring 4.2, an custom exception handler class looks like the following.

	@ControllerAdvice()
	public class RestExceptionHandler {

		@ExceptionHandler(value = {SomeException.class})
		@ResponseBody
		public ResponseEntity<ResponseMessage> handleGenericException(SomeException ex, WebRequest request) {
		}
	}
	
In Spring 4.3, it becomes:

	@RestControllerAdvice()
	public class RestExceptionHandler {

		@ExceptionHandler(value = {SomeException.class})
		public ResponseEntity<ResponseMessage> handleGenericException(SomeException ex, WebRequest request) {
		}
	}
	
###Auto constructor injection

If there is a only one constructor defined in the bean, the arguments as dependencies will be injected by default.	

Before 4.3, you have to add `@Inject` or `@Autowired` on the constructor to inject the dependencies.

	@RestController
	@RequestMapping(value = Constants.URI_API_PREFIX + Constants.URI_POSTS)
	public class PostController {

		@Inject
		public PostController(BlogService blogService) {
			this.blogService = blogService;
		}
	}

`@Inject` can be removed in Spring 4.3.

	@RestController
	@RequestMapping(value = Constants.URI_API_PREFIX + Constants.URI_POSTS)
	public class PostController {

		public PostController(BlogService blogService) {
			this.blogService = blogService;
		}
	}

	
##Spring Security 4.1

The Java configuration is improved. 

Before 4.1, you can configure `passwordEncoder` and `userDetailsService` via `AuthenticationManagerBuilder`.

	@Configuration
	@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
	protected static class ApplicationSecurity extends WebSecurityConfigurerAdapter {

	  @Override
	  protected void configure(HttpSecurity http) throws Exception {}

	  @Override
	  protected void configure(AuthenticationManagerBuilder auth)
			  throws Exception {
		  auth
			  .userDetailsService(new SimpleUserDetailsServiceImpl(userRepository))
			  .passwordEncoder(passwordEncoder);
	  }

	  @Bean
	  @Override
	  public AuthenticationManager authenticationManagerBean() throws Exception {
		  return super.authenticationManagerBean();
	  }
	  
	}

In 4.1, `userDetailsService` and `passwordEncoder` bean can be detected automaticially. No need to wire them by `AuthenticationManagerBuilder` manually. No need to override the `WebSecurityConfigurerAdapter` class and provide a custom configuration, a generic `WebSecurityConfigurerAdapter` bean is enough.

	@Bean
    public BCryptPasswordEncoder passwordEncoder() {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        return passwordEncoder;
    }
    
    @Bean
    public UserDetailsService userDetailsService(UserRepository userRepository){
        return new SimpleUserDetailsServiceImpl(userRepository);
    }
    
    @Bean
    public WebSecurityConfigurerAdapter securityConfig(){
        return new WebSecurityConfigurerAdapter() {
            @Override
            protected void configure(HttpSecurity http) throws Exception {//...}
    }	



More details can be found in the [What’s New in Spring Security 4.1](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#new) chapter of Spring Secuirty documentation.

##Hibernate 5.2

The biggest change of Hibernate 5.2 is the packages had been reorganised, Hibernate 5.2 is Java 8 ready now.

**hibernate-java8** (Java 8 DateTime support) and **hibernate-entitymanager** (JPA provider bridge) are merged into **hibernate-core**.

Remove the following dependencies when upgrade to Hibernate 5.2.

	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-java8</artifactId>
		<version>${hibernate.version}</version>
	</dependency>
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-entitymanager</artifactId>
		<version>${hibernate.version}</version>
	</dependency>

**NOTE**:If you are using Spring 4.2 with Hibernate 5.2.0.Final, it could break some dependencis, such as `spring-orm`, `spring-boot-data-jpa-starter` which depends on **hibernate-entitymanager**. Spring Boot 1.4.0.RC1 and Spring 4.3 GA fixed the issues. But I noticed in the Hibernate 5.2.1.Final,  **hibernate-entitymanager** is back.

Hibernate 5.2 also added Java Stream APIs support, I hope it will be available in the next JPA specification.

##Source code

Clone the codes from Github account.

	git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
	
	