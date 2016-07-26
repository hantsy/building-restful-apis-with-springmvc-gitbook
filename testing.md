#Testing

Before release your applicaiton to the public world, you have to make sure it works as expected.

Testing is the most effective way to prove your codes correct.

##Test driven development

In the XP and Agile world, lots of developers are TDD advocate, and use it in daily work.

The basic flow of TDD can be summaried as:

1. Write a test first, then run test and get failure, failure info indicates what to do(You have not written any codes yet).
2. Code the implementation, and run test again and again, untill the test get passed.
3. Adjust the test to add more featurs, and refactor the codes, till all considerations are included.

But some developers prefer writing codes firstly and then write tests to verify them, it is OK. There is no policy to force you accept TDD. For a skilled developer, both are productive in work.

We have written some codes in the early posts, now it is time to add some test codes to show up how to test Spring components.

Spring provides a test context environment for developers, it supports JUnit and TestNG.

In this sample applcation, I will use JUnit as test runner, also use [Mockito](http://mockito.org) to test service in isolation, and use [Rest Assured](http://rest-assured.io/) BDD like fluent APIs to test REST from client view.

##A simple POJO test

A classic JUnit test could look like this.

	public class PostTest {

		public PostTest() {
		}

		@BeforeClass
		public static void setUpClass() {
		}

		@AfterClass
		public static void tearDownClass() {
		}

		private Post post;

		@Before
		public void setUp() {
			post = new Post();
			post.setTitle("test title");
			post.setContent("test content");
		}

		@After
		public void tearDown() {
			post = null;
		}

		/**
		 * Test of getId method, of class Post.
		 */
		@Test
		public void testPojo() {
			assertEquals("test title", post.getTitle());
			assertEquals("test content", post.getContent());
		}

	}

`@BeforeClass` and `AfterClass` method must be *static*, these will be executed after the test class is constructed and before it is destoryed.

`@Before` and `@After` will be executed around a test case.

A test case is a method annotated with `@Test`.

Another annotation we always used is `@RunWith`, such as `@RunWith(SpringJUnit4ClassRunner.class)`, which will prepare specific test context for Spring tests.
	
Run the test in command line.

	mvn test -Dtest=PostTest

You could see the following output summary for this test.	

	-------------------------------------------------------
	 T E S T S
	-------------------------------------------------------
	Running com.hantsylabs.restexample.springmvc.domain.PostTest
	Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.015 sec - in com.hantsylabs.restexample.springmvc.domain.PostTest

	Results :

	Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
	
`Post` is a simple POJO, does not depend on other dependencies.	

##Test Service

`BlogService` depends on `PostRepository`, but most of time, we only want to check if the business logic and flow correct in the `BlogService` and assume the dependency `PostRepository` are always working as expected. Thus it is easy to focus on testing `BlogService` itself. 

Mockito provides the simplest approaches to mock the dependencies, and setup the assumption, and provides an isolation environment to test `BlogService`.

### Isolate dependencies with Mockito

Mocks `PostRepository`, when invoke methods of `PostRepository`, return the dummy data we assumed. Mockito gives us a simple way to complete the assumption progress.

Create a `MockDataConfig` configuration class in test package.

	@Configuration
	public class MockDataConfig {

		@Bean
		public PostRepository postRepository() {
			final Post post = createPost();
			PostRepository posts = mock(PostRepository.class);
			when(posts.save(any(Post.class))).thenAnswer((InvocationOnMock invocation) -> {
				Object[] args = invocation.getArguments();
				Post result = (Post) args[0];
				result.setId(post.getId());
				result.setTitle(post.getTitle());
				result.setContent(post.getContent());
				result.setCreatedDate(post.getCreatedDate());
				return result;
			});

			when(posts.findOne(1000L)).thenThrow(new ResourceNotFoundException(1000L));
			when(posts.findOne(1L)).thenReturn(post);
			when(posts.findAll(any(Specification.class), any(Pageable.class))).thenReturn(new PageImpl(Arrays.asList(post), new PageRequest(0, 10), 1L));
			when(posts.findAll()).thenReturn(Arrays.asList(post));
			return posts;
		}

		@Bean
		public CommentRepository commentRepository() {
			return mock(CommentRepository.class);
		}
		
		@Bean
		public BlogService blogService(PostRepository posts, CommentRepository comments){
			return new BlogService(posts, comments);
		}

		@Bean
		public Post createPost() {
			Post post = new Post();
			post.setCreatedDate(new Date());
			post.setId(1L);
			post.setTitle("First post");
			post.setContent("Content of my first post!");
			post.setCreatedDate(new Date());

			return post;
		}
	}
	
We create mocked `PostRepository` in this configuration. 

Create `MockBlogServiceTest` for `BlogService`, used `MockDataConfig` to load configurations.

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = {MockDataConfig.class})
	public class MockBlogServiceTest {

		private static final Logger LOG = LoggerFactory.getLogger(MockBlogServiceTest.class);

		@Inject
		private PostRepository postRepository;

		@Inject
		private CommentRepository commentRepository;
		
	}	

