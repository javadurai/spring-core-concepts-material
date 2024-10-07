Bean Scopes are a fundamental concept in the Spring Framework that determine the lifecycle and visibility of Spring beans within an application context. Understanding bean scopes is essential for designing applications that are efficient, maintainable, and aligned with specific architectural requirements. This comprehensive guide delves into the intricacies of Bean Scopes in Spring, exploring their definitions, types, implementations, best practices, potential drawbacks, and practical examples to provide a thorough understanding of how to leverage them effectively in your Spring-based applications.
Table of Contents

    Introduction to Bean Scopes
    Understanding Bean Scopes
        What are Bean Scopes?
        Importance of Bean Scopes
    Built-in Bean Scopes in Spring
        Singleton Scope
        Prototype Scope
        Request Scope
        Session Scope
        Global Session Scope
        Websocket Scope
    Custom Bean Scopes
        Defining a Custom Scope
        Registering a Custom Scope
        Using Custom Scopes
    Defining Bean Scopes
        Using Annotations
        Using XML Configuration
        Using Java-Based Configuration
    Bean Scopes in Spring Boot
        Default Behavior
        Configuring Bean Scopes
    Best Practices for Bean Scopes
    Potential Drawbacks and Considerations
    Practical Examples
        Example 1: Singleton and Prototype Scopes
        Example 2: Request and Session Scopes in a Web Application
        Example 3: Creating and Using a Custom Bean Scope
    Conclusion
    Further Reading and Resources

Introduction to Bean Scopes

In the Spring Framework, a bean is an object that is instantiated, assembled, and otherwise managed by the Spring IoC (Inversion of Control) container. The scope of a bean defines its lifecycle, specifying how and when a bean is created, shared, or destroyed within the application context. Selecting the appropriate bean scope is crucial for optimizing resource usage, ensuring thread safety, and aligning with application architecture.

Key Points:

    Lifecycle Management: Bean scopes determine how long a bean exists and when it is garbage collected.
    Resource Optimization: Proper scope selection can lead to efficient memory and resource utilization.
    Thread Safety: Certain scopes are more suitable for multi-threaded environments.
    Modularity and Reusability: Scoped beans can enhance the modularity and reusability of components.

Understanding Bean Scopes
What are Bean Scopes?

Bean Scopes in Spring define the lifecycle and visibility of beans within the Spring IoC container. They determine how many instances of a bean are created, how they are shared, and when they are destroyed. Spring provides several built-in scopes, each tailored to different use cases and application architectures.
Importance of Bean Scopes

Choosing the right bean scope is vital for:

    Performance Optimization: Minimizing unnecessary object creation can enhance application performance.
    Resource Management: Efficiently managing resources like database connections, file handles, and memory.
    Concurrency Control: Ensuring thread safety in multi-threaded applications.
    Flexibility and Scalability: Adapting to varying load conditions and application growth.

Built-in Bean Scopes in Spring

Spring offers multiple built-in bean scopes to cater to different application needs. Understanding each scope's characteristics is essential for appropriate usage.
Singleton Scope

    Definition: A single instance of the bean is created and shared across the entire Spring container.
    Default Scope: If no scope is specified, beans are singleton by default.
    Lifecycle: Bean is created at container startup and destroyed when the container is closed.
    Use Cases: Stateless beans, service layers, data access objects.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

@Component
@Scope("singleton") // Optional, as singleton is default
public class SingletonBean {
    // Bean implementation
}

Prototype Scope

    Definition: A new instance of the bean is created every time it is requested from the container.
    Lifecycle: Bean creation is managed by the container, but destruction is the responsibility of the client.
    Use Cases: Stateful beans, objects that maintain conversational state, tasks that require fresh instances.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

@Component
@Scope("prototype")
public class PrototypeBean {
    // Bean implementation
}

Request Scope

    Definition: A new bean instance is created for each HTTP request.
    Lifecycle: Bean exists during the lifecycle of a single HTTP request.
    Use Cases: Web applications where beans need to maintain request-specific state.

Note: Requires Spring's web context.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_REQUEST)
public class RequestScopedBean {
    // Bean implementation
}

Session Scope

    Definition: A new bean instance is created for each HTTP session.
    Lifecycle: Bean exists for the duration of an HTTP session.
    Use Cases: Web applications requiring session-specific state management.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_SESSION)
public class SessionScopedBean {
    // Bean implementation
}

