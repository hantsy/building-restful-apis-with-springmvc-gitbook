#Configure Spring Security

Spring Security provides a specific `WebApplicationInitializer` to initialize Spring Security facilities.

```java
@Order(1)
public class SecurityInitializer extends AbstractSecurityWebApplicationInitializer {

}    
```

Similiar with `AbstractAnnotationConfigDispatcherServletInitializer`, it is a `WebApplicationInitializer` implementation, and aleady configured Spring Security filter chain for you.
  
```java  
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
```

`AuthenticationManagerBuilder` is the simplest entry to configure the essential security requirements. *InMemory* authentication is frequently used for demonstration or test purpose. In a real world project, it is better to implement a `UserDetailsService` to load users from database. 