`PostRepository` and `CommentRepository` are mocked object defined in the `MockDataConfig`.

Add a test method in `MockBlogServiceTest`.

    @Test
    public void testSavePost() {
        PostForm form = new PostForm();
        form.setTitle("saving title");
        form.setContent("saving content");

        PostDetails details = blogService.savePost(form);

        LOG.debug("post details @" + details);
        assertNotNull("saved post id should not be null@", details.getId());
        assertTrue(details.getId() == 1L);
	
		//...
	}	

In the above test, asserts id value of the returned post is 1L. 

Have a look at `MockDataConfig`, when calls  save method of `PostRepository` and return a dummy post instance which id is 1L.	

We can also test if the exception threw as expected in the `MockDataConfig`.

    @Test(expected = ResourceNotFoundException.class)
    public void testGetNoneExistingPost() {
        blogService.findPostById(1000L);
    }
	
Run the test in command line tools.

	mvn test -Dtest=MockBlogServiceTest
	
You will see the test result like.

	-------------------------------------------------------
	 T E S T S
	-------------------------------------------------------
	Running com.hantsylabs.restexample.springmvc.test.MockBlogServiceTest
	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.943 sec - in com.hantsylabs.restexample.springmvc.test.MockBlogServiceTest

	Results :

	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

	
### Integration test 

We have known `BlogService` works when we mocked the dependencies. 

Now we write a test to check if it works against a real database.

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = {AppConfig.class, DataSourceConfig.class, DataJpaConfig.class, JpaConfig.class})
	public class BlogServiceTest {

		private static final Logger LOG = LoggerFactory.getLogger(BlogServiceTest.class);

		@Inject
		private PostRepository postRepository;

		@Inject
		private BlogService blogService;

		private Post post;

		public BlogServiceTest() {
		}

		@Before
		public void setUp() {
			postRepository.deleteAll();
			post = postRepository.save(Fixtures.createPost("My first post", "content of my first post"));

			assertNotNull(post.getId());
		}

		@After
		public void tearDown() {
		}

		@Test
		public void testSavePost() {
			PostForm form = new PostForm();
			form.setTitle("saving title");
			form.setContent("saving content");

			PostDetails details = blogService.savePost(form);

			LOG.debug("post details @" + details);
			assertNotNull("saved post id should not be null@", details.getId());
			assertNotNull(details.getId());

			Page<PostDetails> allPosts = blogService.searchPostsByCriteria("", null, new PageRequest(0, 10));
			assertTrue(allPosts.getTotalElements() == 2);

			Page<PostDetails> posts = blogService.searchPostsByCriteria("first", Post.Status.DRAFT, new PageRequest(0, 10));
			assertTrue(posts.getTotalPages() == 1);
			assertTrue(!posts.getContent().isEmpty());
			assertTrue(Objects.equals(posts.getContent().get(0).getId(), post.getId()));

			PostForm updatingForm = new PostForm();
			updatingForm.setTitle("updating title");
			updatingForm.setContent("updating content");
			PostDetails updatedDetails = blogService.updatePost(post.getId(), updatingForm);

			assertNotNull(updatedDetails.getId());
			assertTrue("updating title".equals(updatedDetails.getTitle()));
			assertTrue("updating content".equals(updatedDetails.getContent()));

		}

		@Test(expected = ResourceNotFoundException.class)
		public void testGetNoneExistingPost() {
			blogService.findPostById(1000L);
		}

	}
	
In the `@Before` method, all Post data are cleared for each tests, and save a `Post` for further test assertion. 

The above codes are similar with early Mockito version, the main difference is we have switched configuraitons to a real database. Check the `@ContextConfiguration` annotated on `BlogServiceTest`.

Run the test.

	mvn test -Dtest=BlogServiceTest

