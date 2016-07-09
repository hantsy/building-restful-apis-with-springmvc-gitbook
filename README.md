Introduction
============

In this step-by-step minibook, I will demonstrate how to implement a RESTful web application with Spring MVC and AngularJS.


##Prerequisite

I assume you have some experience of Spring, especially Spring MVC.

In this sample application, the following technologies are used to build the backend REST API.

1. **Spring** framework is the infrastructure of the sample codes.
2. **JPA** is responsible of the data persistence, and **Hibernate** is chosen as the JPA provider in the sample code. **Spring Data(JPA)** is used to simplify the data operations.
3. **Spring MVC** is used to produce REST APIs.
4. **Spring Security** is the guard of this application and it will secure the REST APIs.

The frontend UI is consist of the static pages, which is written with AngularJS/Bootstrap. It will be a SPA(Single Page Application) application.

I will introduce you how to create a Blog sample application step by step in the following sections.

##Sample codes

The complete sample codes are hosted on my Github.com account.

[https://github.com/hantsy/angularjs-springmvc-sample](https://github.com/hantsy/angularjs-springmvc-sample)

Follow the following the steps to run this project.

  1. Clone the codes.

    <pre>
    git clone https://github.com/hantsy/angularjs-springmvc-sample
    </pre>
  
  2. And enter the root folder, run `mvn tomcat7:run` to start up an embedded tomcat7 to serve this application.
  
    <pre>
    mvn tomcat7:run
    </pre>

  3. Open your browser, and navigate [http://localhost:8080/angularjs-springmvc-sample/](http://localhost:8080/angularjs-springmvc-sample/).

