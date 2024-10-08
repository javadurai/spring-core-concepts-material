Lazy Initialization is a strategic approach in software development, particularly within the Spring Framework, that delays the creation or loading of an object until it is explicitly needed. This technique optimizes resource utilization, enhances application performance, and reduces the startup time by avoiding the unnecessary instantiation of objects that may never be used during the application's lifecycle.

This comprehensive guide delves into the concept of Lazy Initialization, exploring its significance, implementation strategies within Spring, benefits, potential drawbacks, best practices, and practical examples to provide a thorough understanding of how to leverage this feature effectively in your Spring-based applications.

# Introduction to Lazy Initialization

In the realm of software development, Lazy Initialization refers to a design pattern that defers the creation of an object, the calculation of a value, or some other expensive process until the first time it is needed. This approach contrasts with Eager Initialization, where resources are allocated and objects are created upfront, regardless of their immediate necessity.

Within the Spring Framework, Lazy Initialization plays a crucial role in optimizing application performance, managing resources efficiently, and improving startup times by ensuring that beans are instantiated only when required. This strategy is particularly beneficial in large-scale applications with numerous beans, where unnecessary instantiation can lead to increased memory consumption and longer initialization periods.

# Core Concepts

## What is Lazy Initialization?

Lazy Initialization is a strategy where the creation or loading of an object is postponed until it is actually needed. This means that the object is not created at the time of application startup or during the initial loading phase but rather at the moment its functionality is invoked.

In the context of the Spring Framework, Lazy Initialization pertains to the instantiation of beans within the Application Context. By default, Spring eagerly instantiates singleton beans at startup, but with Lazy Initialization, their creation can be deferred, allowing for more efficient resource utilization.

## Eager vs. Lazy Initialization

| Aspect               | Eager Initialization                                     | Lazy Initialization                                          |
| -------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| Definition           | Beans are instantiated at application startup.           | Beans are instantiated only when needed.                     |
| Startup Time         | Longer, as all beans are created upfront.                | Shorter, as only necessary beans are created.                |
| Resource Utilization | Higher memory consumption due to all beans being loaded. | Lower memory consumption, as only required beans are loaded. |

## Benefits of Lazy Initialization

1. **Improved Startup Performance:**
   By deferring bean creation, the application can start faster, especially when dealing with a large number of beans.

1. **Optimized Resource Usage:**
   Resources are allocated only for beans that are actually used, reducing memory overhead.

1. **Enhanced Scalability:**
   Applications can handle more extensive configurations and dependencies without being bogged down by unnecessary instantiation.

1. **Reduced Initial Load:**
   Minimizes the load on the system during the startup phase, distributing the resource demand over time as beans are accessed.

## Potential Drawbacks

1. **Delayed Failure Detection:**
   Errors related to bean creation may surface later during runtime rather than at startup, potentially complicating debugging.

1. **Concurrency Issues:**
   In multi-threaded environments, lazy initialization might introduce synchronization challenges if multiple threads attempt to instantiate the same bean simultaneously.

1. **Increased Complexity:**
   Managing the lifecycle and dependencies of lazily initialized beans can add complexity to the application's configuration.

1. **Performance Overhead on First Access:**
   The first access to a lazily initialized bean incurs the cost of instantiation, which might lead to latency spikes in critical paths.

# Lazy Initialization in Spring Framework

Spring provides robust support for Lazy Initialization, allowing developers to fine-tune when beans are instantiated based on application requirements.

## Default Bean Initialization Behavior

By default, Spring employs Eager Initialization for singleton beans. This means that as soon as the Application Context is loaded, all singleton beans are created, their dependencies are injected, and any initialization callbacks are executed.

**Implications:**

- **Predictable Behavior:** All beans are ready for use once the context is loaded.
- **Immediate Resource Allocation:** Beans consume resources right at startup, which might be unnecessary if some beans are never used.

## Configuring Lazy Initialization

Spring offers multiple ways to configure Lazy Initialization, providing flexibility to suit various application architectures.

### Using Annotations

The @Lazy annotation is a straightforward way to indicate that a bean should be lazily initialized. It can be applied at both the bean class level and the injection point.

