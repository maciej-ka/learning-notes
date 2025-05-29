Java Fundamentals
=================
https://frontendmasters.com/courses/java

#### package name
```java
package basics;
```

lowercase underline  
used for logical separation of files

#### class name
```java
public class GrossPayCalculator {
}
```

class name has to be same as filename  
class names are case sensitive



Enterprise Java: Spring Boot Fundamentals
=========================================
https://frontendmasters.com/workshops/spring-boot/  
Josh Long

Java Spring was inspired by article about problems with Java.  
It's rare case of documentation preceeding code.

Java failed back a bit because it was not opinionated as Rails.  
However Spring has a lot of autoconfiguration, which is optional.

#### who uses Spring
It's easier to talk about who doesn't use Spring  
Google, Twitter, Facebook

#### IDE
intelliJ IDEA Ultimate  
JetBrains  
Spring Boot

#### way to start Spring
https://start.spring.io/  
remembers settings  
Maven, JAva, 3.4.4, Jar, 24 Java  
Spring Boot DevTools  
Spring Web  
Spring Boot Actuator  
Spring Data JDBC  
PostgreSQL Driver  
Docker Compose Support  
GraalVM Native Support

there is a way to share setup you like  
curl -G https://start.spring.io/starter.tgz -d dependencies=web,data-jpa -d type=gradle-project -d baseDir=my-dir | tar -xzvf

#### langs
Kotlin is very popular in Java  
Groovy a nice language

#### sdkman.io
great tool to manage Java versions

```bash
curl -s "https://get.sdkman.io" | bash
sdk list java
```

A lot of different vendors of Java  
different implementations.  
Some are value added, some are reimplementation.

we will use  
24-graalce

```bash
sdk install java 24-graalce
sdk default java 24-graalce
```

graalvm.org

```bash
sdk env init
```

this will create .sdkmanrc file  
that will pin version of java used

#### most popular JDK distrubution
Corretto Amazon.  
Most popular because it's default in a lot of places.

#### Liberrica
Another nice one, has a lot of extra support.

#### Spring code, Juergenization
Code of Spring is legendary,  
it's very backward comp,  
more than linux 

Juergen Hoeller: legendary architect  
https://www.youtube.com/watch?v=qNDMvyUfkpA  
one of first two people joined Spring  
made a great work of layering abstractions  
that they can handle future changes.

Juergen is also famous for rewriting PR completely  
"Juergenization", taking PR, redo it, and merge.

#### run project

```bash
curl -s "https://get.sdkman.io" | bash
sdk install java 24-graalce
sdk default java 24-graalce
git clone https://github.com/joshlong-attic/2025-frontend-masters-spring-boot-course
cd 2025-frontend-masters-spring-boot-course/intro/demo
docker compose up
./mvnw spring-boot:run
```

then visit  
http://localhost:8080/hello

create jar
```bash
./mvnw package
java -Dserver.port=8023 -Dspring.datasource.username=myuser -Dspring.datasource.password=secret -jar target/demo-0.0.1-SNAPSHOT.jar.original
```

perhaps this should work also...  
./mvnw spring-boot:build-image

#### Build native

```bash
./mvnw -DskipTests -Pnative native:compile
```

#### direnv
```bash
touch .envrc
```

.envrc
```
echo "hello frontendmasters"
export FOO=bar
```

then
```bash
direnv allow
echo $FOO
```

#### secrets, password manager
but for important secrets  
use password secrets  
they have CLI

they can accept name of key, and pull the secret  
example in bit warden

```
export FOO=$( bw get item key-name )
```

it will ask for credentails to access Bit Warden.

### Java
has 30 years  
Java is 3rd most popular, after Python/Java.  
Most critic is about syntax.

It's partially true, because it's not very short,  
but things improved in Java 14 and also Java 21,  
which supports data oriented programming.

#### paradigms
Object Oriented: obviously always was, it was almost selling point,  
Functional: has elements of functional idioms  
Data oriented: new paradigm

Today we build systems around messages.

#### new features of language
sealed types  
pattern matching  
smart switch

#### records
In Java everything has to have a type.

```java
sealed interface Load permits SecuredLoan, UnsecuredLoan {}
final class SecuredLoad implements Load {}
record UnsecuredLoad (float interest) implements Load {}
```

previous way

```java
class Loans {
  String displayMessageFor(Load loan) {
    var msg = ""

    if (loan instanceof SecuredLoad) {
      msg = "good job, nice load
    }
  }
}
```

smart switch way

```java
class Loans {
  String dsplayMessageFor(Load loan) {
    var msg = switch (loan) {
      case SecuredLoad _ -> "good job. nice loan";
      case UnsecuredLoadn(var interest) -> "you are in trouble " + interest + "% is a lot!";
    }
  }
}
```

### Spring
Supports DI, dependency injection.  
It's popular now, it was a big thing then.

J2EE, from late 90'  
One of antipattern was RAII: resource aquisition is initialization


```java
final DataSource db = new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2).build();

Collection<Customer> customers() throws Exception {
  var customers = new ArrayList<Customer>();
  var sql = "SELECT id, name FROM customer";
  try (var conn = this.db.getConnection(); //
      var statement = conn.prepareStatement(sql); //
      var rs = statement.executeQuery()) {
    while (rs.next()) {
      var id = rs.getInt("id");
      var name = rs.getString("name");
      customers.add(new Customer(id, name));
    }
  }
  return customers;
}
```

we do a lot of boiler plate above,  
it's the least pleasent code, direct JDBC  
one step above is to use DI for creating

at least data source is comming from outside  
it's not created here

```java
class CustomerService {

  private final DataSource db;

  CustomerService(DataSource db) {
    this.db = db;
  }

  Collection<Customer> customers() throws Exception {
    var customers = new ArrayList<Customer>();
    var sql = "SELECT id, name FROM customer";
    try (var conn = this.db.getConnection(); //
        var statement = conn.prepareStatement(sql); //
        var rs = statement.executeQuery()) {
      while (rs.next()) {
        var id = rs.getInt("id");
        var name = rs.getString("name");
        customers.add(new Customer(id, name));
      }
    }
    return customers;
  }

}
```

spring triangle  
portable service abstractions  
utility abstractions that make working with low level apis more pleasent


```java
class CustomerService {

  private final JdbcClient db;

  CustomerService(JdbcClient db) {
    this.db = db;
  }

  Collection<Customer> customers() throws Exception {
    return db.sql("select id, name from CUSTOMER")
      .query((rs, _) -> new Customer(rs.getInt("id"), rs.getString("name")))
      .list();

  }

}
```

changing behaviour in low class   
using subclass to extend behaviour

```java
var customerService = LoggableProxyMaker.proxy(new CustomerService(jdbcClient));
```

and that subclass is created dynamically  
in a runtime


```java
@SuppressWarnings("unchecked")
public static <T> T proxy(T target) {
  var pfb = new ProxyFactoryBean();
  pfb.setTarget(target);
  pfb.setProxyTargetClass(true);
  pfb.addAdvice((MethodInterceptor) invocation -> {
    var start = System.currentTimeMillis();
    var result = (Object) null;
    try {
      result = invocation.proceed();
    }
    finally {
      var end = System.currentTimeMillis();
      logger.info("method {} took {} ms", invocation.getMethod().getName(), end - start);
    }
    return result;
  });
  return (T) pfb.getObject();
}
```

AOP, aspect oriented

```java
class CustomerService {

  private final JdbcClient db;

  CustomerService(JdbcClient db) {
    this.db = db;
  }

  Collection<Customer> customers() throws Exception {
    return db.sql("select id, name from CUSTOMER")
      .query((rs, _) -> new Customer(rs.getInt("id"), rs.getString("name")))
      .list();
  }

}
```

DI  
AOP  
portable service abstractions

Beans  
ApplicationContext

Component scan  
A way for Spring to autodiscover Beans  
without need to configure them and define in separate file

```javascript
@Service
class CustomerService {

  private final JdbcClient db;

  CustomerService(JdbcClient db) {
    this.db = db;
  }

  Collection<Customer> customers() throws Exception {
    return db.sql("select id, name from CUSTOMER")
      .query((rs, _) -> new Customer(rs.getInt("id"), rs.getString("name")))
      .list();
  }

}
```

Life cycle methods

```java
@Service
class CustomerService implements InitializingBean {

  private final JdbcClient db;

  CustomerService(JdbcClient db) {
    this.db = db;
  }

  Collection<Customer> customers() throws Exception {
    return db.sql("select id, name from CUSTOMER")
      .query((rs, _) -> new Customer(rs.getInt("id"), rs.getString("name")))
      .list();
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    Assert.notNull(this.db, "the db should not be null");
  }

  @PreDestroy
  void destroy() {
    System.out.println("shutting down " + getClass().getSimpleName() + '.');
  }

}
```

#### Bean processors

Spring can visit all the object.  
One way to to visit is using BeanPostProcessor  
This is run before running app.

BeanFactoryPostProcessor  
A way to run custom code when Spring creates metamodel,  
when it creates a list of managed objects.  
This happens even earlier then BeanPostprocessor

Example: give me all the Beans that have some interface.

By default every object has a singleton scope.

### Above Spring 1.0
Spring boot: convention over configuration

#### Bean can be optional
And it's possible to handle situations when Bean is not in class path.  
It's possible to use default managed behaviour of Spring,  
but Spring Boot will take step back if we want to create ourself.

```java
@Autowired
```

#### Environment
It's possible to add configuration from password storage.  
So that `Environment` can be populated from password storage.

#### Events
A lot of events to work with:  
Event when someone authenticates,  
Event when application is ready,  
When Context is ready,  
When some custom event is published

```java
@EventListener
```

#### Spring boot will not be good for
Kernel extensions

Frontend application (Vue, React)  
however GraalVM: image compiler that can target web assembly  
so perhaps this will change in future

building AI models, not great  
but great for consuming AI models

mobile apps  
because Swift  
also Android is not good fit for Spring Boot

#### GraalVM AOT
AOT Ahead of time compilation.  
Amazing thing.

GraalVM Oracle
```bash
sdk install java 25.ea.18-greaal
```

Java has a release every 6 months.

```bash
javac Main.java
java Main
native-image Main
./main
```

No JRE,  
Running it assembly,  
as fast as possible

Will be complete, doesn't require Java to run  
it's self contained, this can be a good fit  
for many applications, like cli tools.

```bash
native-image --tool:svm-wasm Main
node main.j
```

however atm there is no network support

