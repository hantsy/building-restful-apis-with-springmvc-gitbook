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

A repository is a resource collection, allow clients retreive resources from repository and saving resource state into repository. 

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

	