#Build REST APIs

As stated in before posts, we are going to build a Blog sample REST APIs.

## Models

Create a JPA Entity class, named `Post`.

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
		
		//getters and setters, hashcode, equals, toString etc. are omitted
	}
	
It is a standard JPA entity. An entity class must be annotated with `@Entity`, and implements `Serializable` interface, and includes an identifier field(annotated with `@Id`), and a default none arguemtns constructor.	

If you are using Lombok, the getters, setters, equals, hashCode, toString and class constructor can generated at compile runtime.

	@Data
	@Builder
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Table(name = "posts")
	public class Post implements Serializable {}

There are some annotations provided in Lombok to archive this purpose.

`@Data` is a composite annotation which includes @Getter, @Setter etc.
`@Builder` will generate a Builder class for the hosted POJO, give user a fluent API to build an object.

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

**NOTE**: If you are using Eclipse based IDE, such as Spring Tool Suite, or Intellij IDEA, you could need to install the Lombok plugin, check the [Lombok download page](https://projectlombok.org/download.html) for installation information.

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

It includes all signatures of the members in *Post.class*. As you see the enssetial methods have been added into the *Post.class*, and there is a *Post$Builder.class* file exists in the same folder, which is an internal class in the Post and implements the *Builder* pattern.

You can create a `Post` object using Post builder like this.

	Post post = Post.builder()
                .title("title of my first post")
                .content("content of my first post")
                .build();

Compare to following legacy approache, the Builder pattern is more friendly to developers, and the code is more readable.

	Post post = new Post();
	post.setTitle("title of my first post");
	post.setContent("content of my first post");

## Repository

A repository is a domain object collection, allow clients retreive data from repository and saving resource state into repository. 

JPA standardised Hibernate 3.x and it is part of Java EE specification since Java EE 5. Currently there are some popular JPA providers, such as Hibernate, OpenJPA, EclipseLink etc. EclipseLink is shipped wtih Glassfish, and Hibernate is included JBoss Wildfly/Redhat EAP.

Spring Data JPA simplifies JPA in Spring, please check [an early post I wrote to discuss this topic](http://hantsy.blogspot.com/2013/10/spring-data-jpa.html).

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
	
And you want to get a pageable result, you can use like this.

	Page<Post> posts = postRepository.findAll(PostSpecifications.filterByKeywordAndStatus(q, status), page);
			
`page` argument is a `Pageable` object which can transfer pagination parameters from client request, and return result is a typed `Page` object, it includes the items of the current page and page naviation meta, such as total items, etc.	

## 	Service

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

## Controller

Like other traditional action based framework, such as Apache Struts, etc, Spring MVC implements the standard MVC pattern. But against the benifit of IOC container, each parts of Spring MVC are not coupled, esp. view and view resolver are pluginable and can be configured. 

For RESTful applications, JSON and XML are the commonly used exchange format, we do not need a template engine(such as Freemarker, Apache Velocity) for view, Spring can detect HTTP headers, such as *Content Type*, *Accept Type*, etc. to determine how to produce corresponding view result. Most of the time, we do not need to configure the view/view resolver explictly. This is called *Content negotiation*.

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

*/api/posts* stand for a collection of `Post`, and */api/posts/{id}* is navigation to the specific Post which identified by id.

A *POST* method on */api/posts* is use for creating a new post, return HTTP status 201, and set HTTP header *Location* value to the new created post url if the creation is completed successfully.

*GET*, *PUT*, *DELETE* on */api/posts/{id}* performs retrieve, update, delete action on the certain post. *GET* will return a 200 HTTP status code, and *PUT*, *DELETE* return 204 if the operations are done expected. 

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

**NOTE**: The tomcat maven plugin development is not active as expected, if you are using Servlet 3.1 features, you could have to use other plugin instead. 

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

    mvn cargo:run	

For Spring boot application, it is simple, just run the application like this.

	mvn spring-boot:run

By default, it uses Tomcat embedded server, but you can switch to Jetty and JBoss Undertow if you like. Check the Spring boot docs for details.



	
	