# Configure security 

By default, Spring Boot will add BASIC authentication for your application. You can set the username and password in `application.yml` directly.

In a real world application, we would use DataSource driven configuration which you can use database to store user info.

Slightly changes the security configuration. Add a custom `WebSecurityConfigurerAdapter` bean is enough.

```
/**
 *
 * @author hantsy
 */
@Configuration
public class SecurityConfig {
    
    @Bean
    public WebSecurityConfigurerAdapter webSecurityConfigure(){
        return new WebSecurityConfigurerAdapter() {
            
            @Override
            protected void configure(HttpSecurity http) throws Exception {
            // @formatter:off
                http
                    .authorizeRequests()
                    .antMatchers("/api/signup", "/api/users/username-check")
                    .permitAll()
                    .and()
                        .authorizeRequests()
                        .regexMatchers(HttpMethod.GET, "^/api/users/[\\d]*(\\/)?$").authenticated()
                        .regexMatchers(HttpMethod.GET, "^/api/users(\\/)?(\\?.+)?$").hasRole("ADMIN")
                        .regexMatchers(HttpMethod.DELETE, "^/api/users/[\\d]*(\\/)?$").hasRole("ADMIN")
                        .regexMatchers(HttpMethod.POST, "^/api/users(\\/)?$").hasRole("ADMIN")
                    .and()
                        .authorizeRequests()
                        .antMatchers("/api/**").authenticated()
                    .and()
                        .authorizeRequests()
                        .anyRequest().permitAll()
                    .and()
                        .sessionManagement()
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                    .and()
                        .httpBasic()
                    .and()
                        .csrf()
                        .disable();
            // @formatter:on
            }
        };
    }
}
```

To customize security, you could have to define your own `UserDetails` and `UserDetailsService`.

```java
@Entity
@Table(name = "users")
public class User implements UserDetails, Serializable {

}

```

Create a JPA entity to emplement the `UserDetails` interface.

```java
@Component
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
```

Define a `UserDetailsService`, which can be detected by the newest Spring Security, there is no need to wire the `UserDetailsService` with `AuthenticationManager` in configuration file. Check the [Upgrade to Spring Boot 1.4](../boot-1.4.md) for more details.