1.  Applying @Lazy at the Bean Class Level:

    ```java
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    @Lazy
    public class LazySingletonBean {
        public LazySingletonBean() {
            System.out.println("LazySingletonBean instantiated.");
        }
    }
    ```

2.  Applying @Lazy at the Injection Point:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final LazySingletonBean lazySingletonBean;

        @Autowired
        public ClientBean(@Lazy LazySingletonBean lazySingletonBean) {
            this.lazySingletonBean = lazySingletonBean;
        }

        public void performAction() {
            System.out.println("ClientBean is performing an action.");
            lazySingletonBean.doWork();
        }
    }
    ```

    **Explanation:**

- When @Lazy is applied at the bean class level, the entire bean is lazily initialized.
- When @Lazy is applied at the injection point, the bean is injected as a proxy, and the actual instantiation is deferred until a method on the bean is invoked.

### Using XML Configuration

For applications using XML-based configurations, the lazy-init attribute can be set to true for individual bean definitions or globally.

1.  Setting lazy-init for a Specific Bean:

    ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="lazyBean" class="com.example.LazyBean" lazy-init="true"/>

    </beans>
    ```

2.  Setting lazy-init Globally:

    ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd"
        default-lazy-init="true">

        <bean id="bean1" class="com.example.Bean1"/>
        <bean id="bean2" class="com.example.Bean2"/>

    </beans>
    ```

    **Explanation:**

- Setting lazy-init="true" for a bean ensures it is instantiated only when required.
- Setting default-lazy-init="true" at the <beans> level makes all beans lazy by default unless explicitly overridden.

### Using Java-Based Configuration

With Spring's Java-based configuration, Lazy Initialization can be controlled using the @Bean annotation's lazyInit attribute or by applying the @Lazy annotation.

1.  Using @Bean with lazyInit:

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class AppConfig {

        @Bean(lazyInit = true)
        public LazyBean lazyBean() {
            return new LazyBean();
        }
    }
    ```

2.  Combining @Lazy with @Bean:

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Lazy;

    @Configuration
    public class AppConfig {

        @Bean
        @Lazy
        public LazyBean lazyBean() {
            return new LazyBean();
        }
    }
    ```

    **Explanation:**

- The lazyInit attribute within the @Bean annotation specifies whether the bean should be lazily initialized.
- The @Lazy annotation provides a declarative way to achieve the same, offering flexibility and readability.

## Lazy Initialization of the Entire Application Context

In addition to configuring individual beans, Spring allows for the Lazy Initialization of the entire Application Context. This approach delays the instantiation of all singleton beans until they are first accessed.

1. Enabling Global Lazy Initialization:

   - Spring Boot (Version 2.2 and Above):

     Spring Boot introduced a global Lazy Initialization feature, which can be enabled via configuration properties.

     application.properties:

     ```properties
     spring.main.lazy-initialization=true
     ```

     **Explanation:**

     - Setting spring.main.lazy-initialization=true ensures that all singleton beans are lazily initialized by default.

   - Java Configuration:

     ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;

     @SpringBootApplication
     public class Application {

         public static void main(String[] args) {
             SpringApplication app = new SpringApplication(Application.class);
             app.setLazyInitialization(true);
             app.run(args);
         }
     }
     ```

2. Considerations:

   - **Performance Trade-offs:** While global Lazy Initialization can reduce startup time, it might introduce latency when beans are first accessed.
   - **Dependency Chains:** Complex dependencies might complicate the Lazy Initialization process, leading to potential issues if not managed correctly.

# Practical Examples

To solidify the understanding of Lazy Initialization in Spring, let's explore several practical examples demonstrating different configurations and use cases.

## Example 1: Lazy Initialization with @Lazy Annotation

**Scenario:**

Implement Lazy Initialization for a singleton bean to defer its instantiation until it is first accessed.

**Implementation:**

1.  Define the Lazy Bean:

    ```java
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    @Lazy
    public class ExpensiveService {
        public ExpensiveService() {
            System.out.println("ExpensiveService instantiated.");
        }

        public void performTask() {
            System.out.println("ExpensiveService is performing a task.");
        }
    }
    ```