Global Session Scope

    Definition: Applicable in a Portlet context, a single bean instance is shared across all portlet sessions.
    Lifecycle: Bean exists for the duration of the global portlet session.
    Use Cases: Portlet-based web applications.

Note: Less commonly used; applicable in specific web contexts.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.portlet.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_GLOBAL_SESSION)
public class GlobalSessionScopedBean {
    // Bean implementation
}

Websocket Scope

    Definition: A new bean instance is created for each WebSocket.
    Lifecycle: Bean exists for the duration of a WebSocket connection.
    Use Cases: Real-time web applications using WebSockets.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.annotation.SessionScope;

@Component
@Scope("websocket")
public class WebSocketScopedBean {
    // Bean implementation
}

Custom Bean Scopes

Spring allows developers to define custom bean scopes to address specific application requirements that are not covered by the built-in scopes.
Defining a Custom Scope

To define a custom scope, you need to implement the Scope interface provided by Spring. This involves defining how beans are stored, retrieved, and cleaned up within the custom scope.

Example: Defining a Thread Scope

A common custom scope is the Thread Scope, where each thread has its own bean instance.

    Implement the Scope Interface:

    java

import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;

public class ThreadScope implements Scope {
    private static final ThreadLocal<Map<String, Object>> threadScope = ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadScope.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Optional: Implement if needed
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}

Register the Custom Scope:

java

import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CustomScopeConfig {

    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("thread", new ThreadScope());
        return configurer;
    }
}

Using the Custom Scope:

java

    import org.springframework.stereotype.Component;
    import org.springframework.context.annotation.Scope;

    @Component
    @Scope("thread")
    public class ThreadScopedBean {
        // Bean implementation
    }

Explanation:

    The ThreadScope class defines how beans are managed within the thread scope.
    The CustomScopeConfigurer registers the new thread scope with the Spring container.
    Beans annotated with @Scope("thread") will now have a separate instance per thread.

Registering a Custom Scope

Custom scopes must be registered with the Spring container before they can be used. This is typically done using CustomScopeConfigurer in a configuration class, as shown in the previous example.
Using Custom Scopes

Once a custom scope is defined and registered, you can use it by specifying the scope name in the @Scope annotation or XML configuration.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

@Component
@Scope("thread")
public class ThreadScopedService {
    // Bean implementation
}

Defining Bean Scopes

Bean scopes can be defined using various configuration methods in Spring, including annotations, XML configuration, and Java-based configuration. Modern Spring applications predominantly use annotations and Java-based configurations for their simplicity and type safety.
Using Annotations

Annotations provide a concise and declarative way to define bean scopes directly within the bean classes.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

@Component
@Scope("prototype")
public class PrototypeBean {
    // Bean implementation
}

Common Scope Annotations:

    @Scope("singleton"): Defines a singleton scoped bean.
    @Scope("prototype"): Defines a prototype scoped bean.
    @Scope("request"): Defines a request scoped bean.
    @Scope("session"): Defines a session scoped bean.
    @Scope("websocket"): Defines a websocket scoped bean.

Example with Web Application Scopes:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_REQUEST)
public class RequestScopedBean {
    // Bean implementation
}

Using XML Configuration

Although less common in modern applications, XML configuration allows defining bean scopes in XML files.

Example:

xml

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="singletonBean" class="com.example.SingletonBean" scope="singleton"/>
    
    <bean id="prototypeBean" class="com.example.PrototypeBean" scope="prototype"/>
    
    <bean id="requestScopedBean" class="com.example.RequestScopedBean" scope="request"/>
    
</beans>

Using Java-Based Configuration

Java-based configuration, introduced with Spring 3.0, provides type-safe and refactoring-friendly ways to define bean scopes using @Bean methods within @Configuration classes.

Example:

java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Configuration
public class AppConfig {

    @Bean
    @Scope("singleton") // Optional, as singleton is default
    public SingletonBean singletonBean() {
        return new SingletonBean();
    }

    @Bean
    @Scope("prototype")
    public PrototypeBean prototypeBean() {
        return new PrototypeBean();
    }

    @Bean
    @Scope(WebApplicationContext.SCOPE_REQUEST)
    public RequestScopedBean requestScopedBean() {
        return new RequestScopedBean();
    }
}

