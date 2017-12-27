# Handles Exceptions

In the real world applications, a user story can be described as different flows.

1. The execution works as expected and get success finally. 
2. Some conditions are not satisfied and the flow should be intercepted and notify users.

For example, when a user tries to register an account in this application. The server side should check if the existance of the input username, if the username is taken by other users, the server should stop the registration flow and wraps the message into `UsernameWasTakenException` and throws it. Later the APIs should translate it to client friend message and reasonable HTTP status code, and finally they are sent to client and notify the user.

## Define Exceptions

Define an exeption which stands for the exception path. For example, `ResourceNotFoundException` indicates an resource is not found in the applicatin when query the resource by id.

```java
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
```	


## Throws exceptions in service

In our service, check the resource existance, and throws the `ResourceNotFoundException` when the resource is not found.	

```java
public PostDetails findPostById(Long id) {

	Assert.notNull(id, "post id can not be null");

	log.debug("find post by id@" + id);

	Post post = postRepository.findOne(id);

	if (post == null) {
		throw new ResourceNotFoundException(id);
	}

	return DTOUtils.map(post, PostDetails.class);
}
```	

Here, we have defined ResourceNotFoundException as a `RuntimeException`, and there is no `throws` clause in the method declaration. Benefit from Spring IOC container, we do not need to use caught exception to force callers to handle it explictly. Alternatively, we can define a specific exception handler later to process it gracefully.
	
## Translates exceptions

Internally, Spring has a series of built-in `ExceptionTranslator`s to translate the exceptions to Spring declarative approaches, eg. all JDBC related exceptions are translated to Spring defined exceptions( `DataAccessException` and its subclasses), see [here](https://github.com/hantsy/spring-sandbox/wiki/data-access-support-overview) fore more details. 

In the presentation layer, these exceptions can be caught and converted into user friendly message.

Spring provides a built-in `ResponseEntityExceptionHandler` to handle the exceptions and translate them into REST API friendly messages.

You can extend this class and override the default exeption hanlder methods, or add your exception hanlder to handle custom exceptions.

```java
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
```		
	
In the above code, `ResourceNotFoundException` is handled by the `RestExceptionHandler`, and send a 404 HTTP status code to the client.

## Handles bean validation failure

For user input form validation, most of time, we could needs the detailed info the validaiton cosntraints. 

Spring supports JSR 303(Bean validation) natively, the bean validation constraints error can be gathered by `BindingResult` in the controller class.

```java
public ResponseEntity<ResponseMessage> createPost(@RequestBody @Valid PostForm post, BindingResult errResult) {

	log.debug("create a new post");
	if (errResult.hasErrors()) {
		throw new InvalidRequestException(errRusult);
	}
	//...
```		
		
In the controller class, if the `BindingResult` has errors, then wraps the error info into an exception.

Handles it in the `RestExceptionHandler`.

```java
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
```	
		
The detailed validation errors are wrapped as content and sent to the client, and a HTTP status to indicate the form data user entered is invalid.

## Exception and HTTP status

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

## Source Code

Check out sample codes from my github account.

```
git clone https://github.com/hantsy/angularjs-springmvc-sample
```
	
Or the Spring Boot version:

```
git clone https://github.com/hantsy/angularjs-springmvc-sample-boot
```
	
Read the live version of thess posts from Gitbook:[Building RESTful APIs with Spring MVC](https://www.gitbook.com/book/hantsy/build-a-restful-app-with-spring-mvc-and-angularjs/details).



	