1.  Define a Client Bean:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ExpensiveService expensiveService;

        @Autowired
        public ClientBean(ExpensiveService expensiveService) {
            this.expensiveService = expensiveService;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing a task.");
            expensiveService.performTask();
        }
    }
    ```

1.  **Main Application:**

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  **Console Output:**

        ```csharp
        Application started.
        ClientBean instantiated.
        ExpensiveService instantiated.
        ClientBean is executing a task.
        ExpensiveService is performing a task.
        ```

    **Explanation:**

- The ExpensiveService bean is annotated with @Lazy, ensuring it is not instantiated at startup.
- When the ClientBean is instantiated, the ExpensiveService bean is not created immediately.
- Upon invoking clientBean.execute(), the ExpensiveService bean is instantiated at that moment, demonstrating Lazy Initialization.

## Example 2: Global Lazy Initialization

**Scenario:**

Configure the entire Spring Application Context to lazily initialize all singleton beans, reducing the application's startup time.

**Implementation:**

1.  Enable Global Lazy Initialization via application.properties:

    ```properties
    spring.main.lazy-initialization=true
    ```

1.  Define Multiple Beans:

    ```java
    import org.springframework.stereotype.Component;

    @Component
    public class ServiceA {
        public ServiceA() {
            System.out.println("ServiceA instantiated.");
        }

        public void doWork() {
            System.out.println("ServiceA is working.");
        }
    }

    @Component
    public class ServiceB {
        public ServiceB() {
            System.out.println("ServiceB instantiated.");
        }

        public void doWork() {
            System.out.println("ServiceB is working.");
        }
    }
    ```

1.  Define a Client Bean:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ServiceA serviceA;
        private final ServiceB serviceB;

        @Autowired
        public ClientBean(ServiceA serviceA, ServiceB serviceB) {
            this.serviceA = serviceA;
            this.serviceB = serviceB;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing tasks.");
            serviceA.doWork();
            serviceB.doWork();
        }
    }
    ```

1.  Main Application:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  Console Output:

    ```csharp
    Application started.
    ClientBean instantiated.
    ServiceA instantiated.
    ServiceB instantiated.
    ClientBean is executing tasks.
    ServiceA is working.
    ServiceB is working.
    ```

    **Explanation:**

- By enabling global Lazy Initialization (spring.main.lazy-initialization=true), all singleton beans (ServiceA, ServiceB, ClientBean) are instantiated only when they are first accessed.
- The beans are not created at application startup, resulting in faster initialization.
- The beans are instantiated in the order they are accessed within the ClientBean.execute() method.

## Example 3: Combining @Lazy with Dependency Injection

**Scenario:**

Use the @Lazy annotation in combination with dependency injection to optimize the instantiation of dependencies within a singleton bean.

**Implementation:**

1.  Define an Expensive Service:

    ```java
    import org.springframework.stereotype.Component;

    @Component
    public class ExpensiveService {
        public ExpensiveService() {
            System.out.println("ExpensiveService instantiated.");
        }

        public void performExpensiveOperation() {
            System.out.println("ExpensiveService is performing an expensive operation.");
        }
    }
    ```

1.  Define a Client Bean with Lazy Dependency:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ExpensiveService expensiveService;

        @Autowired
        public ClientBean(@Lazy ExpensiveService expensiveService) {
            this.expensiveService = expensiveService;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing.");
            expensiveService.performExpensiveOperation();
        }
    }
    ```

1.  Main Application:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  Console Output:

    ```csharp
    Application started.
    ClientBean instantiated.
    ClientBean is executing.
    ExpensiveService instantiated.
    ExpensiveService is performing an expensive operation.
    ```

    **Explanation:**

- The ExpensiveService bean is annotated with @Lazy at the injection point within ClientBean.
- This configuration ensures that ExpensiveService is not instantiated when ClientBean is created.
- The ExpensiveService is instantiated only when clientBean.execute() invokes a method on it, demonstrating deferred instantiation.

# Advanced Topics

## Lazy Initialization with @DependsOn

The @DependsOn annotation can be used in conjunction with Lazy Initialization to manage bean dependencies explicitly. It ensures that certain beans are initialized before others, even if they are lazily initialized.

**Example:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.DependsOn;
import org.springframework.stereotype.Component;

@Component
@DependsOn("initializingBean")
@Lazy
public class DependentBean {

    private final InitializingBean initializingBean;

    @Autowired
    public DependentBean(InitializingBean initializingBean) {
        this.initializingBean = initializingBean;
        System.out.println("DependentBean instantiated.");
    }

    public void useInitializingBean() {
        initializingBean.initialize();
    }
}

@Component
public class InitializingBean {
    public InitializingBean() {
        System.out.println("InitializingBean instantiated.");
    }

    public void initialize() {
        System.out.println("InitializingBean is initializing.");
    }
}
```