Advantages of Java-Based Configuration:

    Type Safety: Compile-time checking of bean configurations.
    Refactoring Support: Easier to refactor bean names and configurations.
    Reduced Boilerplate: Less verbose compared to XML configurations.

Bean Scopes in Spring Boot

Spring Boot builds upon the Spring Framework's capabilities, offering conventions and defaults that streamline bean scope management.
Default Behavior

    Singleton Scope: Beans are singleton by default in Spring Boot, aligning with Spring's default behavior.
    Component Scanning: Beans annotated with @Component, @Service, @Repository, etc., are singleton unless specified otherwise.

Configuring Bean Scopes

Spring Boot supports all standard Spring bean scopes. Configuration can be done using annotations or Java-based configuration as demonstrated earlier.

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_SESSION)
public class SessionScopedBean {
    // Bean implementation
}

External Configuration:

Profiles can be used in Spring Boot to manage bean scopes dynamically across different environments (development, testing, production).

Example:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.Profile;
import org.springframework.web.context.WebApplicationContext;

@Component
@Profile("web")
@Scope(WebApplicationContext.SCOPE_REQUEST)
public class WebRequestScopedBean {
    // Bean implementation
}

Best Practices for Bean Scopes

Adhering to best practices ensures that bean scopes are used effectively, enhancing application performance, maintainability, and scalability.

    Understand Default Scope:
        Remember that beans are singleton by default. Explicitly specify other scopes only when necessary.

    Use Prototype Scope Judiciously:
        Prototype beans can lead to increased memory usage if overused. Consider whether the bean truly requires multiple instances.

    Prefer Scoped Proxies for Non-Singleton Beans:
        When injecting request or session-scoped beans into singleton beans, use scoped proxies to manage the lifecycle correctly.

    Example:

    java

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Scope;
    import org.springframework.context.annotation.ScopedProxyMode;
    import org.springframework.web.context.WebApplicationContext;

    @Configuration
    public class ProxyConfig {

        @Bean
        @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
        public RequestScopedBean requestScopedBean() {
            return new RequestScopedBean();
        }
    }

    Leverage Profiles for Environment-Specific Scopes:
        Use Spring Profiles to manage bean scopes and configurations across different environments seamlessly.

    Avoid Overcomplicating Scopes:
        Stick to standard scopes unless a custom scope is absolutely necessary. Custom scopes can introduce complexity.

    Ensure Thread Safety:
        For singleton beans that hold state, ensure thread safety to prevent concurrency issues.

    Document Bean Scopes:
        Clearly document the scope of each bean to aid in maintenance and onboarding of new developers.

    Monitor Resource Usage:
        Use profiling tools to monitor the impact of bean scopes on application memory and performance.

    Use Scoped Proxies When Necessary:
        Scoped proxies help manage the injection of beans with narrower scopes into beans with broader scopes, preventing lifecycle mismatches.

    Test Scoped Beans Thoroughly:
        Ensure that beans with non-singleton scopes behave as expected across different scenarios and usage patterns.

Potential Drawbacks and Considerations

While bean scopes provide flexibility in managing bean lifecycles, they come with certain challenges and considerations that developers must address to maintain application quality.

    Increased Complexity with Multiple Scopes:
        Managing various scopes can complicate the application architecture, making it harder to understand and maintain.

    Memory Overhead:
        Non-singleton scopes like prototype, request, and session can lead to increased memory consumption if not managed properly.

    Thread Safety Concerns:
        Singleton beans that are not thread-safe can cause issues in multi-threaded environments. Proper synchronization or stateless designs are necessary.

    Scoped Bean Injection Issues:
        Injecting beans with narrower scopes into broader-scoped beans (e.g., request-scoped beans into singleton beans) can lead to lifecycle mismatches and unexpected behaviors if not handled with scoped proxies.

    Lifecycle Management Challenges:
        Non-singleton beans may require explicit destruction callbacks or proper resource management to prevent leaks.

    Potential for Configuration Errors:
        Incorrectly specifying bean scopes can lead to runtime errors, such as missing beans or unintended singleton behavior.

    Testing Complexity:
        Scoped beans, especially in web contexts, can complicate unit and integration testing due to their dependencies on specific environments or contexts.

Mitigation Strategies:

    Adhere to Best Practices: Follow the best practices outlined above to minimize complexity and prevent common pitfalls.
    Use Scoped Proxies: Utilize Spring's scoped proxies to manage lifecycle mismatches effectively.
    Monitor and Profile: Regularly profile the application to detect memory leaks or performance issues related to bean scopes.
    Ensure Thread Safety: Design singleton beans to be stateless or implement proper synchronization mechanisms.
    Comprehensive Testing: Implement thorough testing strategies that account for different bean scopes and their interactions.

