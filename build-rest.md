#Building REST API

As stated in before posts, we are going to build a Blog system.

To demonstrate REST API, we use a simple `Post` entity to persist blog entries, and expose the CRUD operations via REST APIs to client applications. As a REST API consumer, the client applications could be a website, a desktop application, or a mobile application.

Following the REST API convention and HTTP protocol specification, the post APIs can be designed as the following table.

|Uri|Http Method|Request|Response|Description|
|---|---|---|---|---|
|/posts|GET||200, [{'id':1, 'title'},{}]|Get all posts|
|/posts|POST|{'title':'test title','content':'test content'}|201|Create a new post|
|/posts/{id}|GET||200, {'id':1, 'title'}|Get a post by id|
|/posts/{id}|PUT|{'title':'test title','content':'test content'}|204|Update a post|
|/posts/{id}|DELETE||204|Delete a post|

Next, we begin to create the domain models: `Post`.

## Modeling the blog application

As planned in [Overview](overview.md), there are some domain objects should be created for this blog sample application.

A `Post` model to store the blog entries posted by users.
A `Comment` model to store the comments on a certain post.
A `User` model to store users will user this blog application.

Every domain object should be identified. JPA entities satisfy this requirement. Every JPA entity has an `@Id` field as identifier. 

A simple `Post` entity can be designated as the following. Besides id, it includes a `title` field , a `content` field, and `createdDate` timestamp, etc.

	@Entity
	@Table(name = "posts")
	public class Post implements Serializable {

		@Id()
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		@Column(name = "id")
		private Long id;

		@Column(name = "title")
		private String title;

		@Column(name = "content")
		@Size(max = 2000)
		private String content;

		@Column(name = "created_date")
		@Temporal(TemporalType.TIMESTAMP)
		private Date createdDate;
		
		//getters and setters, hashcode, equals, toString etc. are omitted
	}
	
It is a standard JPA entity. 

* An entity class must be annotated with `@Entity`
* An entity should implement `Serializable` interface
* An entity must include an identifier field(annotated with `@Id`), such as `id` of `Post`.
* An entity should have a default none-arguments constructor. By default, if there is no explict constructor declaration, there is an implict none-argument constructor. If there is a constructor accepts more than one arguments, you have to add another none-argument explictly.

	For example.
	
		public Post(String title, String content){}
		public Post(){}//must add this constructor explictly

Optionally, it is recommended to implement your own `equals` and `hashCode` methods for every entities, if there is a requirement to identify them in a collection.

For example, there are a post existed in a collection, adding another `Post` into the same collection should check the post existance firstly.

	publc class PostCollection{
		private List<Post> posts=new ArrayList<>();
		
		public void addPost(Post p){
			if(!this.posts.contains(p)){
				this.posts.add(p);
			}
		}
	}
	
The title field can be used to identify two posts in a collection, because they are not presisted in a persistent storage at the moment, `id` value are same--null.

Implements `equals` and `hashCode` with title field of `Post`.

	public boolean equals(Object u){ 
		// omitted null check
		return this.title=(Post)u.getTitle();
	}
	
	public int hashCode(){
		return 17* title.hashCode();
	}

When an entity instance is being persisted into a database table, the id will be filled. 

In JPA specification, there is a sort of standard id generation strategies available.

By default, it is `AUTO`, which uses the database built-in id generation approache to assign an primary key to the inserted record. 

**WARNING**: Every databases has its specific generation strategy, if you are building an application which will run across databases. `AUTO` is recommended.

Other id generation strategies include *TABLE*, *IDENTITY*. And JPA providers have their extensions, such as with Hibernate, you can use *uuid2* for PostgresSQL. 	

###Lombok
	
