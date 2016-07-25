#Secures APIs

We have configured Spring Security in before posts. 

In this post, I will show you using Spring Security to protect APIs, aka provides Anthentication and Anthorization service for this sample application.

* **Authentication** answers the question: if the user is a valid user.
* **Authorization** resolves the problem: if the authenticated user has corresponding permissions to access resources.

##Authentication

In Spring security, it is easy to configure JAAS compatible authentication strategy, such as FORM, BASIC, X509 Certiciate etc. 

Unlike JAAS in which the authentication management is very dependent on the container itself. Spring Security provides some extension points(such as `UserDetails`, `UserDetailsService`, `Authority`) and allows developers to customize and implement the authentication and authorization in a progammatic approach. 

Motioned in before posts, the simplest way to configure Spring security is using `AuthenticationManagerBuilder` to build essential required resources. 

	@Override
	protected void configure(AuthenticationManagerBuilder auth)
			throws Exception {
		auth.inMemoryAuthentication()
				.passwordEncoder(passwordEncoder())
				.withUser("admin").password("test123").authorities("ROLE_ADMIN")
				.and()
					.withUser("test").password("test123").authorities("ROLE_USER");
	}
	
An in-memory database and a HTTP BASIC authentication is easy to prototype applications, as showing as above codes. 

If you want to store users into your database, firstly create a custom `UserDetailsService` bean and implement the `findByUsername` method and return a `UserDetails` object.

	public class SimpleUserDetailsServiceImpl implements UserDetailsService {

		private static final Logger log = LoggerFactory.getLogger(SimpleUserDetailsServiceImpl.class);

		private UserRepository userRepository;

		public SimpleUserDetailsServiceImpl(UserRepository userRepository) {
			this.userRepository = userRepository;
		}

		@Override
		public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
			User user = userRepository.findByUsername(username);
			if (user == null) {
				throw new UsernameNotFoundException("username not found:" + username);
			}

			log.debug("found by username @" + username);

			return user;

		}

	}
	
`User` class implements `UserDetails`.

	@Data
	@Builder
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Table(name = "users")
	public class User implements UserDetails, Serializable {

		/**
		 *
		 */
		private static final long serialVersionUID = 1L;

		@Id()
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		@Column(name = "id")
		private Long id;

		@Column(name = "username")
		private String username;

		@Column(name = "password")
		private String password;

		@Column(name = "name")
		private String name;

		@Column(name = "email")
		private String email;

		@Column(name = "role")
		private String role;

		@Column(name = "created_date")
		@CreatedDate
		private LocalDateTime createdDate;

		public String getName() {
			if (this.name == null || this.name.trim().length() == 0) {
				return this.username;
			}
			return name;
		}

		@Override
		public Collection<? extends GrantedAuthority> getAuthorities() {
			return Arrays.asList(new SimpleGrantedAuthority("ROLE_" + this.role));
		}

		@Override
		public String getPassword() {
			return this.password;
		}

		@Override
		public String getUsername() {
			return this.username;
		}

		@Override
		public boolean isAccountNonExpired() {
			return true;
		}

		@Override
		public boolean isAccountNonLocked() {
			return true;
		}

		@Override
		public boolean isCredentialsNonExpired() {
			return true;
		}

		@Override
		public boolean isEnabled() {
			return true;
		}

	}

Then configure `AuthenticationManager` with custom `UserDetailsService` instead of `inMemoryAuthentication`.	
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth)
			throws Exception {
		auth.userDetailsService(new SimpleUserDetailsServiceImpl(userRepository))
			.passwordEncoder(passwordEncoder);
	}
	
If you want to design a customized Authentication strategy, you could have to create a custom `AuthenticationEntryPoint` and `AuthenticationProvider` for it. We will discuss this later.

##Anthorization

Once user is authenticated, when he tries to access some resources, such as URL, or execute some methods, it should check if the resource is protected, or has granted permissions on executing the methods.

### Declarative URL pattern based authorizations

For REST APIs, the API resources are identified by URI, it is easy to grant authorizations via URL.

Override the `configure(HttpSecurity)` of `WebSecurityConfigurerAdapter`.

	public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http   
				.authorizeRequests()   
				.antMatchers("/api/ping")
				.permitAll()
			.and()
				.authorizeRequests()   
				.antMatchers("/api/**")
				.authenticated()
				//....
		}

The access control is filter by the `Matcher`, there are two built-in matchers, Apache Ant path matcher, and perl like regex matchers. The later is a little complex, but more powerful. 	

`http...antMatchers("/api/**").authenticated()` means all resource URLs match '/api/**' need a valid authentication.

`http...antMatchers(HttpMethod.POST, "/api/posts").hasRoles("ADMIN")` indicates only users that have been granted **ADMIN** role have permission to create a new post. 

Combined with resource URLs and HTTP methods, it follows rest convention exactly.

In the a real world application, you can centralize the *URL pattern*, *HTTP Method*, and granted *ROLES* into a certain persistent storage(such as RDBMS or NOSQL) and desgin a friendly web UI to control resource access. 
		
### Method level authorizations 

Like JAAS, Spring Security provides several annotations to authorize access on method level.

Firstly you should add `@EnableGlobalMethodSecurity` on `@Configuration` class to enable it.

	
	@Configuration
	@EnableWebSecurity
	@EnableGlobalMethodSecurity(prePostEnabled = true, jsr250Enabled = true, securedEnabled = true)
	public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
	}

If **prePostEnabled** is true, the `@PreAuthorized` and `@PostAuthorized` can be used, it accept Spring EL string and evaluate the result.

For example, only *ADMIN* user can save post.
	
	@PreAuthorized("hasRole('ADMIN')")
	public void savePost(Post post){}
	