The test result should be shown as below.

	-------------------------------------------------------
	 T E S T S
	-------------------------------------------------------
	Running com.hantsylabs.restexample.springmvc.test.BlogServiceTest
	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.908 sec - in com.hantsylabs.restexample.springmvc.test.BlogServiceTest

	Results :

	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
	
##Test Controller

Spring provides a sort of mock APIs to emulate a Servlet container environment, thus it is possbile to test MVC related feature without a real container.

###Use MockMvc with mocked service

MockMvc does not need a Servlet container, but can test most of the Controller features.

Like the former `MockBlogServiceTest`, we can mock the controller's dependencies, thus is no need to load Spring configuraitons. 


	@RunWith(MockitoJUnitRunner.class)
	public class MockPostControllerTest {

		private static final Logger log = LoggerFactory.getLogger(MockPostControllerTest.class);

		private MockMvc mvc;

		ObjectMapper objectMapper = new ObjectMapper();

		@Mock
		private BlogService blogService;

		@Mock
		Pageable pageable = mock(PageRequest.class);

		@InjectMocks
		PostController postController;

		@BeforeClass
		public static void beforeClass() {
			log.debug("==================before class=========================");
		}

		@AfterClass
		public static void afterClass() {
			log.debug("==================after class=========================");
		}

		@Before
		public void setup() {
			log.debug("==================before test case=========================");
			Mockito.reset();
			MockitoAnnotations.initMocks(this);
			mvc = standaloneSetup(postController)
					.setCustomArgumentResolvers(new PageableHandlerMethodArgumentResolver())
					.setViewResolvers(new ViewResolver() {
						@Override
						public View resolveViewName(String viewName, Locale locale) throws Exception {
							return new MappingJackson2JsonView();
						}
					})
					.build();
		}

		@After
		public void tearDown() {
			log.debug("==================after test case=========================");
		}

		@Test
		public void savePost() throws Exception {
			PostForm post = Fixtures.createPostForm("First Post", "Content of my first post!");

			when(blogService.savePost(any(PostForm.class))).thenAnswer(new Answer<PostDetails>() {
				@Override
				public PostDetails answer(InvocationOnMock invocation) throws Throwable {
					PostForm fm = (PostForm) invocation.getArgumentAt(0, PostForm.class);

					PostDetails result = new PostDetails();
					result.setId(1L);
					result.setTitle(fm.getTitle());
					result.setContent(fm.getContent());
					result.setCreatedDate(new Date());

					return result;
				}
			});
			mvc.perform(post("/api/posts").contentType(MediaType.APPLICATION_JSON).content(objectMapper.writeValueAsString(post)))
					.andExpect(status().isCreated());

			verify(blogService, times(1)).savePost(any(PostForm.class));
			verifyNoMoreInteractions(blogService);
		}

		@Test
		public void retrievePosts() throws Exception {
			PostDetails post1 = new PostDetails();
			post1.setId(1L);
			post1.setTitle("First post");
			post1.setContent("Content of first post");
			post1.setCreatedDate(new Date());

			PostDetails post2 = new PostDetails();
			post2.setId(2L);
			post2.setTitle("Second post");
			post2.setContent("Content of second post");
			post2.setCreatedDate(new Date());

			when(blogService.searchPostsByCriteria(anyString(), any(Post.Status.class), any(Pageable.class)))
					.thenReturn(new PageImpl(Arrays.asList(post1, post2), new PageRequest(0, 10, Direction.DESC, "createdDate"), 2));

			MvcResult response = mvc.perform(get("/api/posts?q=test&page=0&size=10"))
					.andExpect(status().isOk())
					.andExpect(jsonPath("$.content[*].id", hasItem(1)))
					.andExpect(jsonPath("$.content[*].title", hasItem("First post")))
					.andReturn();

			verify(blogService, times(1))
					.searchPostsByCriteria(anyString(), any(Post.Status.class), any(Pageable.class));
			verifyNoMoreInteractions(blogService);
			log.debug("get posts result @" + response.getResponse().getContentAsString());
		}

		@Test
		public void retrieveSinglePost() throws Exception {

			PostDetails post1 = new PostDetails();
			post1.setId(1L);
			post1.setTitle("First post");
			post1.setContent("Content of first post");
			post1.setCreatedDate(new Date());

			when(blogService.findPostById(1L)).thenReturn(post1);

			mvc.perform(get("/api/posts/1").accept(MediaType.APPLICATION_JSON))
					.andExpect(status().isOk())
					.andExpect(content().contentType("application/json;charset=UTF-8"))
					.andExpect(jsonPath("id").isNumber());

			verify(blogService, times(1)).findPostById(1L);
			verifyNoMoreInteractions(blogService);
		}

		@Test
		public void removePost() throws Exception {
			when(blogService.deletePostById(1L)).thenReturn(true);
			mvc.perform(delete("/api/posts/{id}", 1L))
					.andExpect(status().isNoContent());

			verify(blogService, times(1)).deletePostById(1L);
			verifyNoMoreInteractions(blogService);
		}

		@Test()
		public void notFound() {
			when(blogService.findPostById(1000L)).thenThrow(new ResourceNotFoundException(1000L));
			try {
				mvc.perform(get("/api/posts/1000").accept(MediaType.APPLICATION_JSON))
						.andExpect(status().isNotFound());
			} catch (Exception ex) {
				log.debug("exception caught @" + ex);
			}
		}

	}

