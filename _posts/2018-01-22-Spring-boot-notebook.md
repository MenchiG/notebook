---
layout: post
title: Spring Boot Notebook
key: 10004
tags: Spring Java note
category: blog
---

Some notes for Spring Boot and Spring Cloud<!--more-->


## Annotation

### Spring Boot

`@SpringBootApplication`

### JPA(Java persistent API)

`@Embeddable` vs `@Entity`

In OO relationship

* a dependent object is considered an aggregate or composite association

In relational model

* The dependent object could have its own table

* Its data could be embedded in the independent object's table. The dependent data is included in the independent object's table.

Assume we have **user** and **address**, if query **address** separately is necessary, **address** should stored in another table, otherwise it can be embedded in **user**.

### Message Queue

`@EnableBinding(Source.class)`

​	Message publisher

`@EnableBinding(Sink.class)`

`@ServiceActivator(inputChannel = Sink.INPUT)`

​	Message consumer

### Eureka

Use service name as url instead of practice address and port number

`@EnableDiscoveryClient`

​	Register to Eureka

### Hystrix

`@EnableCircuitBreaker`

`@HystrixCommand`

### Utility

`@Slf4j`

​	Simple log, use object log instead of logger factory

@Data

​	lombok for setter and getter

## Parent POM

modules: share dependency Management

## Design Skill

### Class Injection

In `application.yml`

```yaml
key: 1
appId: 1
```

Class Defination

```java
public class TestClass {

    private String appKey;

    private String appId;

    public static class Builder{

        private String appKey;

        private String appId;

        public Builder setKey(String appKey){
            this.appKey = appKey;
            return this;
        }

        public Builder setAppId(String appId){
            this.appId = appId;
            return this;
        }

        public TestClass build(){
            return new TestClass(this);
        }
    }

    public static Builder options(){
        return new TestClass.Builder();
    }

    private TestClass(Builder builder){
        this.appKey = builder.appKey;
        this.appId = builder.appId;
    }

    public String getAppKey() {
        return appKey;
    }

    public String getAppId() {
        return appId;
    }
}
```

Then add code in a class annotated by `@SpringBootConfiguration`

```java
	@Value("${appKey}")
	private String appKey;
	@Value("${appId}")
	private String appId;

	@Bean
	public TestClass testClass(){
        return TestClass.options()
                .setAppKey(appKey)
                .setAppId(appId)
                .build();
    }
```

When use TestClass

```java
	@Autowired
	private TestClass testClass;
```

### Interceptor

Create a new interceptor class implements `HandlerInterceptor`

```java
public class ApiInterceptor implements HandlerInterceptor {
    //Before Request
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("Going into interceptor");
        return true;
    }
    //Requesting
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }
    //After Request
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

Class with `@SpringBootConfiguration` extends `WebMvcConfigurationSupport`, and overrides `addInterceptors` method to add `ApiInterceptor` interceptor class

```java
@SpringBootConfiguration
public class WebConfig extends WebMvcConfigurationSupport{

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
        registry.addInterceptor(new ApiInterceptor());
    }
}
```

### Handle Exception

```java
@Aspect
@Component
public class WebExceptionAspect {

    private static final Logger logger = LoggerFactory.getLogger(WebExceptionAspect.class);

	//method with @RequestMapping will be intercepted   
	@Pointcut("@annotation(org.springframework.web.bind.annotation.RequestMapping)")
    private void webPointcut() {
    }

    /**
     * Intercetpt web layer exception，log，and response
     *
     * @param e 
     *       		exception object     
     */
    @AfterThrowing(pointcut = "webPointcut()", throwing = "e")
    public void handleThrowing(Exception e) {
        e.printStackTrace();
        logger.error("Exception！" + e.getMessage());
        logger.error(JSON.toJSONString(e.getStackTrace()));
        //Input friendly content
        writeContent("Exception");
    }

    /**
     * Output to browser
     *
     * @param content
     *            output content
     */
    private void writeContent(String content) {
        HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
                .getResponse();
        response.reset();
        response.setCharacterEncoding("UTF-8");
        response.setHeader("Content-Type", "text/plain;charset=UTF-8");
        response.setHeader("icop-content-type", "exception");
        PrintWriter writer = null;
        try {
            writer = response.getWriter();
        } catch (IOException e) {
            e.printStackTrace();
        }
        writer.print(content);
        writer.flush();
        writer.close();
    }
}
```

### Validation

In traditional ways, `BindingResult` is necessary but it will throw exception

```java
@GetMapping("authorize")
public void authorize(@Valid AuthorizeIn authorize, BindingResult ret){
    if(result.hasFieldErrors()){
            List<FieldError> errorList = result.getFieldErrors();
            //Throw Excetpion by assert
            errorList.stream().forEach(item -> Assert.isTrue(false,item.getDefaultMessage()));
        }
}

public class AuthorizeIn extends BaseModel{

    @NotBlank(message = "need response_type")
    private String responseType;
    @NotBlank(message = "need client_id")
    private String ClientId;

    private String state;

    @NotBlank(message = "need redirect_uri")
    private String redirectUri;

    public String getResponseType() {
        return responseType;
    }

    public void setResponseType(String responseType) {
        this.responseType = responseType;
    }

    public String getClientId() {
        return ClientId;
    }

    public void setClientId(String clientId) {
        ClientId = clientId;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public String getRedirectUri() {
        return redirectUri;
    }

    public void setRedirectUri(String redirectUri) {
        this.redirectUri = redirectUri;
    }
}
```

So it is better to use Handle Exception method.

```java
	//Intercept @GetMapping method   
	@Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
    private void webPointcut() {
    }

    @AfterThrowing(pointcut = "webPointcut()",throwing = "e")
    public void afterThrowing(Exception e) throws Throwable {
        logger.debug("exception！");
        if(StringUtils.isNotBlank(e.getMessage())){
                           writeContent(e.getMessage());
        }else{
            writeContent("Parameter Error！");
        }
    }
    
    private void writeContent(String content) {
    	//output
    }
```

Encaplulate `validate` method
```java
protected void validate(BindingResult result){
        if(result.hasFieldErrors()){
            List<FieldError> errorList = result.getFieldErrors();
            errorList.stream().forEach(item -> Assert.isTrue(false,item.getDefaultMessage()));
        }
    }
```

## URL

* Use service name, do not need port number
* Inject url in yml files (@Value)  need port number

## Debug

### 1.1 mvn clean install

error -> compile time error -> syntax error, dependency issue

### 2.1 service1

#### 2.1.1 Spring configuration issue.

@Autowired Service1 service1 spring startup failed xxx bean cannot be injected

#### 2.1.2 Address used

change port number

#### 2.1.3 Runtime Dependency issue 

ClassNotFound Exception. ClassCastException, NotFoundException

* Look up whether this dependency can be managed by Spring Boot or Spring Cloud	

* check document and change version number

### 3.1  service1, service2. Service1 -> Service 2 Error

Logging and logging

#### 3.1.1 Before service 1 call service 2

do logging

#### 3.1.2 Made call but got error response.

After service call finished, do logging . service2 network issue

#### 3.1.3 ELK

Splunk

ELK = Elastic Search + Logstash + Kibana

log stream -> Kafka -> log message logging service -> data store, indexing, search