#### limitation of AOT, GraalVM:
it's not possible to generate executable  
for all three systems  
just working from macOS  
(they way it's possible with Golang)

#### check how much ram process takes
get PID  
ps -o rss <PID>

#### GraalVM from Oracle
not open, proprietary  
some extra tools,  
one of them is profile optimalizations  
and JIT which works very well, but only for hot path.

however, you can still use it to build jar  
and ship it how you want

but you cannot make another GraalVM distribution  
and claim, that it's yours.

#### GraalVM CE
open source


#### GraalVM limitation
Reflaction requries metadata,  
which is rejected with GraalVM,  
so these will not work normally:

```java
class Foo {
  void bar() {}
}

class ReflectionThingy {
  ReflectionThingy() {
    var fooClass = Foo.class;
    for (var field: fooClass.getDeclaredMethods())
      System.out.println(field.getName());
  }
}
```

However, we can give compilation hints.  
They will be run at a compilation time  
and instruct compiler to keep metadata.

```java
static class Hints implements RuntimeHintsRegistrar {
  @Override
  public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
    hints.reflection().registerType(Foo.class, MemberCategory.values())
  }
}
```

#### GraalVM limitations
jdk proxies  
resource loading  
reflection  
serialization  
jni

However there are solutions for all these.  
And Java is going ahead of that direction.

#### Project Leyden
Inspired by GraalVM  
result is not that impressive.  
But also goal is loar than GraalVM.

Attempt to move some things  
which traditionally are at runtime to compile time  
Class Loading and linking, code compilation, method profiling

It works well with reflection without hints

#### Spring Beans
All the annotations are set by reflection.  
Spring builds 

#### Reflection in Java
You cannot avoid Reflection in Java,  
It's everywhere, all annotations use it.

### Scalability
Some time ago, developer machine was way slower than production.  
But today opposite is true: dev laptop is amazing compared to prod.  
Because prod is often run on some stock machine just to run some linux.  
And today idea is to scale horizontally and use load balancers.

#### Thread, VirtualThread
When waiting for socket response, using Thread is bad idea  
because Thread will lock whole processor just waiting for response.

Way better is to use VirtualThread for that case.

#### Executor
You run code in Executor, and that Executor can be either: Thread or VirtualThread.  
In Executor you decide is IO blocking or non blocking.  
So not the code decides

In compared to that, in Kotlin and Javascript you can achieve this  
but you have to put async/await or suspended everywhere, which is polluting the code.

#### Flyway migrations
Tool to manage migrations

file for migration:  
V1__setup.sql

file that defines rollback of that migration:  
U1__setup.sql  
(or R1 perhaps)

#### Embedded database
H2, Derby

#### Meta annotations
`@Repository` is a meta annotation,  
which consists of serveral other annotations.

#### ListCrudRepository
A way to provide typical model.  
Has a lot of similarities with Ruby on Rails.

```java
interface PersonRepository extends ListCrudRepository<Person, Integer> {
  @Query("select ....")
  // easy way to find all, etc.
}
```

#### YugabyteDB
PostgreSQL frontend with very scallable backend.  
It will scale like Cassandra will.  
distributed PostgreSQL

DynamoDB, RDBs, ... 

#### Spring Batch
framework to solve ETL batch processing  
framework built on top of Spring Boot  
Extraction Transformation Loading

Spring Batch is meant to handle large amounts of data  
and to turn for long time. For that reason track of task  
is being traced in database, in tables: batch_step_execution,  
batch_job_execution, batch_job_execution_context ...

They way to work with Spring Batch is to define jobs  
and then run them several times.

If job has same name as previous, it will not be run again,  
this can be used to a benefit, when wanting to run job  
but never more than once a day, then use date as part of job name.

When you have milions of rows, don't read all in `select *`,  
instead use chunks. Chunk size is also important for two reasons:
- it should be able to nicely fit into memory,
- it's a size you're ready to loose in case of error.

a lot of financial services use it

Partition steps: a way to divide work in pararell.  
It can use Kafka, in a setup where there is leader node  
and that node will send some work to do for another node.

#### MCP: model context protocol
MCP server for Spring Batch.  
You can ask, how far is my batch work,  
ask it in human language.

### Simple HTTP client
These are dependencies for whole web part of course:

https://start.spring.io/  
Maven, Java, Boot 3.4.4, Jar packaging, Java 24

GraalVM Native Support Developer Tools  
Spring Web Web  
Spring for GraphQL Web  
Spring gRPC [Experimental] I/O  
Thymeleaf Template Engines  
Spring HATEOAS Web  
Spring Boot DevTools Developer Tools

Fake API for testing and prototyping  
https://jsonplaceholder.typicode.com/  
https://jsonplaceholder.typicode.com/users

#### Define domain model

```java
record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}
```

#### Client
Create client for the second of these urls.  
It's going to be a Bean, managed by Spring.

There are few ways to mark that code to be managed:

```
@Component: Generic stereotype annotation for any Spring-managed component
@Service: Used for service layer classes
@Repository: Used for persistence layer/DAO classes
@Controller/@RestController: Used for web controllers
@Configuration
```

And it will use http client:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestClient;

@Component
class SimpleUsersClient {
  private final RestClient http;

  SimpleUsersClient(RestClient.Builder http) {
    this.http = http.build();
  }
}
```

#### Reusable configuration
Or even better, with reusable configuration  
and because of it will be possible to inject RestClient  
instead of needing to build it every time.

It will define base url of the http client.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestClient;

@Configuration
class WebConfiguration {
  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}
```

#### Limitation of Java collections
Java has no reified generics.  
At runtime there is no way to use reflection  
to ask `List<String> string` what generics it uses

In runtime that list just looks like list of objects.  
It also means there is no way to have a class for list of users.

They hack to solve this:  
bake generic type into superclass hierarchy.  
This gets reified. Because we are extending concrete type,  
then UserList has a type, it's possible to get it using reflection.

```java
class UserList extends ArraList<User> {}
```

#### Parametrized Type Token Pattern
this also works: it will create anonymous private class  
Because there is a curly bracket at the end  
then technically it's a subclass of ArrayList

```java
List<User> users = new ArrayList<>(){}
```

and this also works, its possible to inspect this type  
and deduce that generic parameter is of a type User.

And for convienience, this is available as ParametrizedTypeReference  
Here you also have to use curly brackets at the end to create subclass.

```java
ParametrizedTypeReference<List<User>> typeRef = new ParametrizedTypeReference<List<User>>() {}
```

but also this can work:

```java
ParametrizedTypeReference<List<User>> typeRef = new ParametrizedTypeReference<>() {}
```

#### Making a call to endpoint
Example of using RestClient with parametrized type.

```java
@Component
class SimpleUsersClient {
  private final RestClient http;

  SimpleUsersClient(RestClient http) {
    this.http = http;
  }

  private final ParameterizedTypeReference<List<User>> typeRef = new ParameterizedTypeReference<List<User>>() {};

  Collection<User> users(){
    return this.http
      .get()
      .uri("/users")
      .retrieve()
      .body(typeRef);
  }
}
```

#### Runner
And example of runner that will start

```java
@Configuration
class WebConfiguration {
  @Bean
  ApplicationRunner runner(SimpleUsersClient usersClient) {
    return _ -> usersClient.users().forEach(System.out::println);
  }
}
```

Then start with:

```bash
./mvnw spring-boot:run
```

#### Complete code
(/my-client-simple)

```java
package com.example.web;

import java.util.Collection;
import java.util.List;

import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestClient;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }

  @Bean
  ApplicationRunner runner(SimpleUsersClient usersClient) {
    return _ -> usersClient.users().forEach(System.out::println);
  }
}

@Component
class SimpleUsersClient {
  private final RestClient http;

  SimpleUsersClient(RestClient http) {
    this.http = http;
  }

  private final ParameterizedTypeReference<List<User>> typeRef = new ParameterizedTypeReference<List<User>>() {};

  Collection<User> users(){
    return this.http
      .get()
      .uri("/users")
      .retrieve()
      .body(typeRef);
  }
}
```

### Declarative Interface Clients
Code above is substantial amount of work.  
And there is no need to repeat it.

#### Describe API we consume
In declarative approach, we describe routes of API that we want to consume.  
We don't use `@Get` but `@GetExchange,` which is a client side analog of `@Get`.

```java
import org.springframework.web.service.annotation.GetExchange;

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

#### Boilerplate
However, to make that work, we need to create HttpServiceProxy.  
This part will be easier soon, but that version of Spring is not ready yet.

```java
@Configuration
class WebConfiguration {
  @Bean
  DeclarativeUsersClient declarativeUserClient(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build()
      .createClient(DeclarativeUsersClient.class);
  }
}
```

#### Reusable boilerplate
Or, for simplification, we can extract first part, factory creation  
into `@Bean`, so that it can be reused and injected.

```java
@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }
}
```

#### RestClient
Can use one of several low level libraries for HTTP.  
It will adopt to use library that it can find in class path.

#### Complete code
(/my-client-declarative)

```java
package com.example.web;

import java.util.Collection;

import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.client.RestClient;
import org.springframework.web.client.support.RestClientAdapter;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }

  @Bean
  ApplicationRunner runner(DeclarativeUsersClient usersClient) {
    return _ -> {
      usersClient.users().forEach(System.out::println);
      System.out.println(usersClient.user(1));
    };
  }
}

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

### Qualifiers
When there is more than one candidate for DI,  
more then one good fit for requested dependency,  

In this example Spring will complain, that it cannot AutoWire,  
(inside httpServiceProxyFactory, the first Bean), because there is  
more then one Bean with type RestClient.

```java
@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }
  
  @Bean
  RestClient securedRestClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}
```

#### Bean function name
One option is to select one by the Bean name.

```java
@Bean
HttpServiceProxyFactory httpServiceProxyFactory(Map <String, RestClient> http) {
  return HttpServiceProxyFactory
    .builder()
    .exchangeAdapter(RestClientAdapter.create(http.get("securedRestClient")))
    .build();
}
```

#### Bean annotated name
Or by giving name in annotation, so that actual name is not that important  
and there is no magical reflective string, that is not bound to anything.

```java
@Bean
HttpServiceProxyFactory httpServiceProxyFactory(Map <String, RestClient> http) {
  return HttpServiceProxyFactory
    .builder()
    .exchangeAdapter(RestClientAdapter.create(http.get(SECURED_REST_CLIENT)))
    .build();
}

static final String SECURED_REST_CLIENT = "securedRestClient";
static final String REST_CLIENT = "restClient";

@Bean(name = SECURED_REST_CLIENT)
RestClient securedRestClient(RestClient.Builder builder) {
  return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
}

@Bean(name = REST_CLIENT)
RestClient restClient(RestClient.Builder builder) {
  return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
}
```

