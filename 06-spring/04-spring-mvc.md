# Spring MVC

* [Spring Boot MVC Auto-Configuration](#spring-boot-mvc-auto-configuration)
* [Exception Handling](#exception-handling)

For the web technology, the Spring Framework has the spring-web, spring-webmvc, spring-webflux, and spring-websocket modules.

Starting with Servlet 3.0+ the web.xml file is no longer necessary to configure a web application. It can be replaced with a class implementing the org.springframework.web.WebApplicationInitializer interface (or a class extending any of the Spring classes that extend this interface). This class will be detected automatically by org. springframework.web.SpringServletContainerInitializer(an internal Spring supported class which is not meant to be used directly or extended), which itself is bootstrapped automatically by any Servlet 3.0+ container. The SpringServletContainerInitializer extends javax.servlet.ServletContainerInitializer and provides a Spring specific implementation for the onStartup method. This class is loaded and instantiated. The onStartup method is invoked by any Servlet 3.0–compliant container during the container’s startup, assuming that the spring-web module JAR is in the classpath.

The Spring MVC is designed around the org.springframework.web.servlet.DispatcherServlet class. This servlet is very flexible and has a very robust functionality that you won’t find in any other MVC web framework out there. With the DispatcherServlet, you have several out-of-the-box resolutions strategies, including view resolvers, locale resolvers, theme resolvers, and exception handlers. In other words, the DispatcherServlet take a HTTP request and redirect it to the right handler (the class marked with the @Controller or @RestController and the methods that use the @RequestMapping annotations) and the right view (your JSPs).

### Spring Boot MVC Auto-Configuration
Web applications can be created easily by adding the spring-boot-starter-web dependency to your pom.xml. 

Auto-configuration adds the following features to your web application.
* **Static content support** This means that you can add static content, such as HTML, JavaScript, CSS, media, and so forth, in a directory named /static (by default) or /public, /resources, or /META-INF/ resources, which should be in you classpath or in your current directory. Spring Boot picks it up and serves them upon request.
* **HttpMessageConverters** If you are using a regular Spring MVC application and you want to get a JSON response, you need to create the necessary configuration (XML or JavaConfig) for the HttpMessageConverters bean. Spring Boot adds this support by default so you don’t have to; this means that you get the JSON format by default (due to the Jackson libraries that the spring-boot- starter- web provides as dependencies).
* **JSON serializers and deserializers** If you want to have more control over the serialization/deserialization to/from JSON, Spring Boot provides an easy way to create your own by extending from JsonSerializer<T> and/or JsonDeserializer<T>, and annotating your class with the @JsonComponent so that it can be registered for usage.
* **Path matching and content negotiation** One of the Spring MVC application practices is the ability to respond to any suffix to represent the content-type response and its content negotiation.
* **Error handling** Spring Boot uses /error mapping to create a white labeled page to show all the global errors. If you are creating a RESTful application, Spring Boot responds as JSON format. Spring Boot also supports Spring MVC to handle errors when you are using @ControllerAdvice or @ExceptionHandler annotations.
* **Template engine support** Spring Boot supports FreeMarker, Groovy Templates, Thymeleaf, and Mustache. When you include the spring-boot-starter-\<template engine> dependency, Spring Boot autoconfigure is necessary to enable and add all the necessary view resolvers and file handlers. By default, Spring Boot looks at the src/main/resources/templates/ path.

### Exception Handling

```java
@ControllerAdvice
public class RestExceptionsHandler {
    @ExceptionHandler
    @ResponseBody
    public ResponseEntity<String> handleException(PersonsException pe) {
        return new ResponseEntity<>(pe.errorMessage(), pe.getStatus());
    }
}
```