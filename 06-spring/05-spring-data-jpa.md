# Spring Data JPA

* [JDBC and JPA with Spring Boot](#jdbc-and-jpa-with-spring-boot)
* [Transaction management](#transaction-management)
* [Introducing Hibernate and ORM](#introducing-hibernate-and-orm)
* [Configure Spring and JPA with Hibernate Support](#configure-spring-and-jpa-with-hibernate-support)
* [Spring Data JPA](#spring-data-jpa-repositories)

### JDBC and JPA with Spring Boot
The Spring Framework has the support for working with SQL databases either using JDBC or ORMs. Spring Boot brings even more to data applications. Spring Boot uses auto-configuration to set sensible defaults when it finds out that your application has a JDBC JARs. Spring Boot auto-configures the datasource based on the SQL driver in your classpath. If it finds that you have any of the embedded database engines (H2, HSQL or Derby), it is configured by default; in other words, you can have two driver dependencies (e.g., MySQL and H2), and if Spring Boot doesn’t find any declared datasource bean, it creates it based on the embedded database engine JAR in your classpath (e.g., H2). Spring Boot also configures HikariCP as connection pool manager by default. Of course, you can override these defaults. If you want to override the defaults, then you need to provide your own datasource declaration, either JavaConfig, XML, or in the application.properties file.

In a simple Spring app, you are required to use the @EnableJpaRepositories annotation that triggers the extra configuration that is applied in the life cycle of the repositories defined within of your application. The good thing is that you don’t need this when using Spring Boot because Spring Boot takes care of it. Another feature from Spring Data JPA are the query methods, which are a very powerful way to create SQL statements with the fields of the domain entities. So, to use Spring Data JPA with Spring Boot, you need spring-boot-starter-datajpa and the SQL driver. When Spring Boot executes its auto-configuration and finds out that you have the Spring Data JPA JAR, it configures the datasource by default (if there is none defined). It configures the JPA provider (by default it uses Hibernate). It enables the repositories(by using the @EnableJpaRepositories configuration). It checks if you have defined any query methods. And more.

### Transaction management
Transaction management is implemented in Spring by making use under the hood of AOP, which is because transactions are a cross-cutting concern. Basically, for every method that must be executed in a transaction, retrieving or opening a transaction before execution and committing it afterward is necessary. For beans that have methods that must be executed in a transactional context, AOP proxies are created that wrap the methods in an Around advice that takes care of getting a transaction before calling the method and committing the transaction afterward. The AOP proxies use two infrastructure beans for this:an org.springframework.transaction.interceptor.TransactionInterceptor in conjunction with an implementation of org.springframework.transaction. PlatformTransactionManager. Spring provides a flexible and powerful abstraction layer for transaction management support. At the core of it is the PlatformTransactionManager interface. Any transaction manager provider framework that is supported can be used.

The simplest way to add transaction management to the application is to do the following.
* Configure transaction management support. Add a declaration of a bean of type org.springframework.jdbc.datasource.DataSourceTransactionManager in a configuration class.
* Activate it with the @EnableTransactionManagement annotation.
* Declare transactional methods with the Spring @Transactional annotation.

### Introducing Hibernate and ORM
To configure JPA with Hibernate in a Spring application, the following components must be introduced.
* The **org.hibernate.SessionFactory** interface is the core component of Hibernate. An object of this type is thread-safe, shareable, and immutable. Usually, an application has a single SessionFactory instance, and threads servicing client requests obtain Session instances from this factory. Once a SessionFactory instance is created, its internal state is set. This internal state includes all the metadata about object relational mapping.
* The **org.hibernate.Session** interface is the hibernate component representing a single functional unit, and it is the main runtime interface between a Java application and Hibernate. The session is a stateful object that manages persistent objects within the functional unit. It acts as a transactional-scoped cache, operations executed in a session are cached, and the changes are persisted to the datasource when the transaction is committed. A Session object can be obtained by calling sessionFactory.getCurrentSession().
* **org.springframework.orm.hibernate5.HibernateTransactionManager** this class is an implementation of PlatformTransactionManager for a single SessionFactory.
* **org.springframework.orm.hibernate5.LocalSessionFactoryBuilder** this is a utility class that can create a SessionFactory bean. The session factory bean requires as parameters the datasource used by the application, the package where the entity classes can be found, and the hibernate properties.
* Because Hibernate is an advanced tool designed to be used with advanced datasource settings, it is time to introduce connection pooling. For this section, HikariCP was chosen to create the dataSource bean. HikariCP is open source, small, and practical, since only one library needs to be added to the project, and it is said to be the fastest connection pool in the Java universe

The SessionFactory bean is then injected into repositories and creates objects of types implementing org.hibernate.query.Query<R>, which are executed and the result is returned.

Entities are manipulated by Hibernate Session instances that provide methods for search, persist, update, delete. This instance is also in charge of managing transactions and is a good replacement for JPA’s EntityManager. The current session is obtained from the SessionFactory bean by calling sessionFactory.getCurrentSession(); The queries for these operations are written in Hibernate Query Language, which allows a more practical way of writing the SQL queries. HQL queries operate on domain objects and are transformed under the hood into matching SQL queries.

### Configure Spring and JPA with Hibernate Support
* The SessionFactory bean declaration is no longer needed, and it will be replaced by a bean declaring an Entity Manager Factory bean. The Spring-specific **LocalContainerEntityManagerFactoryBean** class will be used for this.
```
@Bean
public EntityManagerFactory entityManagerFactory(){
    LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
    factoryBean.setPackagesToScan("com.apress.cems.dao");
    factoryBean.setDataSource(dataSource());
    factoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    factoryBean.setJpaProperties(hibernateProperties());
    factoryBean.afterPropertiesSet();
    return factoryBean.getNativeEntityManagerFactory();
}
```
* HibernateTransactionManager is replaced by a bean of type**JpaTransactionManager** that uses the **EntityManagerFactory** implementation to associate Entity Manager operations with transactions. 
* The repository classes will be modified to use an instance of type EntityManager mapped to the application persistence context. The annotation **@PersistenceContext** expresses a dependency on a container-managed EntityManager and its associated persistence context.

### Spring Data JPA Repositories
Spring Data is a Spring project designed to help defining repository classes in a more practical way. Repository classes have a lot of common functionality, so the Spring team tried to provide the possibility to offer this functionality out of the box. The solution was to introduce abstract repositories that can also be customized by the developer to reduce the boilerplate code required for data access. To use Spring Data components in a JPA project, a dependency on the package spring-data-jpa must be introduced. The central interface of Spring Data is **Repository<T,ID extends Serializable>**.

If all that is needed is to extend the **Repository<T,ID extends Serializable>** interface, but you do not like the idea of extending a Spring component, you can avoid that by annotating your repository class with **@RepositoryDefinition(domainClass = Detective.class, idClass = Long.class)**.

The **CrudRepository<T, ID extends Serializable>** is the interfaces that exposes all basic operations for an entity type, even the name makes the purpose obvious.

But, for more advanced operations you might want to go further in the hierarchy, which brings us to the **PagingAndSortingRepository<T, ID extends Serializable>** interfaces that support entity pagination. This interface exposes a special version of the findAll(..) method that receives as a parameter an instance of type Pageable that is used for pagination information.