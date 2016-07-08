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

A `webEnvironment` property in the `@SpringBootTest` is use for deciding if set up a web environment for test.

There are some configuration options of the `webEnvironment`.

* **MOCK** is the default, provides a mock web environment.
* **NONE** does not give a web environment.
* **DEFINED_PORT** provides an embedded web environment and run the application on a defined port.
* **RANDOM_PORT** provides an embedded web environment, but use a random port number.

If **RANDOM_PORT** is used, add `@LocalSeverPort` annotation to an `int` field will inject the port number at runtime.

	@LocalSeverPort
	int port;
	
Which replaced the `@Value("${local.server.port}")`	in Spring Boot 1.3.

Similarly, **classes** property is similar to the one of `@SpringApplicationConfiguration`. You can specify the configuration classes to be loaded for the test.

	@SpringBootTest(classes = {Application.class, SwaggerConfig.class})

Which is equivalent to `@SpringApplicationConfiguration(classes={...})` in Spring Boot 1.3.

### SpringRunner

Spring Boot 1.4 introduced a new JUnit Runner, `SpringRunner`, which is an alias for the `SpringJUnit4ClassRunner`.

	@RunWith(SpringRunner.class)

If you have to use other runner instead of SpringRunner, and want to use the Spring test context in the tests, use `SpringClassRule` and `SpringMethodRule` to fill the gap.

	@RunWith(AnotherRunner.class)
	public class SomeTest{

		@ClassRule
		public static final SpringClassRule SPRING_CLASS_RULE = new SpringClassRule();
		
		@Rule
		public final SpringMethodRule springMethodRule = new SpringMethodRule();
 
	}
	

### Test slice

The most exciting feature provided in Spring Boot 1.4 is it provides capability to test some feature slice, which just pick up essential beans and configuration for the specific purpose based test. 

Currently there is a series of new annotations available for this purpose.

**@JsonTest** provides a simple Jackson environment to test the json serialization and deserialization.

**@WebMvcTest** provides a mock web environment, it can specify the controller class for test and inject the `MockMvc` in the test.

	@WebMvcTest(PostController.class)
    public class PostControllerMvcTest{
	
		@Inject MockMvc mockMvc;
		
	}
	
**DataJpaTest** will prepare an embedded database and provides basic JPA environment for the test.	
		
**@RestClientTest** provides REST client environment for the test, esp the RestTemplateBuilder etc.

###JsonComponent

`@JsonComponent` is a specific `@Component` to register custome Jackson based `JsonSerializer` and `JsonDeserializer`. 

For example, a custom `JsonSerializer` and `JsonDeserializer` are registered for `LocalDateTime` class.

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

If you are using the default config, it will be activated by default when the application.

But if you have a custom `ObjectMapper` configuration, you have to install `JsonComponentModule` manually, else the `@JsonComponent` annotated bean will not be discovered.

    @Bean
    public Jackson2ObjectMapperBuilder objectMapperBuilder(JsonComponentModule jsonComponentModule) {
   
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
		//...
                .modulesToInstall(jsonComponentModule);
    
        return builder;
    } 

##MockBean and MockSpy

Spring Boot 1.4 integrates Mockito tightly, and provides Spring specific `@MockBean` and `MockSpy` annotations.
	
	@RunWith(SpringRunner.class)
	public class MockBeanTest {
		
		@MockBean
		private UserRepository userRepository;
		
	}
	
##Spring 4.3

The effort of [Spring Composed](https://github.com/sbrannen/spring-composed) are merged into Spring 4.3. 

A series of new composed annotations are available.

For example, a RestController can be simplfied by the new annotations, list as the following table.

|Spring 4.2| Spring 4.3|
|----|----|
|@RequestMapping(value = "", method = RequestMethod.GET)|@GetMapping()|
 @ResponseBody
|@RequestMapping(value = "", method = RequestMethod.POST)|@PostMapping()|
 @ResponseBody
|@RequestMapping(value = "/{id}", method = RequestMethod.PUT)|@PutMapping(value = "/{id}")|
 @ResponseBody
|@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)|@DeleteMapping(value = "/{id}")|
 @ResponseBody
 
A new `@RestControllerAdvice()` is provided for exception handling, you can remove the `@ResponseBody` in the method.

	@ControllerAdvice()
	public class RestExceptionHandler {

		@ExceptionHandler(value = {SomeException.class})
		@ResponseBody
		public ResponseEntity<ResponseMessage> handleGenericException(SomeException ex, WebRequest request) {
		}
	}
	
Becomes:

	@RestControllerAdvice()
	public class RestExceptionHandler {

		@ExceptionHandler(value = {SomeException.class})
		public ResponseEntity<ResponseMessage> handleGenericException(SomeException ex, WebRequest request) {
		}
	}
	

 
##Spring Security 4.1

##Hibernate 5.2

One of the big change is the packages have been reorganised.

***hibernate-java8**(Java 8 DateTime support) and **hibernate-entitymanager**(JPA provider bridge) are merged into **hibernate-core**.

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

**NOTE**: Hibernate 5.2.0.Final will break some dependencis before Spring 4.3, such as spring-orm, spring-boot-data-jpa-starter which depends on **hibernate-entitymanager**, and Spring Boot 1.4.0.RC1 and Spring 4.3 GA fix the issues. But I noticed in the Hibernate 5.2.1.Final, the **hibernate-entitymanager** is back.

Hibernate 5.2 also added Java Stream APIs support, I hope it will be available in the next JPA specification.