Practical Examples

To illustrate the application of bean scopes in Spring, let's explore several practical examples covering different scenarios and scopes.
Example 1: Singleton and Prototype Scopes

Scenario:

Demonstrate the difference between singleton and prototype bean scopes by injecting them into a client bean and observing their behavior.

Implementation:

    Define Singleton and Prototype Beans:

    java

import org.springframework.stereotype.Component;

@Component
public class SingletonBean {
    public SingletonBean() {
        System.out.println("SingletonBean instance created.");
    }
}

@Component
public class PrototypeBean {
    public PrototypeBean() {
        System.out.println("PrototypeBean instance created.");
    }
}

Define a Client Bean that Injects Both Beans:

java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ClientBean {

    @Autowired
    private SingletonBean singletonBean;

    @Autowired
    private PrototypeBean prototypeBean;

    public void displayBeans() {
        System.out.println("SingletonBean: " + singletonBean);
        System.out.println("PrototypeBean: " + prototypeBean);
    }
}

Main Application:

java

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
        clientBean.displayBeans();
        clientBean.displayBeans();
    }
}

Console Output:

graphql

    SingletonBean instance created.
    PrototypeBean instance created.
    PrototypeBean instance created.
    SingletonBean: com.example.SingletonBean@1a2b3c4
    PrototypeBean: com.example.PrototypeBean@5d6e7f8
    SingletonBean: com.example.SingletonBean@1a2b3c4
    PrototypeBean: com.example.PrototypeBean@9a0b1c2

Explanation:

    SingletonBean: Only one instance is created when the application starts. Both calls to displayBeans() refer to the same SingletonBean instance.
    PrototypeBean: A new instance is created each time it is injected. However, in this example, since the PrototypeBean is injected into the ClientBean at startup, both displayBeans() calls refer to the same PrototypeBean instance. To truly observe prototype behavior, you need to request the prototype bean multiple times.

Enhanced Example with Prototype Behavior:

To observe the prototype scope properly, modify the ClientBean to request the prototype bean each time.

    Modify ClientBean to Use ObjectFactory:

    java

import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ClientBean {

    @Autowired
    private SingletonBean singletonBean;

    @Autowired
    private ObjectFactory<PrototypeBean> prototypeBeanFactory;

    public void displayBeans() {
        PrototypeBean prototypeBean = prototypeBeanFactory.getObject();
        System.out.println("SingletonBean: " + singletonBean);
        System.out.println("PrototypeBean: " + prototypeBean);
    }
}

Run the Application:

Console Output:

graphql

    SingletonBean instance created.
    PrototypeBean instance created.
    PrototypeBean instance created.
    SingletonBean: com.example.SingletonBean@1a2b3c4
    PrototypeBean: com.example.PrototypeBean@5d6e7f8
    PrototypeBean instance created.
    SingletonBean: com.example.SingletonBean@1a2b3c4
    PrototypeBean: com.example.PrototypeBean@9a0b1c2

Explanation:

    Each call to displayBeans() now retrieves a new PrototypeBean instance via ObjectFactory, showcasing the prototype behavior accurately.

Example 2: Request and Session Scopes in a Web Application

Scenario:

Implement request-scoped and session-scoped beans in a Spring Boot web application to manage user-specific data.

Implementation:

    Define RequestScopedBean and SessionScopedBean:

    java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
@Scope(WebApplicationContext.SCOPE_REQUEST)
public class RequestScopedBean {
    private String requestData;

    public String getRequestData() {
        return requestData;
    }

    public void setRequestData(String requestData) {
        this.requestData = requestData;
    }
}

@Component
@Scope(WebApplicationContext.SCOPE_SESSION)
public class SessionScopedBean {
    private String sessionData;

    public String getSessionData() {
        return sessionData;
    }

    public void setSessionData(String sessionData) {
        this.sessionData = sessionData;
    }
}

Create a Controller that Uses These Beans:

java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DataController {

    @Autowired
    private RequestScopedBean requestScopedBean;

    @Autowired
    private SessionScopedBean sessionScopedBean;

    @GetMapping("/setRequestData")
    public String setRequestData(@RequestParam String data) {
        requestScopedBean.setRequestData(data);
        return "Request data set to: " + data;
    }

    @GetMapping("/getRequestData")
    public String getRequestData() {
        return "Request data: " + requestScopedBean.getRequestData();
    }

    @GetMapping("/setSessionData")
    public String setSessionData(@RequestParam String data) {
        sessionScopedBean.setSessionData(data);
        return "Session data set to: " + data;
    }

    @GetMapping("/getSessionData")
    public String getSessionData() {
        return "Session data: " + sessionScopedBean.getSessionData();
    }
}

Run the Application and Test:

    Set Request Data:

    bash

GET http://localhost:8080/setRequestData?data=HelloRequest
Response: "Request data set to: HelloRequest"

Get Request Data:

bash

GET http://localhost:8080/getRequestData
Response: "Request data: HelloRequest"

Set Session Data:

bash

GET http://localhost:8080/setSessionData?data=HelloSession
Response: "Session data set to: HelloSession"

Get Session Data:

bash

        GET http://localhost:8080/getSessionData
        Response: "Session data: HelloSession"

Explanation:

    RequestScopedBean: Data is specific to a single HTTP request. Each request to /setRequestData and /getRequestData operates within its own request scope.
    SessionScopedBean: Data persists across multiple HTTP requests within the same session. Setting and retrieving session data maintains state across different endpoints.

Example 3: Creating and Using a Custom Bean Scope

Scenario:

Implement a custom Thread Scope where each thread has its own instance of a bean, useful in multi-threaded applications where beans maintain thread-specific state.

Implementation:

    Define the Custom Scope:

    java

import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;

import java.util.HashMap;
import java.util.Map;

public class ThreadScope implements Scope {

    private final ThreadLocal<Map<String, Object>> threadScope = ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadScope.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Optional: Implement if destruction is needed
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}

Register the Custom Scope:

java

import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CustomScopeConfig {

    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("thread", new ThreadScope());
        return configurer;
    }
}

Define a Bean with the Custom Scope:

java

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

@Component
@Scope("thread")
public class ThreadScopedBean {
    private int counter = 0;

    public void increment() {
        counter++;
        System.out.println("Thread: " + Thread.currentThread().getName() + ", Counter: " + counter);
    }
}

Using the ThreadScopedBean in a Multi-Threaded Context:

java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application implements CommandLineRunner {

    @Autowired
    private ThreadScopedBean threadScopedBean;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        Runnable task = () -> {
            threadScopedBean.increment();
            threadScopedBean.increment();
        };

        Thread thread1 = new Thread(task, "Thread-1");
        Thread thread2 = new Thread(task, "Thread-2");

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
    }
}

Console Output:

mathematica

    Thread: Thread-1, Counter: 1
    Thread: Thread-1, Counter: 2
    Thread: Thread-2, Counter: 1
    Thread: Thread-2, Counter: 2

Explanation:

    ThreadScopedBean: Each thread has its own instance of ThreadScopedBean, maintaining separate counter values.
    Application: Two separate threads (Thread-1 and Thread-2) increment their respective ThreadScopedBean instances without interfering with each other.

Conclusion

Bean Scopes in Spring are integral to managing the lifecycle and visibility of beans within an application context. By understanding and appropriately leveraging the various bean scopes, developers can design applications that are efficient, scalable, and aligned with specific architectural and operational requirements.

Key Takeaways:

    Diverse Scopes for Diverse Needs: Spring provides multiple built-in scopes (singleton, prototype, request, session, etc.) to cater to different application scenarios.
    Custom Scopes for Flexibility: When built-in scopes are insufficient, Spring allows the creation of custom scopes to address unique application requirements.
    Configuration Flexibility: Bean scopes can be defined using annotations, XML configurations, or Java-based configurations, offering flexibility in how applications are structured.
    Best Practices Enhance Effectiveness: Adhering to best practices ensures optimal resource utilization, maintainability, and performance.
    Awareness of Potential Drawbacks: Understanding the challenges associated with different bean scopes helps in mitigating risks and avoiding common pitfalls.
    Practical Implementation: Real-world examples illustrate how bean scopes operate in various contexts, reinforcing theoretical knowledge with practical application.

By mastering Bean Scopes, developers can harness the full potential of the Spring Framework, creating robust, efficient, and maintainable applications that effectively manage resources and align with diverse operational contexts.