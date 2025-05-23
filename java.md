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

if you forget to pick them on web site,  
then it's possible to add them later in pom.xml

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

And modify files in `/main/resources`  
Create database schema:

```sql
-- schema.sql
create table if not exists customer(
  id serial primary key,
  name text
);
```

Seed the database

```sql
-- data.sql
insert into customer(name) values('Maciejka');
insert into customer(name) values('Agata');
```

Set application.properties  
enable automatically loading above sql files  
and don't block IO while querying with postgres.

```
spring.application.name=demo
spring.sql.init.mode=always
spring.threads.virtual.enabled=true
spring.docker.compose.lifecycle-management=start_only
```

Run (it will start postgres container for you)

```bash
./mvnw spring-boot:run
```

and visit  

http://localhost:8080/customers

#### Actuator
application.properties
```
management.endpoints.web.exposure.include=*
```

https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator/3.4.4

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Actuator will surface state about your application.  
It will show all endpoints available.  
Don't use it on production

http://localhost:8080/actuator  
http://localhost:8080/actuator/health  
http://localhost:8080/actuator/beans

Also it's possible to create own Health Indicators.  
With simple lambda function

http://localhost:8080/actuator/info

Info is blank page  
application.properties

```
management.info.env.enabled=true
info.best-course-ever=frontend masters
```

(needs restart)

There is a way to show git commit info, the last git commit  
that was before the deployment.

#### Micrometer
https://micrometer.io/  
They way to do observability for Spring.  
(it can be used also for other things).

It can work with Actuator after some setup.

#### Secure Actuator
Can be local network only, so only via VPN.  
Can be given different port.  
And handled by Spring security policy.

#### Outside dev
Docker will not be started  
we need to provide how to connect:

target/application.properties
```
spring.datasource.password=secret
spring.datasource.username=myuser
spring.datasource.url=jdbc:postgresql://localhost:5432/mydatabase
```

or make sure you have env variables set  
in a terminal in which you run docker image

```
export SPRING_DATASOURCE_PASSWORD=secret
export SPRING_DATASOURCE_USERNAME=myuser
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/mydatabase
```

or consider bash script for running run.sh

```bash
#!/usr/bin/env bash

export SPRING_DATASOURCE_PASSWORD=secret
export SPRING_DATASOURCE_USERNAME=myuser
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/mydatabase

docker run -e SPRING_DATASOURCE_PASSWORD=$SPRING_DATASOURCE_PASSWORD \
  -e SPRING_DATASOURCE_URL = $SPRING_DATASOURCE_URL \
  -e SPRING_DATASOURCE_USERNAME = $SPRING_DATASOURCE_USERNAME \
  -p 8080:8080 \
  docker.io/library/e2e:0.0.1-SNAPSHOT
```

and give run permission

```bash
chmod a+x run.sh
```


#### Build docker image
```bash
./mvnw -DskipTests -Pnative spring-boot:build-image
```

#### Typical Spring cases
Data loading and formatting  
Endpoint to serve  
Client of endpoints

We will look into: messaging

#### Kafka
Reliable.

Spring has few layers of support for Kafka  
spring Kafka  
spring for Apache Pulsar

RabbitMQ is made from same group of devs as Spring  
Spring AMPQ, open standard to work with RabbitMQ

JMS: very misguided "standard" from Oracle  
25 years ago, meant as abstraction for messaging  
but run away from it

Docker container for kafka

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

Send and receive  
Code is quite easy  
Configuration is tedious

Messages travel through the wire, and a part of incomming message  
is a field which says what classname should it deserialize to.  
But for that to work, we have to approve to that

application.properties

```
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

in real scenarios use comma list of allowed  
but for demo purpose, we will use *

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

OpenSource, integrations  
when you don't want to pay for 10 consultants  
Mule, MuleSoft, Ross Maison

Four styles of integrations:  
(mentioned in the book)  

**File Transfer**  
**Shared Database**  
**Remote Procedure Invocation**  
**Messaging**

#### Spring Integration
Project that supports all these four styles.

Microservices should be mainstream  
they are not at the moment.

### Modularity
Allows to not have to constantly synchronize with other people.  
Mill Conway
- null was one billion mistake
- software reflects team organization

#### Spring Moduleth
If you organize your code in a way  
that rest of the system can depend on it,  
then rest of the system will depend on it  
and it will become a problem

don't use public by default,  
remove it as much as you can.