#### Qualifier
Or by using qualifier to guide AutoWiring.  
That Qualifier is saying, find the bean that has that name.

```java
@Bean
HttpServiceProxyFactory httpServiceProxyFactory(@Qualifier(SECURED_REST_CLIENT) RestClient http) {
  return HttpServiceProxyFactory
    .builder()
    .exchangeAdapter(RestClientAdapter.create(http))
    .build();
}
```

Also it's very easy to crate custom, composed annotation

### HTTP Web Server

```java
@Controller
@ResponseBody
class UsersController {
	private final DeclarativeUsersClient usersClient;

	UsersController(DeclarativeUsersClient usersClient) {
		this.usersClient = usersClient;
	}

	@GetMapping("/users")
	Collection<User> users() {
		return this.usersClient.users();
	}
}
```

#### Virtual threads
Because we are calling another service,  
we need to set virtual threads, otherwise IO will be CPU blocking.

Also if we would be talking with database.  
Generally recommendation is to enable virtual  
threads by default.

application.properties
```
spring.application.name=web
spring.threads.virtual.enabled=true
```

#### Complete code
(/my-server-http)

```java
package com.example.web;

import java.util.Collection;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.client.RestClient;
import org.springframework.web.client.support.RestClientAdapter;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

@Controller
@ResponseBody
class UsersController {
  private final DeclarativeUsersClient usersClient;

  UsersController(DeclarativeUsersClient usersClient) {
    this.usersClient = usersClient;
  }

  @GetMapping("/users")
  Collection<User> users() {
    return this.usersClient.users();
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

#### Start

```bash
./mvnw spring-boot:run
```

and visit: http://localhost:8080/users

#### Other http methods
Example of Post

```java
@PostMapping("/users/{id}")
void createuser(@RequestParam String name, @PathVariable int id, HttpServletRequest request) {}
```

lowest level mapping annotation is actually a RequestMapping

```java
@RequestMapping (method = RequestMethod.GET, value = "/users")
// which is equivalent to
@GetMapping("/users")
```

#### Composed annotations, Meta annotations
Spring is meant to write composed meta annotations,  
so that code can be idiomatic. In domain modeling we talk  
about ubiquous language.

```java
@Controller
@ResponseBody
// both are part of
@RestController
```

but what we had done so far is not REST it's HTTP...

### HATEOAS REST web server
#### REST Hypermedia idea
REST is not really REST unless it uses hypermedia  
Roy Fielding dissertation, famous talk:

When you look at HTML document, link has rel.  
The rel is the important part, it tells you what is the nature of link  
why would you click on this. We don't use it often.

We use it for "stylesheet" to instruct browser, that it could  
and it should follow this link, to get styles for the page.

Enterprise Integration Patterns, 2004 Hoppe and Wolf.  
Hoppe "Starbucks Does Not Use Two-Phase Commit"  
Starbucks trail of cups can fail and be reverted on any stage,  
there is no one huge transaction.

Idea that with API you get menu which tells,  
how this API can be used to. And also it limits,  
what can you call, depending on current state of application.

Example of this is when you have api for orders that allow to refund,  
you shouldn't be able to even call refund, if you don't have order.  
This way programming becomes easier,  
because you don't have to defend against that case.

The Server controlls the state, it's where the state lives,  
if there is no link in response, then client shuold not show that link.

#### HATEOAS acronym
Hypermedia As The Engine Of Application State.

For this to work, it needs to be apparent from data,  
what state is application in, and what operations are possible.

It works by server tracing its state and modifying enable API  
depending on changes in that state. This way server in some part  
drives the client.

#### Example result
Example result of calling http://localhost:8080/users will be:

```json
{
  "_embedded": {
    "userList": [
      {
        "id": 1,
        "name": "Leanne Graham",
        "username": "Bret",
        "email": "Sincere@april.biz",
        "address": {
          "stree": null,
          "suite": "Apt. 556",
          "city": "Gwenborough",
          "zipcode": "92998-3874",
          "geo": {
            "lat": -37.3159,
            "lng": 81.1496
          }
        },
        "_links": {
          "all": {
            "href": "http://localhost:8080/users"
          },
          "self": {
            "href": "http://localhost:8080/users/1"
          }
        }
      }
    ]
  }
}
```

Each entity has links,  
It's also possible to add links on resource root level.  
And it's possible to use non-hypertext links.

#### Hypermedia endpoints
Basically endpoint handler its the same code as before,  
but wrapped in model envelope.

```java
@GetMapping("/users/{id}")
EntityModel<User> one(@PathVariable int id) {
  return this.userModelAssembler.toModel(usersClient.user(id));
}

@GetMapping("/users")
CollectionModel<EntityModel<User>> all() {
  return this.userModelAssembler.toCollectionModel(usersClient.users());
}
```

#### Model envelope
Links are created in model assembler.  
And important part is that they are created in a dynamic way.

```java
@Component
class UserModelAssembler implements RepresentationModelAssembler<User, EntityModel<User>> {
  @Override
  public EntityModel<User> toModel(User entity) {
    var controller = HateoasUsersController.class;
    var self = linkTo(methodOn(controller).all()).withRel("all");
    var one = linkTo(methodOn(controller).one(entity.id())).withSelfRel();
    return EntityModel.of(entity, self, one);
  }
}
```

This line says: if someone is calling `all` method on a controller of choosen class,  
then that's called "all" link.

```java
var self = linkTo(methodOn(controller).all()).withRel("all");
```

And when someone calls `one` method, then this is called "self" link

```java
var one = linkTo(methodOn(controller).one(entity.id())).withSelfRel();
```

And you can make these links dynamic.  
So you can dynamically show less or more links,  
based on state of the entity.

```java
@Override
public EntityModel<User> toModel(User entity) {
  if (entity.isSuspended()) {
    // ...
  }
  // ...
}
```

This way server side drives representation of the client side.

#### Complete code
(/my-server-hateoas)

```java
package com.example.web;

import java.util.Collection;

import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.hateoas.CollectionModel;
import org.springframework.hateoas.EntityModel;
import org.springframework.hateoas.server.RepresentationModelAssembler;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.client.RestClient;
import org.springframework.web.client.support.RestClientAdapter;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.linkTo;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.methodOn;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

@Component
class UserModelAssembler implements RepresentationModelAssembler<User, EntityModel<User>> {
  @Override
  public EntityModel<User> toModel(User entity) {
    var controller = HateoasUsersController.class;
    var self = linkTo(methodOn(controller).all()).withRel("all");
    var one = linkTo(methodOn(controller).one(entity.id())).withSelfRel();
    return EntityModel.of(entity, self, one);
  }
}

@Controller
@ResponseBody
class HateoasUsersController {
  private final DeclarativeUsersClient usersClient;
  private final UserModelAssembler userModelAssembler;

  HateoasUsersController(
    DeclarativeUsersClient usersClient,
    UserModelAssembler userModelAssembler
  ) {
    this.usersClient = usersClient;
    this.userModelAssembler = userModelAssembler;
  }

  @GetMapping("/users/{id}")
  EntityModel<User> one(@PathVariable int id) {
    return this.userModelAssembler.toModel(usersClient.user(id));
  }

  @GetMapping("/users")
  CollectionModel<EntityModel<User>> all() {
    return this.userModelAssembler.toCollectionModel(usersClient.users());
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

### MVC (Model View Controller)
Simple static server side. It's still there if you need it.  
Potentially this can work well with htmlx, there is quite good support  
for sending fragments of html in Spring.

If you want to send two fragments of html to update two islands  
of web page, it's possible with Spring and htmlx. And perhaps it's  
possible to update multiple views at the same time.

#### Thymeleaf
One of server templating technologies.  
Template that will render collection of users.

/src/main/resources/templates/users.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>Users</title>

</head>
<body>

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Users</title>
    <meta charset="UTF-8">
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
            font-family: sans-serif;
        }
        th, td {
            border: 1px solid #ccc;
            padding: 8px;
        }
        th {
            background-color: #f5f5f5;
        }
    </style>
</head>
<body>

<h1>Users</h1>

<table>
    <thead>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Username</th>
        <th>Email</th>
        <th>Street</th>
        <th>Suite</th>
        <th>City</th>
        <th>Zipcode</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="user : ${users}">
        <td th:text="${user.id}">1</td>
        <td th:text="${user.name}">John Doe</td>
        <td th:text="${user.username}">johnd</td>
        <td th:text="${user.email}">john@example.com</td>
        <td th:text="${user.address.street}">Main St</td>
        <td th:text="${user.address.suite}">Apt. 4</td>
        <td th:text="${user.address.city}">Springfield</td>
        <td th:text="${user.address.zipcode}">12345</td>
    </tr>
    </tbody>
</table>

</body>
</html>

</body>
</html>
```

#### Controller
Return from the method mvc will tell the name of template to use.  
This is done internally by "ViewResolver". In example below, "users"  
will be resolved to /src/main/resources/templates/users.html

This only works if `@Controller` doesn't have `@ResponseBody`.  
And when you use ResponseBody annotation, then this is a signal,  
that intent of endpoint is different (usually render json object)

```java
@Controller
class MvcController {

  @GetMapping("/users.html")
  String mvc() {
    return "users";
  }

}
```

We send data to view using Model (which is a ViewModel),  
by calling addAttribute on that model.

```java
@GetMapping("/users.html")
String mvc(Model model) {
  model.addAttribute("users", declarativeUsersClient.users());
  return "users";
}
```

#### Complete code
(/my-mvc)

```java
package com.example.web;

import java.util.Collection;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.client.RestClient;
import org.springframework.web.client.support.RestClientAdapter;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

@Controller
class MvcController {
  private final DeclarativeUsersClient declarativeUsersClient;

  MvcController(DeclarativeUsersClient declarativeUserClient) {
    this.declarativeUsersClient = declarativeUserClient;
  }