**Explanation:**

- The DependentBean depends on InitializingBean.
- Even though DependentBean is lazily initialized, InitializingBean is instantiated first due to the @DependsOn annotation.
- This ensures that dependencies are correctly initialized in the desired order.

## Lazy Initialization and Proxies

When a bean is lazily initialized, Spring often uses proxies to handle the deferred creation. This allows the application to interact with a placeholder until the actual bean is instantiated upon first use.

**Types of Proxies:**

1. JDK Dynamic Proxies:

   - Used when the bean implements at least one interface.
   - Proxy implements the same interfaces.

1. CGLIB Proxies:
   - Used when the bean does not implement any interfaces.
   - Proxy subclasses the actual bean class.

**Example:**

```java
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
@Lazy
public class NonInterfaceBean {
    public NonInterfaceBean() {
        System.out.println("NonInterfaceBean instantiated.");
    }

    public void performAction() {
        System.out.println("NonInterfaceBean is performing an action.");
    }
}
```

**Client Bean:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ProxyClientBean {

    private final NonInterfaceBean nonInterfaceBean;

    @Autowired
    public ProxyClientBean(NonInterfaceBean nonInterfaceBean) {
        this.nonInterfaceBean = nonInterfaceBean;
        System.out.println("ProxyClientBean instantiated.");
    }

    public void execute() {
        System.out.println("ProxyClientBean is executing.");
        nonInterfaceBean.performAction();
    }
}
```

**Console Output:**

```csharp
ProxyClientBean instantiated.
ProxyClientBean is executing.
NonInterfaceBean instantiated.
NonInterfaceBean is performing an action.
```

**Explanation:**

- NonInterfaceBean is lazily initialized and does not implement any interfaces, so Spring uses a CGLIB proxy.
- The proxy intercepts calls to performAction(), triggering the instantiation of NonInterfaceBean at that moment.

## Interaction with Spring Profiles

Lazy Initialization can be combined with Spring Profiles to control bean instantiation based on the active profile, providing granular control over application behavior in different environments.

**Example:**

```java
import org.springframework.context.annotation.Profile;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
@Profile("development")
@Lazy
public class DevOnlyBean {
    public DevOnlyBean() {
        System.out.println("DevOnlyBean instantiated.");
    }

