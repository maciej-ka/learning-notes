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

#### Spring code, Jurgenization
Code of Spring is legendary,  
it's very backward comp,  
more than linux 

Jurgen: legendary architect  
one of first two people joined Spring  
made a great work of layering abstractions  
that they can handle future changes.

Jurgen is also famous for rewriting PR completely  
"Jurgenization", taking PR, redo it, and merge.

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

not sure
```bash
./mvnw install
```

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

#### Limitation of Java collections
There is no way to specify what is element of collection.  
They way to do it is to bake generic type into superclass.

```java
class UserList extends ArraList<User> {}
```

Parametrized Type Reference  
this also works:  
it will create anonymous private class

```java
List<User> users = new ArrayList<>(){
  //...
}
```

#### Qualifiers
When there is more than one candidate for DI,  
more then one good fit for requested dependency,  
then DI has to be guided by qualifier.

```java
@Bean (name = ...)
```

#### Hypermedia idea, HATEAOS
In very early spec of HTTP, idea was, that every link  
will have role attribute assigned to it.

Idea that with API you get menu which tells,  
how this API can be used to. And also it limits,  
what can you call, depending on current state of application.

Example of this is when you have api for orders that allow to refund,  
you shouldn't be able to even call refund, if you don't have order.  
This way programming becomes easier,  
because you don't have to defend against that case.

For this to work, it needs to be apparent from data,  
what state is application in, and what operations are possible.

It works by server tracing its state and modifying enable API  
depending on changes in that state. This way server in some part  
drive the client.

#### GraphQL
you can ask for as much or as little data as you need

queries  
mutations  
subscriptions: long lived reads

Typically, GraphQL is schema first.

#### GraphiQL
Way to test graph queries

They way to make another query when one of fields is resolved  
but this leads to N+1 problem, 1 to get all, N to get address  
get address for user

```java
@SchemaMapping
Address address (User user) {
  System.out.println("returin address for " + user.id());
  return ...
}
```

so better, use BatchMapping, which will avoid N+1 problem  
the way that we are doing this in batch is hidden for client,  
client will not know is it SchemaMapping or BatchMapping

```java
@BatchMapping
Map <User, Address addresses(Collection<User> users) {
  var addresses = new HashMap<User, Address>()
  users.forEach(User user -> addresses.put(user, ...))
  System.out.println("Resolving")
  return addresses
}
```

#### gRPC
This is also schema first approach.

users.proto

```proto
syntax="proto3"
import "google/protobuf/empty.proto";
option java_package = "com.example.web.grpc"
// ...

service UsersService {
  rpc Users(google.protobuf.Empty) returns (users){}
}

message User {
  int32 id = 1;
  string name = 2;
  string username = 3
}

message Users {
  repeated user users = 1
}
```

That schema has no concept of collection,
so that every time we need a collection,
we need to define separate object
that will represent collection.

After having that schema, we have to generate code, using gRPC compiler
and then we extend that generated code with our code

gRPC requires HTTP2
