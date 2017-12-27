# Secures APIs

We have configured Spring Security in before posts. 

In this post, I will show you using Spring Security to protect APIs, aka provides Authentication and Authorization service for this sample application.

* **Authentication** answers the question: if the user is a valid user.
* **Authorization** resolves the problem: if the authenticated user has corresponding permissions to access resources.

## Authentication

In Spring security, it is easy to configure JAAS compatible authentication strategy, such as FORM, BASIC, X509 Certificate etc. 

Unlike JAAS in which the authentication management is very dependent on the container itself. Spring Security provides some extension points(such as `UserDetails`, `UserDetailsService`, `Authority`) and allows developers to customize and implement the authentication and authorization in a programmatic approach. 

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

## Authorization

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

The access control is filter by the `Matcher`, there are two built-in matchers, Apache Ant path matcher, and Perl like regex matchers. The later is a little complex, but more powerful. 	

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

For example, get current Authentication from `SecurityContextHolder`.

	Authentication auth = SecurityContextHolder.getContext().getAuthentication();

And you can also inject current authenticated Principal like this.

	publc List<Post> getByCurrentUser(@AuthenticationPrincipal Principal principal){}

After got the security principal info, you can control the authorizations in codes.



## Source Code

Check out sample codes from my github account.

	git clone https://github.com/hantsy/angularjs-springmvc-sample
	
Or the Spring Boot version:

	git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
	
Read the live version of theses posts from Gitbook:[Building RESTful APIs with Spring MVC](https://www.gitbook.com/book/hantsy/build-a-restful-app-with-spring-mvc-and-angularjs/details).



	
	