    public void performDevTask() {
        System.out.println("DevOnlyBean is performing a development task.");
    }
}
```

**Explanation:**

- DevOnlyBean is only active in the development profile and is lazily initialized.
- This ensures that development-specific beans do not consume resources in production environments unless explicitly required.

# Best Practices for Lazy Initialization

Implementing Lazy Initialization effectively requires adherence to best practices to maximize its benefits while minimizing potential drawbacks.

1. **Use Lazy Initialization Judiciously:**
   Apply Lazy Initialization to beans that are resource-intensive or not always required.
   Avoid overusing it, as unnecessary lazy beans can complicate the application flow.

1. **Combine with Spring Profiles:**
   Use Spring Profiles to manage Lazy Initialization in different environments, ensuring that beans are instantiated only when necessary based on the deployment context.

1. **Leverage Scoped Proxies:**
   When injecting lazily initialized beans into singleton beans, use scoped proxies to handle lifecycle management correctly and prevent premature instantiation.

1. **Ensure Thread Safety:**
   For beans that might be accessed by multiple threads, ensure that their instantiation is thread-safe to prevent concurrency issues.

1. **Monitor Application Performance:**
   Use profiling tools to monitor the impact of Lazy Initialization on application performance, adjusting configurations as needed to balance startup time and runtime performance.

1. **Handle Exceptions Gracefully:**
   Implement robust error handling for bean instantiation failures that occur during Lazy Initialization to prevent unexpected application crashes.

1. **Document Bean Scopes and Initialization Strategies:**
   Clearly document which beans are lazily initialized and the reasons behind their configurations to aid future maintenance and development efforts.

1. **Test Thoroughly:**
   Implement comprehensive tests to ensure that lazily initialized beans behave as expected across various scenarios and that their dependencies are correctly managed.

1. **Avoid Circular Dependencies:**
   Be cautious of circular dependencies when using Lazy Initialization, as they can lead to instantiation issues and runtime errors.

1. **Understand Proxy Limitations:**
   Recognize that proxies used for Lazy Initialization might not support all bean features, such as final methods or classes when using CGLIB proxies.

# Potential Drawbacks and Considerations

While Lazy Initialization offers numerous advantages, it's essential to be aware of potential drawbacks and considerations to ensure its effective implementation.

1. **Delayed Error Detection:**
   Errors related to bean creation may surface during runtime rather than at startup, complicating debugging and error tracking.

1. **Concurrency Issues:**
   In multi-threaded environments, multiple threads might attempt to instantiate the same lazy bean simultaneously, leading to synchronization challenges.

1. **Increased Complexity:**
   Managing the lifecycle of lazily initialized beans, especially in large applications with complex dependencies, can add to the application's complexity.

1. **Performance Overhead on First Access:**
   The initial access to a lazily initialized bean incurs the cost of instantiation, which might lead to latency in critical application paths.

1. **Potential for Resource Leaks:**
   If lazily initialized beans hold resources, improper management or cleanup can lead to resource leaks, impacting application stability.

1. **Proxy Limitations:**
   Proxies used for Lazy Initialization may not fully support all bean functionalities, such as final methods or classes, potentially limiting their use cases.

1. **Dependency Management:**
   Ensuring that all dependencies of a lazily initialized bean are available and correctly managed can be more challenging compared to eagerly initialized beans.

**Mitigation Strategies:**

- **Comprehensive Testing:** Implement thorough tests to detect and resolve issues related to Lazy Initialization early in the development cycle.
- **Thread-Safe Bean Creation:** Utilize Spring's built-in mechanisms and best practices to manage thread-safe bean creation, minimizing concurrency issues.
- **Monitoring and Profiling:** Regularly monitor the application to identify performance bottlenecks or resource leaks associated with Lazy Initialization.
- **Clear Documentation:** Maintain detailed documentation of bean scopes and initialization strategies to facilitate maintenance and onboarding.
- **Fallback Mechanisms:** Implement fallback mechanisms or default behaviors to handle scenarios where lazy beans fail to instantiate correctly.

# Practical Examples

To illustrate the practical application of Lazy Initialization in Spring, let's explore several real-world scenarios and code examples.

## Example 1: Lazy Initialization with @Lazy Annotation

**Scenario:**

Implement Lazy Initialization for a singleton bean to defer its instantiation until it is first accessed, optimizing resource usage and improving startup time.

**Implementation:**

1.  Define the Expensive Bean:

    ```java
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    @Lazy
    public class ExpensiveService {
        public ExpensiveService() {
            System.out.println("ExpensiveService instantiated.");
        }

        public void performTask() {
            System.out.println("ExpensiveService is performing a task.");
        }
    }
    ```

1.  Define a Client Bean:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ExpensiveService expensiveService;

        @Autowired
        public ClientBean(ExpensiveService expensiveService) {
            this.expensiveService = expensiveService;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing a task.");
            expensiveService.performTask();
        }
    }
    ```

1.  Main Application:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  Console Output:

    ```csharp
    Application started.
    ClientBean instantiated.
    ExpensiveService instantiated.
    ClientBean is executing a task.
    ExpensiveService is performing a task.
    ```

    **Explanation:**

- The ExpensiveService bean is annotated with @Lazy, ensuring it is not instantiated during the Application Context's startup.
- When the ClientBean is instantiated, it receives a proxy for ExpensiveService instead of the actual instance.
- The actual instantiation of ExpensiveService occurs only when performTask() is invoked, demonstrating Lazy Initialization.