[Lombok](http://projectlombok.org) is a greate helper every Java developer should use in projects. Utilize Java annotation processor, it can generate getters, setters, equals, hashCode, toString and class constructor at compile runtime with some Lombok annotations.

Add `@Data` to `Post` class, you can remove all getters and setters, and `equals`, `hashcode`, `toString` methods. The code now looks more clean.

	@Data
	@Builder
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Table(name = "posts")
	public class Post implements Serializable {		
		@Id()
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		@Column(name = "id")
		private Long id;

		@Column(name = "title")
		private String title;

		@Column(name = "content")
		@Size(max = 2000)
		private String content;

		@Column(name = "status")
		@Enumerated(value = EnumType.STRING)
		private Status status = Status.DRAFT;

		@Column(name = "created_date")
		@Temporal(TemporalType.TIMESTAMP)
		private Date createdDate;
	}

There are some annotations provided in Lombok to archive this purpose.

`@Data` is a composite annotation which includes @Getter, @Setter, @EqualsAndHashCode,@ToString etc.
`@Builder` will generate an inner Builder class in the hosted class which provides a fluent API to build an object.

Add lombok dependency in *pom.xml*.

	<!--Lombok-->
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>${lombok.version}</version>
	</dependency>

If there are several JAP(Java annotaition processor) exist in the project, such as JPA metadata generator, it is better to add Lombok processor to maven compiler plugin.

For example.

	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-compiler-plugin</artifactId>
		<version>3.5.1</version>
		<configuration>
			<compilerArgument>-Xlint</compilerArgument>
			<annotationProcessors>
				<annotationProcessor>lombok.launch.AnnotationProcessorHider$AnnotationProcessor</annotationProcessor>
				<annotationProcessor>org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor</annotationProcessor>
			</annotationProcessors>
		</configuration>
	</plugin>

**NOTE**: If you are using Eclipse based IDE, such as Spring Tool Suite, or Intellij IDEA, you could have to install the Lombok plugin manually, check the [Lombok download page](https://projectlombok.org/download.html) for installation information. Luckily, NetBeans IDE can recognise the Lombok facilities automatcially.

Unlike JPA metadata generator which generates metedata source for JPA entities. Lombok modifies target classes directly.

Execute `javap Post.class` in command line, you can get the follwing info.

	#>javap  classes\com\hantsylabs\restexample\springmvc\domain\Post.class
	Compiled from "Post.java"
	public class com.hantsylabs.restexample.springmvc.domain.Post implements java.io.Serializable {
	  public com.hantsylabs.restexample.springmvc.domain.Post(java.lang.String, java.lang.String);
	  public static com.hantsylabs.restexample.springmvc.domain.Post$PostBuilder builder();
	  public java.lang.Long getId();
	  public java.lang.String getTitle();
	  public java.lang.String getContent();
	  public com.hantsylabs.restexample.springmvc.domain.Post$Status getStatus();
	  public com.hantsylabs.restexample.springmvc.domain.User getCreatedBy();
	  public java.time.LocalDateTime getCreatedDate();
	  public com.hantsylabs.restexample.springmvc.domain.User getLastModifiedBy();
	  public java.time.LocalDateTime getLastModifiedDate();
	  public void setId(java.lang.Long);
	  public void setTitle(java.lang.String);
	  public void setContent(java.lang.String);
	  public void setStatus(com.hantsylabs.restexample.springmvc.domain.Post$Status);
	  public void setCreatedBy(com.hantsylabs.restexample.springmvc.domain.User);
	  public void setCreatedDate(java.time.LocalDateTime);
	  public void setLastModifiedBy(com.hantsylabs.restexample.springmvc.domain.User);
	  public void setLastModifiedDate(java.time.LocalDateTime);
	  public boolean equals(java.lang.Object);
	  protected boolean canEqual(java.lang.Object);
	  public int hashCode();
	  public java.lang.String toString();
	  public com.hantsylabs.restexample.springmvc.domain.Post();
	  public com.hantsylabs.restexample.springmvc.domain.Post(java.lang.Long, java.lang.String, java.lang.String, com.hantsylabs.restexample.springmvc.domain.Post$Status, com.hantsylabs.restexample.springmvc.domain.User, java.time.LocalDateTime, com.hantsylabs.restexample.springmvc.domain.User, java.time.LocalDateTime);
	} 

It prints all signatures of members of *Post.class*. As you see all essential methods(getters, setters, equals, hashCode, toString) have been added into the *Post.class*, and there is a *Post$Builder.class* file existed in the same folder, which is an inner class in the Post and implements the *Builder* pattern.

You can create a `Post` object using Post builder like this.

	Post post = Post.builder()
                .title("title of my first post")
                .content("content of my first post")
                .build();

Compare to following legacy new an object, the Builder pattern is more friendly to developers, and codes become more readable.

	Post post = new Post();
	post.setTitle("title of my first post");
	post.setContent("content of my first post");
	
###Model associations

Let's create other related models, `Comment` and `User`.

`Comment` class is associated with `Post` and `User`. Every comment should be belong to a post, and has an author(`User`).

	@Getter
	@Setter
	@ToString
	@Builder
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Table(name = "comments")
	public class Comment implements Serializable {

		/**
		 *
		 */
		private static final long serialVersionUID = 1L;

		@Id()
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		@Column(name = "id")
		private Long id;

		@Column(name = "content")
		private String content;

		@JoinColumn(name = "post_id")
		@ManyToOne()
		private Post post;

		@ManyToOne
		@JoinColumn(name = "created_by")
		@CreatedBy
		private User createdBy;

		@Column(name = "created_on")
		@CreatedDate
		private LocalDateTime createdDate;

		@Override
		public int hashCode() {
			int hash = 5;
			hash = 89 * hash + Objects.hashCode(this.content);
			return hash;
		}

		@Override
		public boolean equals(Object obj) {
			if (this == obj) {
				return true;
			}
			if (obj == null) {
				return false;
			}
			if (getClass() != obj.getClass()) {
				return false;
			}
			final Comment other = (Comment) obj;
			if (!Objects.equals(this.content, other.content)) {
				return false;
			}
			return true;
		}
	}

`User` class contains fields of a user account, including username and password which used for authentication.
	
	@Data
	@Builder
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Table(name = "users")
	public class User implements Serializable {

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
	}	
	
A `Post` should have an author.

	public class Post{
	
	    @ManyToOne
		@JoinColumn(name = "created_by")
		@CreatedBy
		User createdBy;
	}

##Data persistence with JPA	

Generally, in Spring application, in order to make JPA work, you have to configure a `DataSource`, `EntityManagerFactory`, `TransactionManager`.

###JPA overview

JPA standardised Hibernate and it is part of Java EE specification since Java EE 5. Currently there are some popular JPA providers, such as Hibernate, OpenJPA, EclipseLink etc. EclipseLink is shipped wtih Glassfish, and Hibernate is included JBoss Wildfly/Redhat EAP.

In the above Modeling section, we have created models, which are JPA entities. In this section, let's see how to make it work.

####Configure DataSources

Like other ORM frameworks, you have to configure a DataSource.

The [DataSourceConfig](https://github.com/hantsy/angularjs-springmvc-sample/blob/master/src/main/java/com/hantsylabs/restexample/springmvc/config/DataSourceConfig.java) defines a series of DataSource for differnt profiles.

	@Configuration
	public class DataSourceConfig {

		private static final String ENV_JDBC_PASSWORD = "jdbc.password";
		private static final String ENV_JDBC_USERNAME = "jdbc.username";
		private static final String ENV_JDBC_URL = "jdbc.url";

		@Inject
		private Environment env;

		@Bean
		@Profile("dev")
		public DataSource dataSource() {
			return new EmbeddedDatabaseBuilder()
					.setType(EmbeddedDatabaseType.H2)
					.build();
		}

		@Bean
		@Profile("staging")
		public DataSource testDataSource() {
			BasicDataSource bds = new BasicDataSource();
			bds.setDriverClassName("com.mysql.jdbc.Driver");
			bds.setUrl(env.getProperty(ENV_JDBC_URL));
			bds.setUsername(env.getProperty(ENV_JDBC_USERNAME));
			bds.setPassword(env.getProperty(ENV_JDBC_PASSWORD));
			return bds;
		}

		@Bean
		@Profile("prod")
		public DataSource prodDataSource() {
			JndiObjectFactoryBean ds = new JndiObjectFactoryBean();
			ds.setLookupOnStartup(true);
			ds.setJndiName("jdbc/postDS");
			ds.setCache(true);

			return (DataSource) ds.getObject();
		}

	}

In development stage("dev" profile is activated), using an embedded database is more easy to write tests, and speeds up development progress. In an integration server, it is recommended to run the appliation and integration tests on an environment close to production deployment. In a production environment,most of case, using a container managed datasource is effective.

The Spring profile can be activiated by an environment varible: `spring.profiles.active`. In our case, I used maven to set `spring.profiles.active` at compile time.

1. In the maven profile section, there is `spring.profiles.active` property defined. eg.

        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>

                <log4j.level>DEBUG</log4j.level>

                <spring.profiles.active>dev</spring.profiles.active>
                <!-- hibernate -->
                <hibernate.hbm2ddl.auto>create</hibernate.hbm2ddl.auto>
                <hibernate.show_sql>true</hibernate.show_sql>
                <hibernate.format_sql>true</hibernate.format_sql>

                <!-- mail config -->
            </properties>
			//...
        </profile>

2.  Then used maven resource filter to replaced the placeholder defined in `app.properties`. Every maven profile could have a specific folder to hold the profiled based files. eg.

		<profile>
            <id>dev</id>
			//...
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources-dev</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
		
		

3. After it is compiled, content of `app.properties` is filtered and replaced with the defined property.

		#app config properties
		spring.profiles.active=@spring.profiles.active@
		
	Becomes:

		#app config properties
		spring.profiles.active=dev
		
4. In configuration class, add `PropertySource` to load the properties file.

		@PropertySource("classpath:/app.properties")
		@PropertySource(value = "classpath:/database.properties", ignoreResourceNotFound = true)
		public class AppConfig {

		}
		
NOTE: Read the Spring official document about [Spring profile and Environment](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/beans.html#beans-definition-profiles).		
		
####Configure JPA and transaction

Let's have a look at the [JpaConfig](https://github.com/hantsy/angularjs-springmvc-sample/blob/master/src/main/java/com/hantsylabs/restexample/springmvc/config/JpaConfig.java), it defines a `EntityManagerFactory` and `TransactionManager`.

	@Configuration
	@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)
	public class JpaConfig {

		private static final Logger log = LoggerFactory.getLogger(JpaConfig.class);

		private static final String ENV_HIBERNATE_DIALECT = "hibernate.dialect";
		private static final String ENV_HIBERNATE_HBM2DDL_AUTO = "hibernate.hbm2ddl.auto";
		private static final String ENV_HIBERNATE_SHOW_SQL = "hibernate.show_sql";
		private static final String ENV_HIBERNATE_FORMAT_SQL = "hibernate.format_sql";

		@Inject
		private Environment env;

		@Inject
		private DataSource dataSource;

		@Bean
		public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
			LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
			emf.setDataSource(dataSource);
			emf.setPackagesToScan("com.hantsylabs.restexample.springmvc");
			emf.setPersistenceProvider(new HibernatePersistenceProvider());
			emf.setJpaProperties(jpaProperties());
			return emf;
		}

		private Properties jpaProperties() {
			Properties extraProperties = new Properties();
			extraProperties.put(ENV_HIBERNATE_FORMAT_SQL, env.getProperty(ENV_HIBERNATE_FORMAT_SQL));
			extraProperties.put(ENV_HIBERNATE_SHOW_SQL, env.getProperty(ENV_HIBERNATE_SHOW_SQL));
			extraProperties.put(ENV_HIBERNATE_HBM2DDL_AUTO, env.getProperty(ENV_HIBERNATE_HBM2DDL_AUTO));
			if (log.isDebugEnabled()) {
				log.debug(" hibernate.dialect @" + env.getProperty(ENV_HIBERNATE_DIALECT));
			}
			if (env.getProperty(ENV_HIBERNATE_DIALECT) != null) {
				extraProperties.put(ENV_HIBERNATE_DIALECT, env.getProperty(ENV_HIBERNATE_DIALECT));
			}
			return extraProperties;
		}

		@Bean
		public PlatformTransactionManager transactionManager() {
			return new JpaTransactionManager(entityManagerFactory().getObject());
		}
	}	

Generally, JPA is activiated by `META-INF/persistence.xml` file in container. 

The following is a classic JPA persistence.xml. The transaction-type is `RESOUECE_LOCAL`, another available option is `JTA`. Most of the time, Spring runs in a Servlet container which does not support `JTA`. 

	<persistence version="2.1"
	   xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="
			http://xmlns.jcp.org/xml/ns/persistence
			http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
	   <persistence-unit name="primary" transaction-type="RESOUECE_LOCAL">

		  <class>...Post</class>
		  <none-jta-data-source/>
		  <properties>
			 <!-- Properties for Hibernate -->
			 <property name="hibernate.hbm2ddl.auto" value="create-drop" />
			 <property name="hibernate.show_sql" value="false" />
		  </properties>
	   </persistence-unit>
	</persistence>

You have to specify the entity classes to be loaded, and datasource here. Spring provides a `LocalEntityManagerFactoryBean` to simplifies configuration work, there is a `setPackagesToScan` method to specify the package will be scanned, and reuse the Spring DataSource configuration for datasource configuration.

More simply, `LocalContainerEntityManagerFactoryBean` does not need persistence.xml any more and builds the JPA evironment from scratch. In above codes, we used `LocalContainerEntityManagerFactoryBean` simplifies JPA configuration.

Now we are ready for using JPA.

An example of JPA usage could like.

	@Repository
	@Transactional
	public class PostRepository{
	
		@PersistenceContext
		private EntityManager em;
	}

`@Repository` is an alias of `@Component`, `Transactional` is used to enable transaction on this bean. `@PersistenceContext` to inject an `EntityManager` to this bean. `EntityManager` provides a plent of methods to operate database.

For example,

`em.persist(Post)` to persist new entity.
`em.merge(Post)` to merge the passed data into the entity existed and return a copy of the updated entity.

If you want to explore all methods provided in EntityManager, check [EntityManager javadoc](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html). 

###Spring Data JPA

Spring Data JPA simplifies JPA, please read [an early post I wrote to discuss this topic](http://hantsy.blogspot.com/2013/10/spring-data-jpa.html).

Use a `EnableJpaRepositories` to activiate Spring Data JPA. `basePackages` specifies packages will be scanned by Spring Data.

	@Configuration
	@EnableJpaRepositories(basePackages = {"com.hantsylabs.restexample.springmvc"})
	@EnableJpaAuditing(auditorAwareRef = "auditor")
	public class JpaConfig {

		@Bean
		public AuditorAware<User> auditor() {
			return () -> SecurityUtil.currentUser();
		}

	}
	
`EnableJpaAuditing` enable a simple auditing features provided in Spring Data JPA. There is some annotations are designated for it. Such as:

* `@CreatedBy`
* `@CreatedDate`
* `@LastModifiedBy`
* `@LastModifiedDate`

When `AuditingEntityListener` is activated globally in */META-INF/orm.xml*. Any fields annotated with above annotaitions will be filled automatically when the hosted entity is created and updated.

	<?xml version="1.0" encoding="UTF-8"?>
	<entity-mappings 
		xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" 
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence/orm http://xmlns.jcp.org/xml/ns/persistence/orm_2_1.xsd" version="2.1">
		<persistence-unit-metadata>
			<persistence-unit-defaults>
				<entity-listeners>
					<entity-listener class="org.springframework.data.jpa.domain.support.AuditingEntityListener" />
				</entity-listeners>
			</persistence-unit-defaults>
		</persistence-unit-metadata>
	</entity-mappings>

`CreatedBy` and `LastModifiedBy` try to find a `AuditAware` bean and inject it into these fields at runtime. A simple implementation is getting the current principal from the Spring Security context.
		
### Repository

You can imagine a repository as a domain object collection, allow you retreive data from it or save change state back. 

[Spring Data Commons project](https://github.com/spring-projects/spring-data-commons) defines a series of interfaces for common data operations for different storages, including NoSQL and RDBMS.

The top-level repository representation is the `Repository` interface.

`CrudRepository` subclasses from `Repository` and includes extra creating, retrieving, updating and deleting operations, aka CRUD.

A simple CRUD operations just need to create an interface extends `CrudRepository`.

	interface MyPository extends CrudRepository{}
	
Create repository for Post.

	public interface PostRepository extends JpaRepository<Post, Long>,//
			JpaSpecificationExecutor<Post>{

	}
	
`JpaRepository` is a JPA specific repository and derived from `PagingAndSortingRepository` which also subclasses from `CrudRepository` and provides pagination and sort capability. It accepts a `Pageable` object as method arguments, and can return a paged result with `Page` class.

`JpaSpecificationExecutor` is designated for JPA Criteria Query API, and provides type safe query instead of literal based JPQL.

For example, to search post by input keyword.

	public static Specification<Post> filterByKeywordAndStatus(
            final String keyword,//
            final Post.Status status) {
        return (Root<Post> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
            List<Predicate> predicates = new ArrayList<>();
            if (StringUtils.hasText(keyword)) {
                predicates.add(
					cb.or(
						cb.like(root.get(Post_.title), "%" + keyword + "%"),
						cb.like(root.get(Post_.content), "%" + keyword + "%")
					)
                );
            }

            if (status != null) {
                predicates.add(cb.equal(root.get(Post_.status), status));
            }

            return cb.and(predicates.toArray(new Predicate[predicates.size()]));
        }; 
    }
	
And you want to get a pageable result, you can use like this, just add a `Pageable` argument.

	Page<Post> posts = postRepository.findAll(PostSpecifications.filterByKeywordAndStatus(q, status), page);
			
`page` argument is a `Pageable` object which can transfer pagination parameters from client request, and return result is a typed `Page` object, it includes the items of the current page and page naviation meta, such as total items, etc.	

## Application Service

A service can delegate CRUD operations to repository, also act as gateway to other bound context, such as messageing, sending email, fire events etc.

	@Service
	@Transactional
	public class BlogService {

		private static final Logger log = LoggerFactory.getLogger(BlogService.class);

		private PostRepository postRepository;
	
		@Inject
		public BlogService(PostRepository postRepository){
			this.postRepository = postRepository;
		}

		public Page<PostDetails> searchPostsByCriteria(String q, Post.Status status, Pageable page) {

			log.debug("search posts by keyword@" + q + ", page @" + page);

			Page<Post> posts = postRepository.findAll(PostSpecifications.filterByKeywordAndStatus(q, status),
					page);

			log.debug("get posts size @" + posts.getTotalElements());

			return DTOUtils.mapPage(posts, PostDetails.class);
		}

		public PostDetails savePost(PostForm form) {

			log.debug("save post @" + form);

			Post post = DTOUtils.map(form, Post.class);

			Post saved = postRepository.save(post);

			log.debug("saved post is @" + saved);

			return DTOUtils.map(saved, PostDetails.class);

		}

		public PostDetails updatePost(Long id, PostForm form) {
			Assert.notNull(id, "post id can not be null");

			log.debug("update post @" + form);

			Post post = postRepository.findOne(id);
			DTOUtils.mapTo(form, post);

			Post saved = postRepository.save(post);

			log.debug("updated post@" + saved);
			
			return DTOUtils.map(saved, PostDetails.class);
		}

		public PostDetails findPostById(Long id) {

			Assert.notNull(id, "post id can not be null");

			log.debug("find post by id@" + id);

			Post post = postRepository.findOne(id);

			if (post == null) {
				throw new ResourceNotFoundException(id);
			}

			return DTOUtils.map(post, PostDetails.class);
		}
	}

In this service there are some POJOs created for input request, response presentation etc.

`PostForm` gathers the user input data from client.

	public class PostForm implements Serializable {

		@NotBlank
		private String title;

		private String content;
	}	

Validation annotations can be applied on it.

`PostDetails` represents the return result of a post details. Usually, it includes some info that should not include in the `PostForm`, such as id, timestamp etc.

	public class PostDetails implements Serializable {

		private Long id;

		private String title;

		private String content;

		private String status;

		private Date createdDate;
		
	}	

Some exceptions are threw in the service if the input data can not satisfy the requirements. In the furthur post, I will focus on *exception handling* topic.

There is a `DTOUtils` which is responsible for data copy from one class to another class. 

	/**
	 *
	 * @author Hantsy Bai<hantsy@gmail.com>
	 */
	public final class DTOUtils {

		private static final ModelMapper INSTANCE = new ModelMapper();
		
		private DTOUtils() {
			throw new InstantiationError( "Must not instantiate this class" );
		}

		public static <S, T> T map(S source, Class<T> targetClass) {
			return INSTANCE.map(source, targetClass);
		}

		public static <S, T> void mapTo(S source, T dist) {
			INSTANCE.map(source, dist);
		}

		public static <S, T> List<T> mapList(List<S> source, Class<T> targetClass) {
			List<T> list = new ArrayList<>();
			for (int i = 0; i < source.size(); i++) {
				T target = INSTANCE.map(source.get(i), targetClass);
				list.add(target);
			}

			return list;
		}

		public static <S, T> Page<T> mapPage(Page<S> source, Class<T> targetClass) {
			List<S> sourceList = source.getContent();

			List<T> list = new ArrayList<>();
			for (int i = 0; i < sourceList.size(); i++) {
				T target = INSTANCE.map(sourceList.get(i), targetClass);
				list.add(target);
			}

			return new PageImpl<>(list, new PageRequest(source.getNumber(), source.getSize(), source.getSort()),
					source.getTotalElements());
		}
	}

It used the effort of [ModelMapper project](http://modelmapper.org/).
	
## Produces REST APIs with Spring MVC

Like other traditional action based framework, such as Apache Struts, etc, Spring MVC implements the standard MVC pattern. But against the benifit of IOC container, each parts of Spring MVC are not coupled, esp. view and view resolver are pluginable and can be configured. 

For RESTful applications, JSON and XML are the commonly used exchange format, we do not need a template engine(such as Freemarker, Apache Velocity) for view, Spring MVC will detect HTTP headers, such as *Content Type*, *Accept Type*, etc. to determine how to produce corresponding view result. Most of the time, we do not need to configure the view/view resolver explictly. This is called *Content negotiation*. There is a `ContentNegotiationManager` bean which is responsible for *Content negotiation* and enabled by default in the latest version.

The configuration details are motioned in before posts. We are jumping to write `@Controller` to produce REST APIs.

Follows the REST design convention, create `PostController` to produce REST APIs.

	@RestController
	@RequestMapping(value = Constants.URI_API + Constants.URI_POSTS)
	public class PostController {

		private static final Logger log = LoggerFactory
				.getLogger(PostController.class);

		private BlogService blogService;

		@Inject
		public PostController(BlogService blogService) {
			this.blogService = blogService;
		}

		@RequestMapping(value = "", method = RequestMethod.GET)
		@ResponseBody
		public ResponseEntity<Page<PostDetails>> getAllPosts(
				@RequestParam(value = "q", required = false) String keyword, //
				@RequestParam(value = "status", required = false) Post.Status status, //
				@PageableDefault(page = 0, size = 10, sort = "title", direction = Direction.DESC) Pageable page) {

			log.debug("get all posts of q@" + keyword + ", status @" + status + ", page@" + page);

			Page<PostDetails> posts = blogService.searchPostsByCriteria(keyword, status, page);

			log.debug("get posts size @" + posts.getTotalElements());

			return new ResponseEntity<>(posts, HttpStatus.OK);
		}

		@RequestMapping(value = "/{id}", method = RequestMethod.GET)
		@ResponseBody
		public ResponseEntity<PostDetails> getPost(@PathVariable("id") Long id) {

			log.debug("get postsinfo by id @" + id);

			PostDetails post = blogService.findPostById(id);

			log.debug("get post @" + post);

			return new ResponseEntity<>(post, HttpStatus.OK);
		}

		@RequestMapping(value = "/{id}/comments", method = RequestMethod.GET)
		@ResponseBody
		public ResponseEntity<Page<CommentDetails>> getCommentsOfPost(
				@PathVariable("id") Long id,
				@PageableDefault(page = 0, size = 10, sort = "createdDate", direction = Direction.DESC) Pageable page) {

			log.debug("get comments of post@" + id + ", page@" + page);

			Page<CommentDetails> commentsOfPost = blogService.findCommentsByPostId(id, page);

			log.debug("get post comment size @" + commentsOfPost.getTotalElements());

			return new ResponseEntity<>(commentsOfPost, HttpStatus.OK);
		}

		@RequestMapping(value = "", method = RequestMethod.POST)
		@ResponseBody
		public ResponseEntity<ResponseMessage> createPost(@RequestBody @Valid PostForm post, BindingResult errResult) {

			log.debug("create a new post");
			if (errResult.hasErrors()) {
				throw new InvalidRequestException(errResult);
			}

			PostDetails saved = blogService.savePost(post);

			log.debug("saved post id is @" + saved.getId());

			HttpHeaders headers = new HttpHeaders();
			headers.setLocation(ServletUriComponentsBuilder.fromCurrentContextPath()
					.path(Constants.URI_API + Constants.URI_POSTS + "/{id}")
					.buildAndExpand(saved.getId())
					.toUri()
			);
			return new ResponseEntity<>(ResponseMessage.success("post.created"), headers, HttpStatus.CREATED);
		}

		@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
		@ResponseBody
		public ResponseEntity<ResponseMessage> deletePostById(@PathVariable("id") Long id) {

			log.debug("delete post by id @" + id);

			blogService.deletePostById(id);

			return new ResponseEntity<>(ResponseMessage.success("post.updated"), HttpStatus.NO_CONTENT);
		}

	}
	
`@RestController` is a REST ready annotation, it is combined with `@Controller` and `@ResponseBody`.	

`@RequestMapping` defines URL, HTTP methods etc are matched, the annotated method will handle the request. 

*/api/posts* stands for a collection of `Post`, and */api/posts/{id}* is navigating to a specific Post which identified by id.

A *POST* method on */api/posts* is use for creating a new post, return HTTP status 201, and set HTTP header *Location* value to the new created post url if the creation is completed successfully.

*GET*, *PUT*, *DELETE* on */api/posts/{id}* performs retrieve, update, delete action on the certain post. *GET* will return a 200 HTTP status code, and *PUT*, *DELETE* return 204 if the operations are done as expected and there is no content body needs to be sent back to clients. 

## Run 

For none Spring Boot application, run it as a general web application in IDE. 

Tomcat also provides maven plugin for those maven users.

	<plugin>
		<groupId>org.apache.tomcat.maven</groupId>
		<artifactId>tomcat7-maven-plugin</artifactId>
		<version>2.2</version>
		<configuration>
			<path>/angularjs-springmvc-sample</path>
		</configuration>
	</plugin>
	
Execute this in the command line to start this application.

    mvn tomcat7:run

**NOTE**: The tomcat maven plugin development is not active, if you are using Servlet 3.1 features, you could have to use other plugin instead. 

Jetty is the fastest embedded Servlet container and wildly used in development community. 

	<plugin>
		<groupId>org.eclipse.jetty</groupId>
		<artifactId>jetty-maven-plugin</artifactId>
		<version>9.3.7.v20160115</version>
		<configuration>
			<scanIntervalSeconds>10</scanIntervalSeconds>
			<stopPort>8005</stopPort>
			<stopKey>STOP</stopKey>
			<webApp>
				<contextPath>/angularjs-springmvc-sample</contextPath>
			</webApp>
		</configuration>
	</plugin>
	
Execute the following command to run the application on an embedded Jetty server.

    mvn jetty:run	

Another frequently used is Cargo which provides support for all popular applcation servers, and ready for all hot build tools, such as Ant, Maven, Gradle etc.

	<plugin>
		<groupId>org.codehaus.cargo</groupId>
		<artifactId>cargo-maven2-plugin</artifactId>
		<configuration>
			<container>
				<containerId>tomcat8x</containerId>
				<type>embedded</type>
			</container>

			<configuration>
				<properties>
					<cargo.servlet.port>9000</cargo.servlet.port>
					<cargo.logging>high</cargo.logging>
				</properties>
			</configuration>
		</configuration>
	</plugin>
	
Execute the following command to run the application on an embedded Tomcat 8 server with the help of Cargo.

    mvn verify cargo:run	

For Spring boot application, it is simple, just run the application like this.

	mvn spring-boot:run

By default, it uses Tomcat embedded server, but you can switch to Jetty and JBoss Undertow if you like. Check the Spring boot docs for details.

##Source Code

Check out sample codes from my github account.

	git clone https://github.com/hantsy/angularjs-springmvc-sample
	
Or the Spring Boot version:

	git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
	
Read the live version of thess posts from Gitbook:[Building RESTful APIs with Spring MVC](https://www.gitbook.com/book/hantsy/build-a-restful-app-with-spring-mvc-and-angularjs/details).


	


	
	