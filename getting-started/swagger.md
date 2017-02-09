#Configure Swagger

Utilizes with Swagger UI, we can visualize API, allow developers to test APIs on fly.

[SpringFox project](http://springfox.github.io/springfox/) provides Swagger support for Spring based REST APIs.

1. Add springfox to dependencies.

		<!-- SpringFox Swagger UI -->
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>${springfox.version}</version>
		</dependency>
			
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>${springfox.version}</version>
		</dependency>
		
	`springfox-swagger-ui` provides static Javascript UI for visualizing the Swagger schema definitions.
	
2. Add a `@Configuration` class to enable Swagger.

		@Configuration
		@EnableSwagger2
		public class SwaggerConfig {

			@Bean
			public Docket postsApi() {
				return new Docket(DocumentationType.SWAGGER_2)
						.groupName("public-api")
						.apiInfo(apiInfo())
						.select()
						.paths(postPaths())
						.build();
			}

			private Predicate<String> postPaths() {
				return or(
						regex("/api/posts.*"),
						regex("/api/comments.*")
				);
			}

			private ApiInfo apiInfo() {
				return new ApiInfoBuilder()
						.title("SpringMVC Example API")
						.description("SpringMVC Example API reference for developers")
						.termsOfServiceUrl("http://hantsy.blogspot.com")
						.contact("Hantsy Bai")
						.license("Apache License Version 2.0")
						.licenseUrl("https://github.com/springfox/springfox/blob/master/LICENSE")
						.version("2.0")
						.build();
			}

		}

	When the application starts up, it will scan all Controllers and generate Swagger schema definition at runtime, Swagger UI will read definitions and render user friendly UI for REST APIs.
	
3. View REST APIs in swagger ui.

	Starts up this application via command line.
	
		mvn tomcat7:run 
		
	Open browser and navigate [http://localhost:8080/angularjs-springmvc-sample/swagger-ui.html](http://localhost:8080/angularjs-springmvc-sample/swagger-ui.html).	
	