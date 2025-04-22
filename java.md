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

#### Above Spring 1.0
Spring boot: convention over configuration
