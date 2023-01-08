# Spring Annotations

* [Spring DI](#spring-di)
* [Spring MVC](#spring-mvc)
* [Spring Data JPA](#spring-data-jpa)

### Spring DI
* **@ImportResource(locations = {"classpath:spring/app-context-xml.xml"})** - import bean definitions from an XML file
* **@Configuration**
* **@ComponentScan(basePackages = {"com.apress.prospring5.ch3.annotation"})**
* **@Service("renderer")**
* **@Component**
* **@Autowired**
* **@Value("Configurable message")** - the value to be injected into the constructor. This is the way in Spring we inject values into a bean
* **@Scope("prototype")**
* **@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)**
* **@DependsOn("gopher")**
* **@Lazy** - better not to use this because it defers dependency injection and missing bean errors might pop up late
* **@Primary**
* **@PropertySource(value = "classpath:message.properties")**
* **@SpringBootApplication(exclude={ActiveMQAutoConfiguration.class,DataSourceA utoConfiguration.class})**

### Spring MVC
* **@ControllerAdvice**
* **@ExceptionHandler({NotFoundException.class})**
* **@ResponseStatus(HttpStatus.NO_CONTENT)**
* **@ResponseBody**

### Spring Data JPA
* **@RepositoryDefinition(domainClass = Detective.class, idClass = Long.class)**