  @GetMapping("/users.html")
  String mvc(Model model) {
    model.addAttribute("users", declarativeUsersClient.users());
    return "users";
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

Run

```bash
./mvnw spring-boot:run
```

And visit (there has to be "html" at the end)  
http://localhost:8080/users.html

### GraphQL
Ask for as much or as little data as the client needs.

#### Schema
GraphQL is schema first. Schema is defining types.  
Custom defined type can be used as a field in other type.

src/main/resourcs/graphql/users.graphqls

```
type User {
  id: Int
  name: String
  username: String
  email: String
  address: Address
}

type Address {
  street: String
  suite: String
}
```

GraphQL has three verbs:  
queries: reads  
mutations: writes  
subscriptions: long lived reads

And if you want GraphQL to expose endpoint,  
you have to describe it in schema.  
Example of query returning single

```
type Query {
  user: User
}
```

Schema definition for endpoint returning multiple

```
type Query {
  users: [User]
}
```

Complete schema  
src/main/resources/graphql/users.graphqls

```
type User {
    id : Int
    name:String
    username:String
    email: String
    address: Address
}

type Geo {
    lat:Float
    lng: Float
}

type Address {
    street:String
    suite:String
    city:String
    zipcode:String
    geo: Geo
}

type Query {
    users : [User]
}
```

#### Controller
Instead for `@GetMapping`, which is for http we use `@QueryMapping`.

```java
@Controller
class GraphqlController {
  private final DeclarativeUsersClient declarativeUsersClient;

  GraphqlController(DeclarativeUsersClient declarativeUserClient) {
    this.declarativeUsersClient = declarativeUserClient;
  }

  @QueryMapping
  Collection<User> users() {
    return declarativeUsersClient.users();
  }
}
```

#### GraphiQL
A client to test our graph queries.  
Can be enabled in application properties

src/main/resources/application.properties

```
spring.graphql.graphiql.enabled=true
```

After that, go to   
http://localhost:8080/graphiql

Type query

```
query {
  users {
    id, name, username
  }
}
```

And see the result

```
{
  "data": {
    "users": [
      {
        "id": 1,
        "name": "Leanne Graham",
        "username": "Bret"
      },
      {
        "id": 2,
        "name": "Ervin Howell",
        "username": "Antonette"
      },
      {
        "id": 3,
        "name": "Clementine Bauch",
        "username": "Samantha"
      },
      // ...
    ]
  }
}
```

Ask for as much or as little data. We can ask to include address.  
(not whole, we have to specify which parts of address)

```
{
  users {
    id
    name
    username
    address {
      suite
      street
    }
  }
}
```

#### Graph part of GraphQL
Let's imagine that user address is not stored in memory,  
together with user data, but that it's separate microservice just for addresses.  
Called AddressService and it just serves address details.

In that situation we would like to somehow resolve that address for the user.  
Let's pretend we need to call another service to get that address.  
Do to this we will override resolution behaviour, by adding SchemaMapping.

```java
@Controller
class GraphqlController {
  // ...
  
	@SchemaMapping
	Address address (User user) {
		System.out.println("returning address for " + user.id());
		// here imagine a call to another service
		return user.address();
	}
}
```

This will result in N calls visible in console.

```
returning address for 1
returning address for 2
returning address for 3
```

#### Solving N+1 problem
Big problem is that for every user we call address endpoint.  
To solve this, we want to resolve User to Address in batch.  
Schema will stay the same.

For that purpose we will use `@BatchMapping` instead of `@SchemaMapping`.  
Contract for BatchMapping is quite simple: given collections of users,  
return a mapping User to Address, `Map<User, Address>`

```java
@BatchMapping
Map<User, Address> address(Collection<User> users) {
  var addresses = new HashMap<User, Address>();
  System.out.println("getting all addresses [" + users + "]");
  users.forEach(user -> addresses.put(user, user.address()));
  return addresses;
}
```

And now it resolves all the addresses in one call.

#### Hiding implementation details
GraphQL allows to hide implementation details.  
We started with a thing locally, then made a separate service for address  
with resolving address on that service, and the refactored to resolve in batch.

And for all that time client stayed the same.  
Contract with consumer of api stayed all time the same.

#### Complete code
(/my-graphql)

```java
package com.example.web;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.graphql.data.method.annotation.BatchMapping;
import org.springframework.graphql.data.method.annotation.QueryMapping;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.client.RestClient;
import org.springframework.web.client.support.RestClientAdapter;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

@Controller
class GraphqlController {
  private final DeclarativeUsersClient declarativeUsersClient;

  GraphqlController(DeclarativeUsersClient declarativeUserClient) {
    this.declarativeUsersClient = declarativeUserClient;
  }

  @QueryMapping
  Collection<User> users() {
    return declarativeUsersClient.users();
  }

  @BatchMapping
  Map<User, Address> address(Collection<User> users) {
    var addresses = new HashMap<User, Address>();
    System.out.println("getting all addresses [" + users + "]");
    users.forEach(user -> addresses.put(user, user.address()));
    return addresses;
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  HttpServiceProxyFactory httpServiceProxyFactory(RestClient http) {
    return HttpServiceProxyFactory
      .builder()
      .exchangeAdapter(RestClientAdapter.create(http))
      .build();
  }

  @Bean
  DeclarativeUsersClient declarativeUserClient(HttpServiceProxyFactory h) {
    return h.createClient(DeclarativeUsersClient.class);
  }

  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}

interface DeclarativeUsersClient {
  @GetExchange("/users/{id}")
  User user(@PathVariable int id);

  @GetExchange("/users")
  Collection <User> users();
}
```

### gRPC
GraphQL is great and it's easy to love it,  
however if you deeply care about performance,  
then probably you want something like gRPC.

gRPC requires to change the way you write code,  
it's little overbearing, apodictic.

#### Schema
It's also schema first, a bit like graphQL.

We define Users to be RPC service that returns collection of users.  
In gRPC if we want to have collection, there is no array.  
You have to define collection separatelly.

Also everything need to be accounted,  
you need to tell which offset everything has,  
relative to root of type.

```proto
message User {
  int32 id = 1;
  string name = 2;
  string username = 3;
}

message Users {
  repeated User users = 1;
}
```

define UsersService to be rpc style,  
(not fire and forget), and we give it a name "All".

It's going to get nothing.  
There is no default syntax for that, so we import Empty as type.

Since this will be transpiled into Java code,  
we need to provide some options to guide that transpilation.  
We need to instruct target package, classname.

src/main/proto/users.proto

```proto
syntax="proto3";

import "google/protobuf/empty.proto";

option java_package = "com.example.web.grpc";
option java_outer_classname = "UsersProto";
option java_multiple_files = true;

service UsersService {
  rpc All(google.protobuf.Empty) returns (Users){};
}

message User {
  int32 id = 1;
  string name = 2;
  string username = 3;
}

message Users {
  repeated User users = 1;
}
```

After having that schema, we have to generate code,  
using gRPC compiler.

Be carefull, if instead of "All" you name endpoint to Users,  
it will conflict with Users type and compilation will fail.  
(using protobuf is a bit tedious).

```bash
./mvnw -DskipTests package
```

Generated code can be visible inside `/target/generated-sources/grpc-java/`  
There should be two folders grpc-java and java.  
In IntelliJ you have to mark these as source.  
(Eclipse and VisualStusio will not have that problem)

#### Extend generated classes
We take code that was generated and subclass it.  
if we don't override method all, it will call the default implementation,  
(which is to throw an error about not implemented service).

```java
@Service
class UsersService extends UsersServiceGrpc.UsersServiceImplBase {
  // ...

  @Override
  public void all(Empty request, StreamObserver<Users> responseObserver) {
    // ...
  }
}
```

Watch out for autogenerated super call,  
if you leave it as it is, then it will throw error.

```java
super.all(request, responseObserver);
```

In implementation we get collection of users,  
for each one we map through the results  
and we turn each row into grpc package User.  
(we map into code generated User)

for turning rows to grpc Users we call the builder  
and manually assign value to each property.

```java
var all = this.usersClient
  .users()
  .stream()
  .map(u -> com.example.web.grpc.User.newBuilder()
    .setUsername(u.username())
    .setName(u.name())
    .setId(u.id())
    .build())
  .toList();
```

#### Complete code
(/my-grpc)

```java
package com.example.web;

import java.util.Collection;
import java.util.List;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

import com.example.web.grpc.Users;
import com.example.web.grpc.UsersServiceGrpc;
import com.google.protobuf.Empty;

import io.grpc.stub.StreamObserver;

@SpringBootApplication
public class WebApplication {
  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }
}

@Service
class UsersService extends UsersServiceGrpc.UsersServiceImplBase {
  private final SimpleUsersClient usersClient;

  UsersService(SimpleUsersClient usersClient) {
    this.usersClient = usersClient;
  }

  @Override
  public void all(Empty request, StreamObserver<Users> responseObserver) {
    var all = this.usersClient
      .users()
      .stream()
      .map(u -> com.example.web.grpc.User.newBuilder()
        .setUsername(u.username())
        .setName(u.name())
        .setId(u.id())
        .build())
      .toList();

    responseObserver.onNext(Users.newBuilder().addAllUsers(all).build());
    responseObserver.onCompleted();
  }
}

record User(int id, String name, String username, String email, Address address) {}
record Address(String street, String suite, String city, String zipcode, Geo geo) {}
record Geo(float lat, float lng) {}

@Configuration
class WebConfiguration {
  @Bean
  RestClient restClient(RestClient.Builder builder) {
    return builder.baseUrl("https://jsonplaceholder.typicode.com").build();
  }
}

@Component
class SimpleUsersClient {
  private final RestClient http;

  SimpleUsersClient(RestClient http) {
    this.http = http;
  }

  private final ParameterizedTypeReference<List<User>> typeRef = new ParameterizedTypeReference<List<User>>() {};

  Collection<User> users(){
    return this.http
      .get()
      .uri("/users")
      .retrieve()
      .body(typeRef);
  }
}
```

#### Running and testing

```bash
./mvnw spring-boot:run
```

In logs it should be visible that gRPC services are up and running.

```
2025-04-25T16:39:53.378+02:00  INFO 66428 --- [web] [  restartedMain] toConfiguration$GrpcServletConfiguration : Registering gRPC service: UsersService
2025-04-25T16:39:53.378+02:00  INFO 66428 --- [web] [  restartedMain] toConfiguration$GrpcServletConfiguration : Registering gRPC service: grpc.reflection.v1.ServerReflection
2025-04-25T16:39:53.378+02:00  INFO 66428 --- [web] [  restartedMain] toConfiguration$GrpcServletConfiguration : Registering gRPC service: grpc.health.v1.Health
```

And to test we will use `grpcurl`  
setup

```bash
brew install grpcurl
```

call  
it will use HTTP2,  
as this is requirement for gRPC

```bash
grpcurl --plaintext localhost:8080 UsersService.All
```

response

```json
{
  "users": [
    {
      "id": 1,
      "name": "Leanne Graham",
      "username": "Bret"
    },
    {
      "id": 2,
      "name": "Ervin Howell",
      "username": "Antonette"
    },
    {
      "id": 3,
      "name": "Clementine Bauch",
      "username": "Samantha"
    }
  ]
}
```

### Excercise demo project

GraalVM Native Support Developer Tools  
Spring Web Web  
PostgreSQL Driver SQL  
Docker Compose Support Developer Tools  
Spring Data JDBC SQL  
Spring Boot DevTools Developer Tools

#### if you forget to pick them on web site
then it's possible to add them later in pom.xml  
go to https://start.spring.io/, configure as you want  
and preview `pom.xml` file with ctrl + space

and to find out the xml snippet that is needed  
it's possible to find this out on maven repository site:  
https://mvnrepository.com/artifact/org.postgresql/postgresql/42.7.5

```java
// DemoApplication.java
package com.example.demo;

import java.util.Collection;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.annotation.Id;
import org.springframework.data.repository.ListCrudRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}

@Controller
@ResponseBody
class CustomController {
  private final CustomerRepository customerRepository;

  CustomController(CustomerRepository customerRepository) {
    this.customerRepository = customerRepository;
  }

  @GetMapping("/customers")
  Collection<Customer> customers() {
    return customerRepository.findAll();
  }
}

record Customer(@Id int id, String name) {}
interface CustomerRepository extends ListCrudRepository<Customer, Integer> {}
```

#### initialize schema
By convention `/main/resources/schema.sql` will initialize database schema

```sql
-- schema.sql
create table if not exists customer(
  id serial primary key,
  name text
);
```

#### seed
Seed the database has another convention file  
`/main/resources/data.sql`

```sql
-- data.sql
insert into customer(name) values('Maciejka');
insert into customer(name) values('Agata');
```

#### spring config
in application.properties tell spring  
that you want to run init sql

Set application.properties  
enable automatically loading above sql files  
and don't block IO while querying with postgres.

```
spring.application.name=demo
spring.sql.init.mode=always
spring.threads.virtual.enabled=true
spring.docker.compose.lifecycle-management=start_only
```

#### @Id
A part of Spring JDBC  
which is a part of Spring Data

#### install maven dependencies
```bash
./mvnw package
./mvnw -DskipTests package
./mvnw -DskipTests clean package
```

last one will rebuild everything from scratch

#### run
Run (it will start postgres container for you)

```bash
./mvnw spring-boot:run
```

and visit  

http://localhost:8080/customers

#### Build tools
Maven  

Gradle  
more concise, but may be problematic  
It uses Kotlin or Groovy to express its config files  
so you have code before code that you have to have working  
(and not being broken) before you can run the actuall code of app.  
... it's better in that case to have declarative configuration

#### Groovy/Kotlin waiting for new Java
But becaue Gradle uses Kotlin or Groovy, when there is a new Java version  
you will like to install it immediatelly but in Kotlin/Groovy you have to  
wait for language to adopt that new version.

### Actuator
Observability module.  
In production no one hears your application scream.

Actuator will surface state about your application.  
It will show all endpoints available.  
Don't use it on production

https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator/3.4.4

pom.xml
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

application.properties
```
management.endpoints.web.exposure.include=*
```

run project and visit  
http://localhost:8080/actuator  
it contains links to other places

#### instances
http://localhost:8080/actuator/beans  
check how many instances Spring DI created  
and what is their scope (of a lifetime, like one Request)

#### health
http://localhost:8080/actuator/health  
show status UP  

to enable kubernetes liveness and readiness probes  
add this line of config

application.properties
```
management.endpoint.health.probes.enabled=true
```

and then visit two new subpages  
http://localhost:8080/actuator/health/liveness  
http://localhost:8080/actuator/health/readiness

To show disk space, database connection, ssl certificate  
is network ping working, other subsystems  
(also any dependencies you use, they may add something here)

add this line of configuration 

application.properties
```
management.endpoint.health.show-details=always
```

Also it's possible to create own health indicators.  
Just register a bean with annotation @HealthIndicator  
With simple lambda function

If you need some third party API to work,  
create a health indicator to make a test request.

#### info
http://localhost:8080/actuator/info  
Info is blank page  
for a developer to fill

application.properties
```
management.info.env.enabled=true
info.best-course-ever=frontend masters
```

which will be visible as
```
{"best-course-ever":"frontend masters"}
```

#### info page: git commit id
There is a way to show git commit info, the last git commit  
that was before the deployment.

good way to identify build: git commit id (sha generated number)

pom.xml
```xml
<plugin>
  <groupId>io.github.git-commit-id</groupId>
  <artifactId>git-commit-id-maven-plugin</artifactId>
</plugin>
```

(needs restart)

install
```bash
./mvnw -DskipTests clean package
```

visit  
http://localhost:8080/actuator/info

```
{"best-course-ever":"frontend masters","git":{"branch":"main","commit":{"id":"8956a4d","time":"2025-04-25T15:37:12Z"}}}
```

#### conditions
http://localhost:8080/actuator/conditions  
conditions, like is there is some specific class name  
on the classloader path. Page with list of conditions.  
"positiveMatches"  
"negativeMatches"

#### configpros
http://localhost:8080/actuator/configprops  
autocompletion for spring properties  
list of keys that properties files will react to

#### envs
http://localhost:8080/actuator/env  
environment variables  
this one is quite sensitive  
(although it masks a lot of information with `****`)

#### logs
http://localhost:8080/actuator/loggers  
all the loggers and what level they use  
off / error / warn / info / debug / trace

#### heapdump
http://localhost:8080/actuator/heapdump  
to understand where memory is allocated in the profiler

It will download a file that can be opened in profiler.

#### threaddump
http://localhost:8080/actuator/threaddump  
A list of threads and information, which are idle.

#### metrics
http://localhost:8080/actuator/metrics  
cpu usage, connection to database, disk space, request count

to check, visit:  
http://localhost:8080/actuator/metrics/http.server.requests

These metrics pair with another tool "Micrometer".

#### Micrometer
https://micrometer.io/  
They way to do observability for Spring.  
(it can be used also for other things).

It can work with Actuator after some setup.

Was built by Jonathan Schnider, after leaving Netflix  
and joining Spring. It's a person who understands telemetry  
as no many people.

#### Software Bill of Materials
http://localhost:8080/actuator/sbom  
You cannot deploy mistery code to production in some areas.  
Geovernment regulation where you need to provide a list  
of dependencies of your software.

installation  
To run it, add dependency  
"CycloneDX SBOM support"

pom.xml
```xml
<plugin>
  <groupId>org.cyclonedx</groupId>
  <artifactId>cyclonedx-maven-plugin</artifactId>
</plugin>
```

install (and restart)
```bash
./mvnw -DskipTests clean package
```

now visit  
http://localhost:8080/actuator/sbom

returns
```json
{
  "ids": [
    "application"
  ]
}
```

http://localhost:8080/actuator/sbom/application

returns details about dependencies, SHA  
creator, origin

Your security team wants it (perhaps they don't yet realize)  
so that they know, what nonsense you have in the code

#### scheduled tasks
http://localhost:8080/actuator/scheduledtasks

#### mappings
http://localhost:8080/actuator/mappings  
a list of exposed endpoints

info about component that handles them  
and what content type they expect and produce

#### Secure Actuator
By default Actuator only shows three subpages.  
health and two more

a) app is local network only, so only via VPN.  
b) use Spring Security, security configurations (will be talked later).  
c) change default endpoint name

You can remap endpoint from /actuator to any name  
and change port number

#### Actuator has separate thread pool
It will work even if main Spring cannot handle anymore traffic.

### Go to production

#### Build native image
Build native graal image  
and run it

```bash
./mvnw -DskipTests -Pnative native:compile
```

#### Run native image

```bash
cd target
./e2e
```

warning: it will not work when trying `./target/e2e`  
application.properties seems to require being in  
a current working directory at the moment of running e2e.

at this point it will fail  
because it doesn't have credentials for the database

```
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
```

Development env  
It was working in development, because that environment  
runs docker compose and starts database for you. Production  
behaves diferently

#### Outside dev
In production we are working without Docker compose

We need to provide how to connect  
We store these in separate application.properties file  
inside target folder.

target/application.properties
```
spring.datasource.password=secret
spring.datasource.username=myuser
spring.datasource.url=jdbc:postgresql://localhost/mydatabase
```

make sure port is exposed  
(by default it's not)

```yaml
services:
  postgres:
    image: 'postgres:latest'
    environment:
      - 'POSTGRES_DB=mydatabase'
      - 'POSTGRES_PASSWORD=secret'
      - 'POSTGRES_USER=myuser'
    ports:
      - '5432:5432'
```

or make sure you have env variables set  
in a terminal in which you run docker image

```
export SPRING_DATASOURCE_PASSWORD=secret
export SPRING_DATASOURCE_USERNAME=myuser
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost/mydatabase
```

Consider bash script for running run.sh  
Path to docker.io... is taken from the end of previous step,  
in which we builded native image with: `./mvnw -DskipTests -Pnative native:compile`

```bash
#!/usr/bin/env bash

export SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal/mydatabase
export SPRING_DATASOURCE_USERNAME=myuser
export SPRING_DATASOURCE_PASSWORD=secret

docker run \
  -e SPRING_DATASOURCE_PASSWORD=$SPRING_DATASOURCE_PASSWORD \
  -e SPRING_DATASOURCE_URL=$SPRING_DATASOURCE_URL \
  -e SPRING_DATASOURCE_USERNAME=$SPRING_DATASOURCE_USERNAME \
  -p 8080:8080 \
  docker.io/library/e2e:0.0.1-SNAPSHOT
```

pay attention, the datasource url is NOT  
`export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/mydatabase`  
correct value for running in docker is

```bash
export SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal/mydatabase
```

give a run permission

```bash
chmod a+x run.sh
```

#### Build docker image
```bash
./mvnw -DskipTests -Pnative spring-boot:build-image
```

It will create docker image that is also native.

This way is reliable enough to be used on production

#### Typical Spring cases
Data loading and formatting  
Endpoint to serve  
Client of endpoints

We will look into: messaging

### Beyond apps that are database wrapper
Also apps that started as database wrapper  
can be expanded with these capabilities.

#### Messaging
known as:
- Messaging,
- Rabbit,
- Redis Queue,
- Kafka,
- Apache Pulsar,
- Amazon SQS,
- Google Cloud Pub-Sub

#### Kafka
We will choose Kafka because its reliable.  
Kafka is written in Scala, running on JVM

Spring has few layers of support for Kafka  

#### kafka lowest level
At the lowest level there is

"Spring for <X>" project  
Spring for Apache Kafka  
Spring for Apache Pulsar

or Spring AMQP  
AMQP is a general protocol  
of which RabbitMQ is largest implementation

Rabbit team is close to devs at Spring  
but Kafka is the new thing

Docker container for kafka

compose.yml
```
services:
  kafka:
    image: 'apache/kafka-native:4.0.0'
    ports:
      - '9092:9092'
      - '9093:9093'
      - '9094:9094'
      - '2181:2181'
```

run

```bash
docker compose up
```

#### Kafka demo app
We will just send and receive.  
Code is quite easy, configuration is tedious.

Bean to start application  
We are using similar to `@EventListener`, a `@KafkaListener`

#### Kafka Group id
Group id in Kafka is exclusive consumer grouper

if you say you are part of a group you will  
not get messages duplicated across partitions

it's a arbitrary name, just be consistient


#### Kafka client app code
KafkaesqueApplication.java

```java
package com.example.kafkaesque;

import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;

@SpringBootApplication
public class KafkaesqueApplication {

  public static void main(String[] args) {
    SpringApplication.run(KafkaesqueApplication.class, args);
  }

  static final String TOPIC = "dog-adoption-request";

  @Bean
  ApplicationRunner sender(KafkaTemplate<String, DogAdoptionRequest> kafkaTemplate) {
    return _ -> kafkaTemplate.send(TOPIC, new DogAdoptionRequest(1, "Bella"));
  }

  @KafkaListener(topics = TOPIC, groupId = "mygroup")
  void listen(DogAdoptionRequest request) {
    System.out.println("got " + request);
  }
}

record DogAdoptionRequest (int dogId, String dogName) {}
```

#### Client Agnostic: Kafka, RabbitMQ
Kafka, like RabbitMQ, is client agnostic.  
It doesn't require Java, it can work with any language.

#### Not client agnostic: JMS
This is in contrast to JMS  
JMS: very misguided "standard" from Oracle  
25 years ago, meant as abstraction for messaging  
if you ever encounter this, run away.

Problems with JMS  
if you want to upgrade server, you have to upgrade all the clients  
it introduces some terms:  
Topics, Queues, Producer, Consumer  
it's just set of JAVA interfaces, no protocol implied  
any client can break it

#### add processing step
If there is a producer and consumer, with Kafka or RabbitMQ  
its possible to add intermediary computing step  
without need to modify consumer.


#### Annoying Kafka Configuration
Messages travel through the wire, and a part of incomming message  
is a field which says what classname should it deserialize to.  
But for that to work, we have to approve to that

src/main/resources/application.properties

```
spring.application.name=kafkaesque

spring.kafka.producer.key-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.consumer.key-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
```

add dependency for JsonSerializer

pom.xml

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

or add larger dependency

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

add to configuration  
application.properties
```
spring.kafka.consumer.group-id=mygroup
```

then we try to run

```bash
./mvnw spring-boot:run
```

#### JSON over the wire and trust
and get error message about `DogAdoptionRequest`  
not in the trusted packages

We are receiving JSON Payload over the wire and Payload  
has information about class it should be deserialized into.  
Would you blindly trust Payload to choose class?

You need to allow Kafka to do that, to turn JSON into class.

application.properties

```
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

in real scenarios don't do that, use comma list of allowed  
but for demo purpose, we will use *

#### Kafka application.properties

group-id seems to be optional

application.properties
```
spring.application.name=kafkaesque

spring.kafka.producer.key-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.consumer.key-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer

spring.kafka.consumer.properties.spring.json.trusted.packages=*

# this one is optional
spring.kafka.consumer.group-id=mygroup
```

#### Pulsar, Rabbit
It's very easy to switch to other, change configuration and then in code  
change from `@KafkaListener` to `@PulsarListener` or `@Rabbitlistener`.

#### Kafka future
It's going to be big.

### Enterprise Integration Patterns
Book by Bobby Wolf, Hoppe, 2004  
aka "pipes and adapters"

Airflow, popular in Python, even though its underpowered compared to Java  
Biztalk in .net  
they usually need a lot of consulatants  
whole world of enterprise integration  
...

everything today is distributed  
nowadays you have applications that need to talk to each other  
and what if you need something more than messaging bus?  
something more than getting the message and process it

OpenSource, integrations  
when you don't want to pay for 10 consultants  
Mule, MuleSoft, Ross Maison

Mule was huge some years ago

#### Four integration styles
Four styles of integrations:  
(mentioned in the book)  

**1 File Transfer**  
When one system produces files  
which are then input to another system.

**2 Shared Database**  
A bit of antipattern today.  
Very antimicroservices approach.

Having one system write to database  
and another reading from it at the same time.

**3 Remote Procedure Invocation**  
Invocation of methods on remote services.  
Asynchronously.  
What if that message dissapears.

This is probably the worst of options.

**4 Messaging**  
Combines all benefits of previous ones.

Like file transfer  
Not coupled like with file, services don't have to be  
in same space, asynchronous. They only have to agree  
on the location of the file.

RPC requires to be at the same place and the same time

#### Decoupling with messaging
Messaging requires that you know about the same broker.  
You can do "fire and forget". It gives a lot of decoupling.

Final form of distributed system has to be messaging.

You start to see messaging as "I send the message and that updates the state"  
opposed to "I ask the system to give me the state"

#### Spring Integration
Project that supports all these four styles.  
Made by Mark Fisher and Josh Long

Microservices should be mainstream  
they are not at the moment.

#### Pipelines and components
In most examples you have thing that produces events  
and a thing that consumes those events.

A source and a sink.

#### Inbound adapter: consumer
takes message from kafka, file system, ...

Consumers sit in a pipeline  
And there are components that act in that pipeline  
filters, transforming, splitting, aggregating

and finally, when you're done, you write file somewhere  
a result of that processing, out, usually.  
and that's done with

#### Outbound adapter: producer
You may take data and put it to database, send to kafka

the point is: code in the middle is unaware of what code  
in the beginning and in the end did.

code in the middle is unaware of Inbound Adapter.

#### Pipes and filters
It's a pipes and filters architecture

#### Integration flow
We are defining integration flow. It's an arragement of steps.

We could put InboundAdapter directly into IntegrationFlow.from  
and we will use files inbound adapter by the way.

#### Add file integration
we could add like this Kafka, Pulsar, MongoDB

pom.xml
```xml
<dependency>
  <groupId>org.springframework.integration</groupId>
  <artifactId>spring-integration-file</artifactId>
</dependency>
```

#### Using Spring Integrate transform
before transform

EipApplication.java

```java
return IntegrationFlow
  .from(filesInboundAdapter)
  .handle(new GenericHandler<File>() {
    @Override
    public Object handle(File payload, MessageHeaders headers) {
      return null;
    }
  })
  .get();
```

after adding transform

```java
return IntegrationFlow
  .from(filesInboundAdapter)
  .transform(new FileToStringTransformer())
  .handle(new GenericHandler<String>() {
    @Override
    public Object handle(String payload, MessageHeaders headers) {
      return null;
    }
  })
  .get();
```

#### Writing Spring Integration Client

EipApplication.java

```java
package com.example.eip;

import java.io.File;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.integration.core.GenericHandler;
import org.springframework.integration.dsl.IntegrationFlow;
import org.springframework.integration.file.dsl.Files;
import org.springframework.integration.file.transformer.FileToStringTransformer;
import org.springframework.messaging.MessageHeaders;

@SpringBootApplication
public class EipApplication {

  public static void main(String[] args) {
    SpringApplication.run(EipApplication.class, args);
  }

  @Bean
  IntegrationFlow purchaseOrderIntegrationFlow(@Value("file://${HOME}/learn/java-spring-fundamentals/my-integration/purchase-orders/") File file) {
    var filesInboundAdapter = Files
      .inboundAdapter(file)
      .autoCreateDirectory(true);

    return IntegrationFlow
      .from(filesInboundAdapter)
      .transform(new FileToStringTransformer())
      .handle(new GenericHandler<String>() {
        @Override
        public Object handle(String payload, MessageHeaders headers) {
          System.out.println("-------");
          System.out.println("payload: " + payload + ".");
          headers.keySet().forEach(System.out::println);
          return null;
        }
      })
      .get();
  }
}
```

running

```bash
./mvnw spring-boot:run
```

will print

```
payload: {
  "orderId": "PO-1001",
  "country": "USA",
  "lineItems": [
    { "sku": "SKU-001", "productName": "Bluetooth Speaker", "quantity": 2, "unitPrice": 49.99 },
    { "sku": "SKU-002", "productName": "USB-C Cable", "quantity": 3, "unitPrice": 9.99 }
  ],
  "total": 129.95
}
file_originalFile
id
file_name
file_relativePath
timestamp
```

#### Pull/Push on example of email protocols
POP is a protocol you pull

IMAP lets you can establish socket connection  
and receive push notification, so that you know  
immediatelly when you have a new email.

But it's bad for battery.

POP: pull  
IMAP: push

Databases don't push

Messaging Queues do push  
Kafka can tell you when something has changed

how to controll pulling rate
```java
.from(filesInboundAdapter, pc -> pc.poller(pm ->
			pm.fixedRate(Duration.ofMinutes(1))
```

#### Queue
new files will not be given at the same time  
Puller is buffering, it gives one message every second.

It's Staged Event-Driven Archtecture (SEDA)  
you have steps in a sequence,  
each of the steps may get overwhelmed  
so you serialize the results that go into those things  
they get absorbed by intristic buffering  nature

#### Kafka
So when you do true pipes and fiters driven architecture  
use something like Kafka or RabitMQ. You send message to Kafka  
deliver it to component, process it, deliver it to Kafka,  
deliver to com component, process it, deliver to Kafka... etc.

#### Integration with JSON Model
Transform String 

EipApplication.java

```java
return IntegrationFlow
  .from(filesInboundAdapter)
  .transform(new FileToStringTransformer())
  .transform(new JsonToObjectTransformer(PurchaseOrder.class))
  .handle(new GenericHandler<PurchaseOrder>() {
    @Override
    public Object handle(PurchaseOrder payload, MessageHeaders headers) {
      System.out.println("-----------------------------");
      System.out.println("payload: " + payload + ".");
      headers.keySet().forEach(System.out::println);
      payload.lineItems().forEach(System.out::println);
      return null;
    }
  })
  .get();
```

which will print

```
-----------------------------
payload: PurchaseOrder[orderId=PO-1008, country=Australia, lineItems=[LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99], LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]], total=254.97].
file_originalFile
id
file_name
file_relativePath
timestamp
LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99]
LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]
```

#### print one of headers
Supplementary information

EipApplication.java

```java
System.out.println("-----------------------------");
System.out.println("payload: " + payload + ".");
System.out.println(headers.get("file_originalFile"));
```

result

```
payload: PurchaseOrder[orderId=PO-1010, country=Italy, lineItems=[LineItem[sku=SKU-016, productionName=null, quantity=1, unitPrice=349.99]], total=349.99].
/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1010.json
```

#### split along lines of order

When we return null, like we did in handle  
that will stop subsequent execution flow.

EipApplication.java

```java
return IntegrationFlow
  .from(filesInboundAdapter)
  // .from(filesInboundAdapter, pc -> pc.poller(pm ->
  // 			pm.fixedRate(Duration.ofMinutes(1))
  .transform(new FileToStringTransformer())
  .transform(new JsonToObjectTransformer(PurchaseOrder.class))
  .handle(new GenericHandler<PurchaseOrder>() {
    @Override
    public Object handle(PurchaseOrder payload, MessageHeaders headers) {
      System.out.println("-----------------------------");
      System.out.println("payload: " + payload + ".");
      System.out.println(headers.get("file_originalFile"));
      headers.keySet().forEach(System.out::println);
      payload.lineItems().forEach(System.out::println);
      return payload;
    }
  })
  .split(PurchaseOrder.class, po -> po.lineItems())
  .handle(new GenericHandler<LineItem>() {
    @Override
    public Object handle(LineItem payload, MessageHeaders headers) {
      System.out.println("got line item " + payload + '.');
      return null;
    }
  })
  .get();
}
```

returns

```
-----------------------------
payload: PurchaseOrder[orderId=PO-1008, country=Australia, lineItems=[LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99], LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]], total=254.97].
/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1008.json
file_originalFile
id
file_name
file_relativePath
timestamp
LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99]
LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]
got line item LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99].
got line item LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49].
```

#### split and aggregate

And its possible to split it to send to Kafka  
wait for the thing to come back and process it.

```java
return IntegrationFlow
  .from(filesInboundAdapter)
  // .from(filesInboundAdapter, pc -> pc.poller(pm ->
  // 			pm.fixedRate(Duration.ofMinutes(1))
  .transform(new FileToStringTransformer())
  .transform(new JsonToObjectTransformer(PurchaseOrder.class))
  .handle(new GenericHandler<PurchaseOrder>() {
    @Override
    public Object handle(PurchaseOrder payload, MessageHeaders headers) {
      System.out.println("-----------------------------");
      System.out.println("payload: " + payload + ".");
      System.out.println(headers.get("file_originalFile"));
      headers.keySet().forEach(System.out::println);
      payload.lineItems().forEach(System.out::println);
      return payload;
    }
  })
  .split(PurchaseOrder.class, po -> po.lineItems())
  .handle(new GenericHandler<LineItem>() {
    @Override
    public Object handle(LineItem payload, MessageHeaders headers) {
      System.out.println("got line item " + payload + '.');
      return payload;
    }
  })
  .aggregate()
  .handle(new GenericHandler<Object>() {
    @Override
    public Object handle(Object payload, MessageHeaders headers) {
      System.out.println("final payload: " + payload + ".");
      return null;
    }})
  .get();
}
```

We could transform LineItems into network call after split  
and then we would aggregate results.

returns

```
-----------------------------
payload: PurchaseOrder[orderId=PO-1008, country=Australia, lineItems=[LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99], LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]], total=254.97].
/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1008.json
file_originalFile
id
file_name
file_relativePath
timestamp
LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99]
LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]
got line item LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99].
got line item LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49].
final payload: [LineItem[sku=SKU-013, productionName=null, quantity=1, unitPrice=229.99], LineItem[sku=SKU-014, productionName=null, quantity=2, unitPrice=12.49]].
```

#### More integration methods
Has way more

```
.route()
.controlerBusOnRegistry()
.transform()
...
```


#### Complete integration client

EipApplication.java

```java
package com.example.eip;