And only the post owner can update the post.

	@PreAuthorized("#post.author.id==principal.id")
    public void update(Post post){}
	
**jsr250Enabled** provides Java common anntotation compatibility, and allow you use JAAS annotations in Spring project.
 	
**securedEnabled** enables the legacy `@Secured` annotation which does not accept Spring EL as property value.

	@Secured("ROLE_USER")	
	public void savePost(Post post){}
	
### Programmatic authorizations

Spring provides APIs to fetch current principal info. 

For example, get current Authetication from SecurityContextHolder.

	Authentication auth = SecurityContextHolder.getContext().getAuthentication();

And you can also inject current authenticated Principal like this.

	publc List<Post> getByCurrentUser(@AuthenticationPrincipal Principal principal){}

After got the security principal info, you can control the authorizations in codes.

#Handles Exceptions

In the real world applications, a user story can be described as different flows.

1. The execution works as expected and get success finally. 
2. Some conditions are not satisfied and the flow should be intercepted and notify users.

For example, when a user tries to register an account in this application. The server side should check if the existance of the input username, if the username is taken by other users, the server should stop the registration flow and wraps the message into `UsernameWasTakenException` and throws it. Later the APIs should translate it to client friend message and reasonable HTTP status code, and finally they are sent to client and notify the user.

##Define Exceptions

Define an exeption which stands for the exception path. For example, `ResourceNotFoundException` indicates an resource is not found in the applicatin when query the resource by id.

	public class ResourceNotFoundException extends RuntimeException {

		private static final long serialVersionUID = 1L;

		private final Long id;

		public ResourceNotFoundException(Long id) {
			this.id = id;
		}

		public Long getId() {
			return id;
		}
	}

##Throws exceptions in service

In our service, check the resource existance, and throws the `ResourceNotFoundException` when the resource is not found.	

    public PostDetails findPostById(Long id) {

        Assert.notNull(id, "post id can not be null");

        log.debug("find post by id@" + id);

        Post post = postRepository.findOne(id);

        if (post == null) {
            throw new ResourceNotFoundException(id);
        }

        return DTOUtils.map(post, PostDetails.class);
    }
	
##Translates exceptions

In the web APIs, the exception can be caught and converted into user friendly message.

Spring provides a built-in `ResponseEntityExceptionHandler` to process the exception and tranlate them into REST API friendly messages.

You can extend this class and override the exeption hanlder methods, or add your exception hanlder.

	@ControllerAdvice(annotations = RestController.class)
	public class RestExceptionHandler extends ResponseEntityExceptionHandler {

		private static final Logger log = LoggerFactory.getLogger(RestExceptionHandler.class);

		@ExceptionHandler(value = {ResourceNotFoundException.class})
		@ResponseBody
		public ResponseEntity<ResponseMessage> handleResourceNotFoundException(ResourceNotFoundException ex, WebRequest request) {
			if (log.isDebugEnabled()) {
				log.debug("handling ResourceNotFoundException...");
			}
			return new ResponseEntity<>(HttpStatus.NOT_FOUND);
		}
	
In the above code, `ResourceNotFoundException` is handled by the `RestExceptionHandler`, and send a 404 HTTP status code to the client.

##Handles bean validation failure

For user input form validation, most of time, we could needs the detailed info the validaiton cosntraints. 

Spring supports JSR 303(Bean validation) natively, the bean validation constraints error can be gathered by `BindingResult` in the controller class.

    public ResponseEntity<ResponseMessage> createPost(@RequestBody @Valid PostForm post, BindingResult errResult) {

        log.debug("create a new post");
        if (errResult.hasErrors()) {
            throw new InvalidRequestException(errRusult);
        }
		//...
		
In the controller class, if the `BindingResult` has errors, then wraps the error info into an exception.

Handles it in the `RestExceptionHandler`.

	@ExceptionHandler(value = {InvalidRequestException.class})
    public ResponseEntity<ResponseMessage> handleInvalidRequestException(InvalidRequestException ex, WebRequest req) {
        if (log.isDebugEnabled()) {
            log.debug("handling InvalidRequestException...");
        }

        ResponseMessage alert = new ResponseMessage(
            ResponseMessage.Type.danger,
            ApiErrors.INVALID_REQUEST,
            messageSource.getMessage(ApiErrors.INVALID_REQUEST, new String[]{}, null));

        BindingResult result = ex.getErrors();

        List<FieldError> fieldErrors = result.getFieldErrors();

        if (!fieldErrors.isEmpty()) {
            fieldErrors.stream().forEach(e -> {
                alert.addError(e.getField(), e.getCode(), e.getDefaultMessage());
            });
        }

        return new ResponseEntity<>(alert, HttpStatus.UNPROCESSABLE_ENTITY);
    }	
		
The detailed validation errors are wrapped as content and sent to the client, and a HTTP status to indicate the form data user entered is invalid.

##Exception and HTTP status

All Spring built-in exceptions have been handled and mapped to a HTTP status code. Read the `ResponseEntityExceptionHandler` code or javadoc for more details.

All business related exceptions should designed and converted to a valid HTTP status code and essential messages.

Some HTTP status codes are used frequently.

* 200 OK
* 201 Created
* 204 NO Content
* 400 Bad Resquest
* 401 Not Authoried
* 403 Forbidden
* 409 Conflict

More details about HTTP status code, please read [https://httpstatuses.com/](https://httpstatuses.com/) or W3C [HTTP Status definition](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

##Source Code

Check out sample codes from my github account.

	git clone https://github.com/hantsy/angularjs-springmvc-sample
	
Or the Spring Boot version:

	git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
	
Read the live version of thess posts from Gitbook:[Building RESTful APIs with Spring MVC](https://www.gitbook.com/book/hantsy/build-a-restful-app-with-spring-mvc-and-angularjs/details).



	
	