In the `setup` method, `mvc = standaloneSetup(postController)` is trying to setup a `MockMvc` for controller. The test codes are easy to understand.

###MockMvc with a real database	

We changed a little on the above tests, replace the mocked service with the real configurations. Thus the tests will run against a real database, but still in mock mvc environment.

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = {AppConfig.class, Jackson2ObjectMapperConfig.class, DataSourceConfig.class, JpaConfig.class, DataJpaConfig.class, WebConfig.class})
	@WebAppConfiguration
	public class PostControllerTest {

		private static final Logger log = LoggerFactory.getLogger(PostControllerTest.class);

		@Inject
		WebApplicationContext wac;

		@Inject
		ObjectMapper objectMapper;

		@Inject
		private PostRepository postRepository;

		private MockMvc mvc;

		private Post post;

		@BeforeClass
		public static void beforeClass() {
			log.debug("==================before class=========================");
		}

		@AfterClass
		public static void afterClass() {
			log.debug("==================after class=========================");
		}

		@Before
		public void setup() {
			log.debug("==================before test case=========================");
			mvc = webAppContextSetup(this.wac).build();

			postRepository.deleteAll();
			post = postRepository.save(Fixtures.createPost("My first post", "content of my first post"));
		}

		@After
		public void tearDown() {
			log.debug("==================after test case=========================");
		}

		@Test
		public void savePost() throws Exception {
			PostForm post = Fixtures.createPostForm("First Post", "Content of my first post!");

			mvc.perform(post("/api/posts").contentType(MediaType.APPLICATION_JSON).content(objectMapper.writeValueAsString(post)))
					.andExpect(status().isCreated());

		}

		@Test
		public void retrievePosts() throws Exception {

			MvcResult response = mvc.perform(get("/api/posts?q=first&page=0&size=10"))
					.andExpect(status().isOk())
					.andExpect(jsonPath("$.content[0].id", is(post.getId().intValue())))
					.andExpect(jsonPath("$.content[0].title", is("My first post")))
					.andReturn();

			log.debug("get posts result @" + response.getResponse().getContentAsString());
		}

		@Test
		public void retrieveSinglePost() throws Exception {

			mvc.perform(get("/api/posts/{id}", post.getId()).accept(MediaType.APPLICATION_JSON))
					.andExpect(status().isOk())
					.andExpect(content().contentType("application/json"))
					.andExpect(jsonPath("$.id").isNumber())
					.andExpect(jsonPath("$.title", is("My first post")));

		}

		@Test
		public void removePost() throws Exception {
			mvc.perform(delete("/api/posts/{id}", post.getId()))
					.andExpect(status().isNoContent());
		}

		@Test()
		public void notFound() {
			try {
				mvc.perform(get("/api/posts/1000").accept(MediaType.APPLICATION_JSON))
						.andExpect(status().isNotFound());
			} catch (Exception ex) {
				log.debug("exception caught @" + ex);
			}
		}

	}

In this test class, the Mockito codes are replaced with Spring test, and load the configuraitons defined in this project. It is close to the final production environment, except there is not a real Servlet container.

###Test REST API as the client view