import java.io.File;
import java.util.Set;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.integration.core.GenericHandler;
import org.springframework.integration.dsl.IntegrationFlow;
import org.springframework.integration.file.dsl.Files;
import org.springframework.integration.file.transformer.FileToStringTransformer;
import org.springframework.integration.json.JsonToObjectTransformer;
import org.springframework.messaging.MessageHeaders;

@SpringBootApplication
public class EipApplication {

  public static void main(String[] args) {
    SpringApplication.run(EipApplication.class, args);
  }

  @Bean
  IntegrationFlow purchaseOrderIntegrationFlow(@Value("file://${HOME}/learn/java-spring-fundamentals/my-integration/purchase-orders/") File file) {
    var filesInboundAdapter = Files
      .inboundAdapter(file)
      .autoCreateDirectory(true);

    return IntegrationFlow
      .from(filesInboundAdapter)
      .transform(new FileToStringTransformer())
      .transform(new JsonToObjectTransformer(PurchaseOrder.class))
      .handle(new GenericHandler<PurchaseOrder>() {
        @Override
        public Object handle(PurchaseOrder payload, MessageHeaders headers) {
          System.out.println("-----------------------------");
          System.out.println("payload: " + payload + ".");
          System.out.println(headers.get("file_originalFile"));
          headers.keySet().forEach(System.out::println);
          payload.lineItems().forEach(System.out::println);
          return payload;
        }
      })
      .split(PurchaseOrder.class, po -> po.lineItems())
      .handle(new GenericHandler<LineItem>() {
        @Override
        public Object handle(LineItem payload, MessageHeaders headers) {
          System.out.println("got line item " + payload + '.');
          return payload;
        }
      })
      .aggregate()
      .handle(new GenericHandler<Object>() {
        @Override
        public Object handle(Object payload, MessageHeaders headers) {
          System.out.println("final payload: " + payload + ".");
          return null;
        }})
      .get();
  }

