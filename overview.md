#Overview

In this minibook, I will demonstrate how to implement a RESTful web application with Spring MVC and AngularJS. 

It will be consist of a series of small posts as time goes by. Every post is a standalone chapter focused on a topic.

##Assumption

I assume you are a Java developer and have some experience of Spring framework.

Else you should learn the basic Java and Java EE knowledge, and master basic usage of Spring framework.

* The official [Oracle Java tutorial](https://docs.oracle.com/javase/tutorial/) and [Java EE tutorial](https://docs.oracle.com/javaee/7/tutorial) are ready for Java newbies.

* Read the [Spring official guides](https://spring.io/guides) to getting started with Spring framework.

In these posts, it will not cover all Spring and Java EE features, but the following technologies will be used.

* Spring framework

	Spring framework is the infrastructure framework of this sample application. 

	It provides a lightweight IOC container and a simple POJO based programming model, and also contains lots of glue codes for Java EE specification support and popular open source framework integration. 

	With the benifit of Spring, it makes Java EE development without container become true, and also eases the Java EE testing. In the past years, Spring was considered as the defacto standard of Java EE development.


* Spring MVC

	One of the most attractive features provided in Spring framework is the Spring MVC framework, like the old Struts framework, it is a web framework based on Servlet specification, and implements the standard MVC(Model, View, Controller) patterns. 

	Spring MVC supports lots of view presentations, for traditional web application or RESTful APIs. In this sample application, we only use Spring MVC as the REST API producer and exposes the APIs to client.

	For the traditional web development, check my samples hosted on [Spring4 sandbox](https://github.com/hantsy/spring4-sandbox).

* Spring Security

	In a traditional Java EE application, JAAS is the specification which is responsible for Authentication and Authorization. But it is very dependent on a specific container. Although most containers include a visual web UI for user maangement. But if you want to manage users and roles in program way.

	Spring Security fills this field, which makes the security control become easy, and provides a simple programming model. Spring Security is also compatible with JAAS specification, and provides JAAS integration at runtime. 

	Java EE 8 is trying to introduce a new Security specification to fix this issue.

* JPA

	Based on JDBC specification, JPA provides a high level ORM abstraction and brings OOP philosophy to interact with traditional RDBMS. Hibernate and EclipseLink also support NoSQL.

* Hibernate

	In this sample application, Hibernate is used as the JPA provider. Most of time, we are trying to avoid to use a provider specific APIs, make the codes can be run cross JPA providers.

* Spring Data JPA

	Spring Data JPA simplifies using JPA in Spring, including a united `Repository` to perform simple CRUD without coding, simplfied type safe Criteria Query and QueryDSL integration, a simple auditing implementation, simple pagination support of query result, Java 8 Optional and DateTime support etc.

	Check the Spring Data samples in [Spring4 sandbox](https://github.com/hantsy/spring4-sandbox).
	 
	
We also used some third party utilities, such as [Lombok project](https://projectlombok.org/) to remove the tedious getters and setters of POJOs. 

For testing purpose, Spring test/JUnit, Mockito, Rest Assured will be used.

##Smaple application

In order to demonstrate how to build RESTful APIs, I will implement a simple Blog system to explain it in details.

Imagine there are two roles will use this blog system.

* ROLE_ADMIN, the administrative user.
* ROLE_USER, the normal user.

A normal user can execute the most common tasks.

1. Create a new post.
2. Update post.
3. View post detail.
4. Search posts by keyword.
5. Delete posts.
6. Comment on posts.

A administrater should have more advanced permissions, eg. he can manage the system users.

1. Create a new user.
2. Update uesr
3. Delete user
4. Search users by keyword.

##Sample codes

The complete sample codes are hosted on my Github.com account.

[https://github.com/hantsy/angularjs-springmvc-sample](https://github.com/hantsy/angularjs-springmvc-sample)

A Spring Boot based envolved version provides more featuers to demonstrate the cutting-edge technologies.

[https://github.com/hantsy/angularjs-springmvc-sample-boot](https://github.com/hantsy/angularjs-springmvc-sample-boot)

Please read the README.md file in these respositories and run them under your local system.

##Feedback

The source of this book are hosted on my github.com account.

[https://github.com/hantsy/angularjs-springmvc-sample-gitbook](https://github.com/hantsy/angularjs-springmvc-sample-gitbook)

Feel free to provide feedback on this project.