OK, now try to verify everything works in a real container.

	@RunWith(BlockJUnit4ClassRunner.class)
	public class IntegrationTest {

		private static final Logger log = LoggerFactory.getLogger(IntegrationTest.class);

		private static final String BASE_URL = "http://localhost:8080/angularjs-springmvc-sample/";

		private RestTemplate template;

		@BeforeClass
		public static void init() {
			log.debug("==================before class=========================");
		}

		@Before
		public void beforeTestCase() {
			log.debug("==================before test case=========================");
			template = new BasicAuthRestTemplate("admin", "test123");
		}

		@After
		public void afterTestCase() {
			log.debug("==================after test case=========================");
		}

		@Test
		public void testPostCrudOperations() throws Exception {
			PostForm newPost = Fixtures.createPostForm("My first post", "content of my first post");
			String postsUrl = BASE_URL + "api/posts";

			ResponseEntity<Void> postResult = template.postForEntity(postsUrl, newPost, Void.class);
			assertTrue(HttpStatus.CREATED.equals(postResult.getStatusCode()));
			String createdPostUrl = postResult.getHeaders().getLocation().toString();
			assertNotNull("created post url should be set", createdPostUrl);

			ResponseEntity<Post> getPostResult = template.getForEntity(createdPostUrl, Post.class);
			assertTrue(HttpStatus.OK.equals(getPostResult.getStatusCode()));
			log.debug("post @" + getPostResult.getBody());
			assertTrue(getPostResult.getBody().getTitle().equals(newPost.getTitle()));

			ResponseEntity<Void> deleteResult = template.exchange(createdPostUrl, HttpMethod.DELETE, null, Void.class);
			assertTrue(HttpStatus.NO_CONTENT.equals(deleteResult.getStatusCode()));

		}

		@Test
		public void noneExistingPost() throws Exception {
			String noneExistingPostUrl = BASE_URL + "api/posts/1000";
			try {
				template.getForEntity(noneExistingPostUrl, Post.class);
			} catch (HttpClientErrorException e) {
				assertTrue(HttpStatus.NOT_FOUND.equals(e.getStatusCode()));
			}
		}
	}

`RestTemplate` is use for interaction with remote REST API, this test acts as a remote client, and shake hands with our backend through REST APIs.

`BasicAuthRestTemplate` is a helper class to process *BASIC* authentication.

	public class BasicAuthRestTemplate extends RestTemplate {

		public BasicAuthRestTemplate(String username, String password) {
			addAuthentication(username, password);
		}

		private void addAuthentication(String username, String password) {
			if (username == null) {
				return;
			}
			List<ClientHttpRequestInterceptor> interceptors = Collections
					.<ClientHttpRequestInterceptor>singletonList(
							new BasicAuthorizationInterceptor(username, password));
			setRequestFactory(new InterceptingClientHttpRequestFactory(getRequestFactory(),
					interceptors));
		}

		private static class BasicAuthorizationInterceptor implements
				ClientHttpRequestInterceptor {

			private final String username;

			private final String password;

			public BasicAuthorizationInterceptor(String username, String password) {
				this.username = username;
				this.password = (password == null ? "" : password);
			}

			@Override
			public ClientHttpResponse intercept(HttpRequest request, byte[] body,
					ClientHttpRequestExecution execution) throws IOException {
				byte[] token = Base64.getEncoder().encode(
						(this.username + ":" + this.password).getBytes());
				request.getHeaders().add("Authorization", "Basic " + new String(token));
				return execution.execute(request, body);
			}

		}
	}

To run this test successfully, you have to configure *maven-failsafe-plugin* to set up a servlet container.

* Start up container before test is running 
* Shutdown the servlet container after the test is completed

	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-surefire-plugin</artifactId>
		<version>2.19</version>
		<configuration>
			<printSummary>true</printSummary>
			<redirectTestOutputToFile>true</redirectTestOutputToFile>
			<excludes>
				<exclude>**/*IntegrationTest*</exclude>
			</excludes>
		</configuration>
	</plugin>

Excludes the `IntegrationTest` in the *maven-surefire-plugin*.	
	
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-failsafe-plugin</artifactId>
		<version>2.12.4</version>
		<configuration>
			<includes>
				<include>**/*IntegrationTest*</include>
			</includes>
		</configuration>
		<executions>
			<execution>
				<id>integration-test</id>
				<goals>
					<goal>integration-test</goal>
				</goals>
			</execution>
			<execution>
				<id>verify</id>
				<goals>
					<goal>verify</goal>
				</goals>
			</execution>
		</executions>
	</plugin>
	
