# Inversion of Control and Dependency Injection

* [Dependency Pull](#dependency-pull)
* [Contextualized Dependency Lookup](#contextualized-dependency-lookup)
* [Constructor Dependency Injection](#constructor-dependency-injection)
* [Setter Dependency Injection](#setter-dependency-injection)
* [Dependency Injection in Spring](#dependency-injection-in-spring)
* [Understanding Bean Instantiation Mode](#understanding-bean-instantiation-mode)

At its core, IoC, and therefore DI, aims to offer a simpler mechanism for provisioning component dependencies (often referred to as an object’s collaborators) and managing these dependencies throughout their life cycles. A component that requires certain dependencies is often referred to as the dependent object or, in the case of IoC, the target. In general, IoC can be decomposed into two subtypes: dependency injection and dependency lookup. These subtypes are further decomposed into concrete implementations of the IoC services. From this definition, you can clearly see that when we are talking about DI, we are always talking about IoC, but when we are talking about IoC, we are not always talking about DI (for example, dependency lookup is also a form of IoC).

### Dependency Pull
To a Java developer, dependency pull is the most familiar types of IoC. In dependency pull, dependencies are pulled from a registry as required.

```
public class DependencyPull {
    public static void main(String... args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("spring/app-context.xml");
        MessageRenderer mr = ctx.getBean("renderer", MessageRenderer.class);
        mr.render();
    }
}
```

### Contextualized Dependency Lookup
Contextualized dependency lookup (CDL) is similar, in some respects, to dependency pull, but in CDL, lookup is performed against the container that is managing the resource, not from some central registry, and it is usually performed at some set point.

### Constructor Dependency Injection
Constructor dependency injection occurs when a component’s dependencies are provided to it in its constructor (or constructors). The component declares a constructor or a set of constructors, taking as arguments its dependencies, and the IoC container passes the dependencies to the component when instantiation occurs

### Setter Dependency Injection
In setter dependency injection, the IoC container injects a component’s dependencies via JavaBean-style setter methods. A component’s setters expose the dependencies the IoC container can manage.

### Dependency Injection in Spring
The core of Spring’s dependency injection container is the **BeanFactory** interface. BeanFactory is responsible for managing components, including their dependencies as well as their life cycles. In Spring, the term bean is used to refer to any component managed by the container.

Although the BeanFactory can be configured programmatically, it is more common to see it configured externally using some kind of configuration file. Internally, bean configuration is represented by instances of classes that implement the BeanDefinition interface. The bean configuration stores information not only about a bean itself but also about the beans that it depends on. For any BeanFactory implementation classes that also implement the BeanDefinitionReader interface, you can read the BeanDefinition data from a configuration file, using either PropertiesBeanDefinitionReader or XmlBeanDefinitionReader. PropertiesBeanDefinitionReader reads the bean definition from properties files, while XmlBeanDefinitionReader reads from XML files.

**DefaultListableBeanFactory** is one of the two main BeanFactory implementations supplied with Spring, and that we are reading in the BeanDefinition information from an XML file by using **XmlBeanDefinitionReader**. Once the BeanFactory implementation is created and configured it can be used to lookup beans.

In Spring, the **ApplicationContext** interface is an extension to BeanFactory. In addition to DI services, ApplicationContext provides other services, such as transaction and AOP service, message source for internationalization (i18n), and application event handling, to name a few. In developing Spring-based applications, it’s recommended that you interact with Spring via the ApplicationContext interface. Spring supports the bootstrapping of ApplicationContext by manual coding (instantiate it manually and load the appropriate configuration) or in a web container environment via ContextLoaderListener.

The **GenericXmlApplicationContext** class implements the ApplicationContext interface and is able to bootstrap Spring’s ApplicationContext from the configurations defined in XML files.

The **AnnotationConfigApplicationContext** class implements the ApplicationContext interface and is able to bootstrap Spring’s ApplicationContext from the configurations defined by the Configuration class.

### Understanding Bean Instantiation Mode
The following bean scopes are supported as of version 4:
* **Singleton**: The default singleton scope. Only one object will be created per Spring IoC container.
* **Prototype**: A new instance will be created by Spring when requested by the application.
* **Request**: For web application use. When using Spring MVC for web applications, beans with request scope will be instantiated for every HTTP request and then destroyed when the request is completed.
* **Session**: For web application use. When using Spring MVC for web applications, beans with session scope will be instantiated for every HTTP session and then destroyed when the session is over.
* **Global session**: For portlet-based web applications. The global session scope beans can be shared among all portlets within the same Spring MVC–powered portal application.
* **Thread**: A new bean instance will be created by Spring when requested by a new thread, while for the same thread, the same bean instance will be returned. Note that this scope is not registered by default.
* **Custom**: Custom bean scope that can be created by implementing the interface org.springframework.beans.factory.config.Scope and registering the custom scope in Spring’s configuration (for XML, use the class org.springframework. beans.factory.config.CustomScopeConfigurer).