## Example 2: Global Lazy Initialization

**Scenario:**

Configure the entire Spring Application Context to lazily initialize all singleton beans, reducing the application's startup time by avoiding the immediate instantiation of beans that may not be required at startup.

**Implementation:**

1.  Enable Global Lazy Initialization via application.properties:

    ```properties
    spring.main.lazy-initialization=true
    ```

1.  Define Multiple Beans:

    ```java
    import org.springframework.stereotype.Component;

    @Component
    public class ServiceA {
        public ServiceA() {
            System.out.println("ServiceA instantiated.");
        }

        public void performServiceA() {
            System.out.println("ServiceA is performing its service.");
        }
    }

    @Component
    public class ServiceB {
        public ServiceB() {
            System.out.println("ServiceB instantiated.");
        }

        public void performServiceB() {
            System.out.println("ServiceB is performing its service.");
        }
    }
    ```

1.  Define a Client Bean:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ServiceA serviceA;
        private final ServiceB serviceB;

        @Autowired
        public ClientBean(ServiceA serviceA, ServiceB serviceB) {
            this.serviceA = serviceA;
            this.serviceB = serviceB;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing.");
            serviceA.performServiceA();
            serviceB.performServiceB();
        }
    }
    ```

1.  Main Application:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  Console Output:

    ```csharp
    Application started.
    ClientBean instantiated.
    ServiceA instantiated.
    ServiceB instantiated.
    ClientBean is executing.
    ServiceA is performing its service.
    ServiceB is performing its service.
    ```

    **Explanation:**

- By enabling spring.main.lazy-initialization=true in application.properties, all singleton beans (ServiceA, ServiceB, ClientBean) are lazily initialized.
- None of the beans are instantiated during the Application Context's startup.
- The beans are instantiated in the order they are accessed within the ClientBean.execute() method.

## Example 3: Combining @Lazy with Dependency Injection

**Scenario:**

Use the @Lazy annotation at the injection point to defer the instantiation of a dependency within a singleton bean, optimizing resource usage and preventing unnecessary instantiation of dependencies that may not always be required.

**Implementation:**

1.  Define the Expensive Bean:

    ```java
    import org.springframework.stereotype.Component;

    @Component
    public class ExpensiveService {
        public ExpensiveService() {
            System.out.println("ExpensiveService instantiated.");
        }

        public void performExpensiveOperation() {
            System.out.println("ExpensiveService is performing an expensive operation.");
        }
    }
    ```

1.  Define a Client Bean with Lazy Dependency:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Lazy;
    import org.springframework.stereotype.Component;

    @Component
    public class ClientBean {

        private final ExpensiveService expensiveService;

        @Autowired
        public ClientBean(@Lazy ExpensiveService expensiveService) {
            this.expensiveService = expensiveService;
            System.out.println("ClientBean instantiated.");
        }

        public void execute() {
            System.out.println("ClientBean is executing.");
            expensiveService.performExpensiveOperation();
        }
    }
    ```

1.  Main Application:

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import com.example.ClientBean;

    @SpringBootApplication
    public class Application implements CommandLineRunner {

        @Autowired
        private ClientBean clientBean;

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            System.out.println("Application started.");
            clientBean.execute();
        }
    }
    ```

1.  Console Output:

    ```csharp
    Application started.
    ClientBean instantiated.
    ClientBean is executing.
    ExpensiveService instantiated.
    ExpensiveService is performing an expensive operation.
    ```

    **Explanation:**

- The ExpensiveService bean is injected into ClientBean with the @Lazy annotation.
- This configuration ensures that ExpensiveService is not instantiated when ClientBean is created.
- Instead, ExpensiveService is instantiated only when clientBean.execute() invokes performExpensiveOperation(), demonstrating deferred instantiation.

# Conclusion

Lazy Initialization is a powerful feature within the Spring Framework that optimizes resource utilization, improves application startup times, and enhances overall performance by deferring the instantiation of beans until they are explicitly required. By strategically applying Lazy Initialization, developers can design more efficient, scalable, and maintainable applications, particularly in scenarios involving numerous or resource-intensive beans.