Filter the `IntegrationTest` in the *maven-failsafe-plugin*. Here I configured jetty as servlet container to run the test.		

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
		<executions>
			<execution>
				<id>start-jetty</id>
				<phase>pre-integration-test</phase>
				<goals>
					<goal>stop</goal>
					<goal>start</goal>
				</goals>
				<configuration>
					<scanIntervalSeconds>0</scanIntervalSeconds>
					<daemon>true</daemon>
				</configuration>
			</execution>
			<execution>
				<id>stop-jetty</id>
				<phase>post-integration-test</phase>
				<goals>
					<goal>stop</goal>
				</goals>
			</execution>
		</executions>
	</plugin> 

In the *pre-integration-test* phase, check if the jetty is running and starts up it, in `post-integration-test` phase, shutdown the container.

Run the `IntegrationTest` in command line.

	mvn clean verify 
	
In the console, after all unit tess are done, it will start jetty and deploy the project war into jetty and run the `IntegrationTest` on it.

	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.847 sec

	Results :

	Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

	[WARNING] File encoding has not been set, using platform encoding Cp1252, i.e. build is platform dependent!
	[INFO]
	[INFO] --- jetty-maven-plugin:9.3.7.v20160115:stop (stop-jetty) @ angularjs-springmvc-sample ---
	[INFO]
	[INFO] --- maven-failsafe-plugin:2.12.4:verify (verify) @ angularjs-springmvc-sample ---	

As you see in the console, after the test is done, it is trying to shutdown jetty.	

###Rest Assured

Rest Assured provides BDD like syntax, such as *given*, *when*, *then*, it is friendly for those familiar with BDD.

	@RunWith(SpringRunner.class)
	@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
	@Slf4j
	public class RestAssuredApplicationTest extends WebIntegrationTestBase {


		@Before
		public void beforeTest() {
			super.setup();
			RestAssured.port = port;
		}

		@Test
		public void testDeletePostNotExisted() {
			String location = "/api/posts/1000";

			given()
					.auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.delete(location)
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_NOT_FOUND);
		}

		@Test
		public void testGetPostNotExisted() {
			String location = "/api/posts/1000";

			given()
					.auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.get(location)
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_NOT_FOUND);
		}

		@Test
		public void testPostFormInValid() {
			PostForm form = new PostForm();

			given()
					.auth().basic(USER_NAME, PASSWORD)
					.body(form)
					.contentType(ContentType.JSON)
					.when()
					.post("/api/posts")
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_BAD_REQUEST);
		}

		@Test
		public void testPostCRUD() {
			PostForm form = new PostForm();
			form.setTitle("test title");
			form.setContent("test content");

			Response response = given()
					.auth().basic(USER_NAME, PASSWORD)
					.body(form)
					.contentType(ContentType.JSON)
					.when()
					.post("/api/posts")
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_CREATED)
					.and()
					.header("Location", containsString("/api/posts/"))
					.extract().response();

			String location = response.header("Location");

			log.debug("header location value @" + location);

			given().auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.get(location)
					.then()
					.assertThat()
					.body("title", is("test title"))
					.body("content", is("test content"));

			PostForm updateForm = new PostForm();
			updateForm.setTitle("test udpate title");
			updateForm.setContent("test update content");

			given()
					.auth().basic(USER_NAME, PASSWORD)
					.body(updateForm)
					.contentType(ContentType.JSON)
					.when()
					.put(location)
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_NO_CONTENT);

			given().auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.get(location)
					.then()
					.assertThat()
					.body("title", is("test udpate title"))
					.body("content", is("test update content"));

			given()
					.auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.delete(location)
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_NO_CONTENT);

			given().auth().basic(USER_NAME, PASSWORD)
					.contentType(ContentType.JSON)
					.when()
					.get(location)
					.then()
					.assertThat()
					.statusCode(HttpStatus.SC_NOT_FOUND);

		}

	}

This test is also run as client, and interacts with backend via REST API.

The above Rest Assured sample codes are available in the [Spring Boot version](https://github.com/hantsy/angularjs-springmvc-sample-boot), check out the codes and experience yourself.

It also includes a simple JBehave sample, if you are a JBehave user, you maybe interested in it.

##Source Code

Check out sample codes from my github account.

	git clone https://github.com/hantsy/angularjs-springmvc-sample
	
Or the Spring Boot version:

	git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
	
Read the live version of thess posts from Gitbook:[Building RESTful APIs with Spring MVC](https://www.gitbook.com/book/hantsy/build-a-restful-app-with-spring-mvc-and-angularjs/details).
	