  record LineItem(String sku, String productionName, int quantity, double unitPrice) {}
  record PurchaseOrder(String orderId, String country, Set<LineItem> lineItems, double total) {}
}
```

#### Reuse integration flows

In integration world I may want to take that whole integration flow  
and make it reusable and available on multiple channels.

#### @Value
https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/value-annotations.html#page-title  
@Value is typically used to inject externalized properties:

```java
@Component
public class MovieRecommender {

	private final String catalog;

	public MovieRecommender(@Value("${catalog.name}") String catalog) {
		this.catalog = catalog;
	}
}
```

With the following configuration:

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

And the following application.properties file:

```
catalog.name=MovieCatalog
```

#### var
```java
Message<String> build = MessageBuilder.withPayload(content).build();
```

is same as

```java
var build = MessageBuilder.withPayload(content).build();
```

#### Refactor and web api
Make reusable integration flows  
Add endpoint to upload file

Allow to have two origins that will produce Message  
Either way we end in purchaseOrderIntegrationFlow

to test run and upload

```bash
./mvnw spring-boot:run
curl -F "file=@/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1010.json" http://localhost:8080/post
```

server logs on first curl run  
(on next one there will bo no initialization)

```
2025-05-29T12:54:45.132+02:00  INFO 53990 --- [eip] [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2025-05-29T12:54:45.132+02:00  INFO 53990 --- [eip] [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2025-05-29T12:54:45.133+02:00  INFO 53990 --- [eip] [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
-----------------------------
payload: PurchaseOrder[orderId=PO-1010, country=Italy, lineItems=[LineItem[sku=SKU-016, productionName=null, quantity=1, unitPrice=349.99]], total=349.99].
null
id
timestamp
LineItem[sku=SKU-016, productionName=null, quantity=1, unitPrice=349.99]
got line item LineItem[sku=SKU-016, productionName=null, quantity=1, unitPrice=349.99].
final payload: [LineItem[sku=SKU-016, productionName=null, quantity=1, unitPrice=349.99]].
```

#### httpie
```bash
brew install httpie
```

these are same

```bash
curl -F "file=@/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1010.json" http://localhost:8080/post
http --form POST :8080/post file@~/Users/maciejka/learn/java-spring-fundamentals/my-integration/purchase-orders/1010.json
```

#### Only care about Inbound
As a principle you should assume anything  
about futher routing or where result goes afterward.

```
Message-processing components should focus only on consuming input (inbound messages).
They shouldn't be concerned with how the result is routed or where it goes afterward.
```

#### Summary, Indirection with Channels
We created channel to create indirection  
between origin of the message and processing of the message.

There are two sources of the message.  
As long as I have the channel, I can send any message into that channel.

origin 1)  
We added Rest controller, with "Post"  
in endpoint handler we are creating message from Post File Payload

origin 2)  
Or I drop a new file in this folder.  
And this folder looks at the new file turns it into string  
and puts it into channel, 

either way message is going to the channel  
and they end up in purchaseOrderIntegrationFlow

(which injects MessageChannel inbound)  
and reads from it immediatelly

We used files adapter, but it could be Kafka, Pulsar it will be almost the same  
instead `Files.inboundAdapter(directory)` you would use `Amqp.inboundAdapter(...)`

#### Refactor full code

EipApplication.java

```java
package com.example.eip;

import java.io.File;
import java.util.Set;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.integration.core.GenericHandler;
import org.springframework.integration.dsl.DirectChannelSpec;
import org.springframework.integration.dsl.IntegrationFlow;
import org.springframework.integration.dsl.MessageChannels;
import org.springframework.integration.file.dsl.Files;
import org.springframework.integration.file.transformer.FileToStringTransformer;
import org.springframework.integration.json.JsonToObjectTransformer;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHeaders;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

@SpringBootApplication
public class EipApplication {

  public static void main(String[] args) {
    SpringApplication.run(EipApplication.class, args);
  }

  @Bean
  DirectChannelSpec inbound() {
    return MessageChannels.direct();
  }

  @Controller
  @ResponseBody
  class PurchaseOrderController {
    private final MessageChannel inbound;

    PurchaseOrderController(MessageChannel inbound) {
      this.inbound = inbound;
    }


    @PostMapping("/post")
    void postPurchaseOrder(@RequestBody MultipartFile file) throws Exception {
      var content = new String(file.getBytes());
      var build = MessageBuilder.withPayload(content).build();
      this.inbound.send(build);
    }
  }

  @Bean
  IntegrationFlow fileInboundFlow (
      MessageChannel inbound,
      @Value("file://${HOME}/learn/java-spring-fundamentals/my-integration/purchase-orders/") File directory) {
    var filesInboundAdapter = Files
      .inboundAdapter(directory)
      .autoCreateDirectory(true);

    return IntegrationFlow
      .from(filesInboundAdapter)
      .transform(new FileToStringTransformer())
      .channel(inbound())
      .get();
  }

  @Bean
  IntegrationFlow purchaseOrderIntegrationFlow(
      MessageChannel inbound) {

    return IntegrationFlow
      .from(inbound)
      .transform(new JsonToObjectTransformer(PurchaseOrder.class))
      .handle(new GenericHandler<PurchaseOrder>() {
        @Override
        public Object handle(PurchaseOrder payload, MessageHeaders headers) {
          System.out.println("-----------------------------");
          System.out.println("payload: " + payload + ".");
          System.out.println(headers.get("file_originalFile"));
          headers.keySet().forEach(System.out::println);
          payload.lineItems().forEach(System.out::println);
          return payload;
        }
      })
      .split(PurchaseOrder.class, po -> po.lineItems())
      .handle(new GenericHandler<LineItem>() {
        @Override
        public Object handle(LineItem payload, MessageHeaders headers) {
          System.out.println("got line item " + payload + '.');
          return payload;
        }
      })
      .aggregate()
      .handle(new GenericHandler<Object>() {
        @Override
        public Object handle(Object payload, MessageHeaders headers) {
          System.out.println("final payload: " + payload + ".");
          return null;
        }})
      .get();
  }

  record LineItem(String sku, String productionName, int quantity, double unitPrice) {}
  record PurchaseOrder(String orderId, String country, Set<LineItem> lineItems, double total) {}
}
```

#### Handling errors
In case something goes wrong,  
you can listen on error channel.

(inside `public class EipApplication`)

```java
@Bean
IntegrationFlow errorIntegrationFlow(@Qualifier("errorchannel") MessageChannel errorChannel) {
  return Integrationflow
    .from(errorChannel)
    ...
  }
```

#### Synchronous by default
By default dispatch from one component to another  
in the same integration flow is synchronous.  
which means they are on the same thread

so you can use database transactions  
in case postPurchaseOrder POST endpoint needs to do all actions

You can put executoner there and make them all asynchronous.  
(And it's very expected that you do that)  
And by definition, if it's handled to Kafka it's asynchronous

#### Messaging / REST Summary
Not everyone is using messaging. I'm argueing: you should be.  
REST is a really bad way to build decoupled systems at scale.

Everyone with experience is moving away from synchronous blocking  
RPC style interactions by default, the have to.

#### Channels
Are tissues in terms of Enterprise Integration.  
You can connect database, so that every message is stored in db.

You can add filters to channels,

#### Security Filter, refuse to process request
For example when doing security  
you may want to validate that person who send request  
has a right to do so

To do it, pack JWT token in the body of the message,  
unpack that in the filter in the channel and call  
OAuth issuer to make sure that this is valid  
and if it doesn't, it's not valid I can refuse to process the request.

#### How to test channels
They are just Beans, so you can inject them in any test code.  
But if you also downcast them, there are bunch of different downcastable versions

Instead of MesssageChannel use SubscribableChannel  
if you do that, you can programmatically call subscribe  
and fix a message after assert whatever you want in that handler

#### Awaitility
And there is also a great library called "Awaitility"  
not part of a Spring, great for things like  
I want to wait 5 seconds and do something.

### Modularity
Java is really good at building large codebases.  
Fast compiler, has modularity, rigity.  
But also because of that it's easy to create a mess

a very large code  
with too many moving parts  
that don't talk well together.

Modularity is super important.  
It's how you keep peace in the organization.  
Your ability to evolve code quickly.

Modularity allows you to stay on your swimlane  
and not having to constantly synchronize  
with rest of system teams.

What slows down software is the need  
of constant synchronization between members.

Mill Conway, computer scientist, coined:
- Null was one billion mistake
- Software reflects organization which built it

If you have four teams working on a compiler  
it will likely be a four pass compiler.

And if you have two components with problems in integration  
that probably means that developer teams behind them  
didn't communicate well.

#### Microservices
Microservices were not about technology,  
there is not a one true technology stack you can use for them.

they are about culture.  
they are about agility  
your ability to make changes quickly  
without having constant interactions with other people

I love microservices. They have complexity cost.  
You have to be that tall to take a ride, to so speak  
It's a very complex thing.  
It's a lot of moving parts.

Anytime you add moving part to system  
you introduce technical complexity  
and therefore risk of technical debt.

At large enough team that complexity is ok to take  
and its actually worth doing it as it will pay in future.  
(Amortyzacja)

For small systems it's too much.  
How do I organize my code to have gains from well organized code?

### Spring Modulith
Organized to stay on rails,  
to build a code so that it's clean and scales well.

#### Modulith Goal
To write a code so that changes have minimal blast radius.

You don't want to have public elements  
that a lot of other parts depend on.  
If they can depend on it, they will.  
Spagetti code.

Follow natural design of the system.

#### Avoid public
Don't use public by default.  
Try to remove public as much as possible.

Fix compiler errors while doing this.  
You should design system so that  
there are few known core interfaces and types  
that the rest of the system depends on

src/main/java/com/example/adoptions/adoptions/DogAdoptionService.java

```java
// don't use public here
public class DogAdoptionService {
```

remove it as much as you can.

#### Spring Boot Dev tools
JetBrains doesn't work right when its added later  
to an already started project in that IDE.  
You have to recreate project in JetBrains.

#### Divide software to modules
If you prefer to have one global space for everything  
just use C language, it's more honest. Don't torture Java like this.

#### Database configuration

application.properties

```
spring.application.name=adoptions

spring.datasource.username=myuser
spring.datasource.password=secret
spring.datasource.url=jdbc:postgresql://localhost/mydatabase
```

#### First version of adoption service

schema.sql

```java
create table if not exists dog
(
    id          serial primary key,
    name        text not null,
    owner       text null,
    description text not null
);

```

data.sql

```sql
delete from dog;

insert into dog (name, owner, description)
values ('Rocky', null, 'A brown Chihuahua known for being protective');

insert into dog (name, owner, description)
values ('Charlie', null, 'A black Bulldog known for being curious.');

insert into dog (name, owner, description)
values ('Duke', null, 'A white German Shepherd known for being friendly.');

insert into dog (id, name, owner, description) values (45, 'Prancer', null, 'a neurotic dog');
```

DogAdoptionService.java

```java
package com.example.adoptions.adoptions;

import org.springframework.data.annotation.Id;
import org.springframework.data.repository.ListCrudRepository;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
class DogController {
  private final DogAdoptionService dogAdoptionService;

  DogController(DogAdoptionService dogAdoptionService) {
    this.dogAdoptionService = dogAdoptionService;
  }

  @PostMapping ("/dogs/{dogId}/adoptions")
  void adopt (@PathVariable int dogId, @RequestParam String owner) {
    dogAdoptionService.adopt(dogId, owner);
  }
}

@Service
@Transactional
class DogAdoptionService {
  private final DogRepository dogRepository;

  DogAdoptionService(DogRepository dogRepository) {
    this.dogRepository = dogRepository;
  }

  void adopt(int dogId, String owner) {
    this.dogRepository.findById(dogId).ifPresent(dog -> {
      var updated = dogRepository.save(new Dog(dog.id(), dog.name(), owner, dog.description()));
      System.out.println("adopted [" + updated + "]");
    });
  }
}

interface DogRepository extends ListCrudRepository<Dog, Integer> {}

record Dog(@Id int id, String name, String owner, String description) {};
```

compose.yml

```yaml
services:
  postgres:
    image: 'postgres:latest'
    environment:
      - 'POSTGRES_DB=mydatabase'
      - 'POSTGRES_PASSWORD=secret'
      - 'POSTGRES_USER=myuser'
    ports:
      - '5432:5432'
```

then, in two terminals

```bash
docker compose up
./mvnw spring-boot:run
```

to observe database

```bash
psql -h localhost -p 5432 -U myuser -d mydatabase
secret
select * from dog;
```

and run

```bash
curl -X POST -F "owner=jlong" http://localhost:8080/dogs/45/adoptions
# or with HTTPie
http --form POST :8080/dogs/45/adoptions owner==jlong
```

#### Add Complication: Veterinary check
This worked, but let's be honest: thats a lot of code  
to just update one dog. And yes, it's pretty trivial.  
It's not realistic. In most real situations you cannot just edit db like that.  
You have to schedule a meeting to see veterinary.

#### Early Solution: public Dogtor Service

src/main/java/com/example/adoptions/vet/Dogtor.java

```java
package com.example.adoptions.vet;

import org.springframework.stereotype.Service;

@Service
public class Dogtor {
  public void schedule(int dogId) {
    System.out.println("scheduling for " + dogId);
  }

}
```

src/main/java/com/example/adoptions/adoptions/DogAdoptionService.java

```java
package com.example.adoptions.adoptions;

import org.springframework.data.annotation.Id;
import org.springframework.data.repository.ListCrudRepository;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import com.example.adoptions.vet.Dogtor;

@Controller
@ResponseBody
class DogController {
  private final DogAdoptionService dogAdoptionService;

  DogController(DogAdoptionService dogAdoptionService) {
    this.dogAdoptionService = dogAdoptionService;
  }

  @PostMapping ("/dogs/{dogId}/adoptions")
  void adopt (@PathVariable int dogId, @RequestParam String owner) {
    dogAdoptionService.adopt(dogId, owner);
  }
}

@Service
@Transactional
class DogAdoptionService {
  private final DogRepository dogRepository;
  private final Dogtor dogtor;

  DogAdoptionService(DogRepository dogRepository, Dogtor dogtor) {
    this.dogRepository = dogRepository;
    this.dogtor = dogtor;
  }

  void adopt(int dogId, String owner) {
    this.dogRepository.findById(dogId).ifPresent(dog -> {
      var updated = dogRepository.save(new Dog(dog.id(), dog.name(), owner, dog.description()));
      dogtor.schedule(dogId);
      System.out.println("adopted [" + updated + "]");
    });
  }
}

interface DogRepository extends ListCrudRepository<Dog, Integer> {}

record Dog(@Id int id, String name, String owner, String description) {};
```

application.properties

```
spring.application.name=adoptions

spring.docker.compose.lifecycle-management=start_only
spring.sql.init.mode=always
spring.modulith.events.republish-outstanding-events-on-restart=true
spring.modulith.events.jdbc.schema-initialization.enabled=true

spring.datasource.username=myuser
spring.datasource.password=secret
spring.datasource.url=jdbc:postgresql://localhost/mydatabase

management.endpoints.web.exposure.include=*
```

runnnig

```bash
curl -X POST -F "owner=jlong" http://localhost:8080/dogs/45/adoptions
```

result

```
scheduling for 45
adopted [Dog[id=45, name=Prancer, owner=jlong, description=a neurotic dog]]
```

#### Asynchronous Event Integration
Service is only calling Dogtor and not waiting for response.
Remove knowledge about Dogtor in DogAdoptionService.
Remove dependency.

https://martinfowler.com/articles/201701-event-driven.html
What do you mean by "Event-Driven"? by Martin Fowler

Event notification
Event with no details, like 

