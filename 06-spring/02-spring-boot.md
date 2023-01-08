# Spring Boot

* [Auto-Configuration](#auto-configuration)

Setting up the basis of an application is a cumbersome job. Configuration files for the project must be created, and additional tools (like an application server) must be installed and configured. Spring Boot is a Spring project that makes it easy to create stand-alone, production-grade Spring-based applications that you can just run. Spring Boot comes with out-of-the-box configurations for different types of Spring applications that are packed in starter packages. The web-starter package, for example, contains a preconfigured and easily customizable web application context and supports Tomcat 7+, Jetty 8+, and Undertow 1.3 embedded servlet containers out of the box. Spring Boot also wraps up all dependencies a Spring application needs, taking into account compatibility between versions.

### Auto-Configuration
The Spring Framework and some of its modules, such as Spring Data, Spring AMQP, and Spring Integration, provide **@Enable<Technology>** annotations; for example, **@EnableTransactionManagement, @EnableRabbit, and @EnableIntegration** are part of the modules mentioned. Within Spring applications, you can use these annotations to follow the convention over configuration pattern, making your apps easier to develop and maintain, and without worrying too much about its configuration. Spring Boot takes advantage of these annotations, which are used within the @EnableAutoConfiguration annotation to do the auto-configuration.

The auto-configuration classes are applied based on the classpath and which beans your app has defined, but what this makes more powerful is the **org.springframework.boot.autoconfigure.AutoConfigurationImportSelector** class that finds all the necessary configuration classes.

**AutoConfigurationImportSelector** class has the **getCandidateConfigurations** method that returns a **SpringFactoriesLoader.loadFactoryNames**. The SpringFactoriesLoader.loadFactoryNames looks for the **METAINF/spring.factories** defined in the spring-boot-autoconfigure jar.

The spring.factories define all the auto-configuration classes that are used to set up any configuration that your application needs for running.

Autoconfiguration classes use the **@ConditionalOnClass** and **@ConditionalOnMissingBean** annotations to decide if the application is of a particular type or not.

Another things to see  is the use of the **@ConditionalOnProperty**annotation, that only applies if a specific property is enabled. It’s worth mentioning that this auto-configuration can be also applied based on a profile enabled, denoted by the**@Profile** annotation. You need to understand that the autoconfiguration uses your classpath to figure out what to configure for your app. That’s why we say that Spring Boot is an opinionated runtime.