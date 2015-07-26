# [Developing a RESTful Web Service Using Spring Boot](http://kielczewski.eu/2014/04/developing-restful-web-service-with-spring-boot/)

# 使用Spring Boot开发RESTful Web服务

Tag: [Spring](http://kielczewski.eu/tag/Spring/), [Spring Boot](http://kielczewski.eu/tag/Spring%20Boot/), [Spring MVC](http://kielczewski.eu/tag/Spring%20MVC/), [JPA](http://kielczewski.eu/tag/JPA/), [Spring Data JPA](http://kielczewski.eu/tag/Spring%20Data%20JPA/), [TDD](http://kielczewski.eu/tag/TDD/)

I'm really impressed with [Spring Boot](http://projects.spring.io/spring-boot/). Its main goal is to take away most of this boilerplate configuration that was needed before and to replace it with some sensible defaults. One might say it's taking the control away from the developer and giving it to the army of fairies. That's probably right, but in this case the fairies are here to help, and moreover they can easily be cast away from doing particular thing. It's just a matter of doing it yourself as before, and thanks to `@ConditionalOn...` behaviour Spring Boot's auto configuration will not fire up.

我对[Spring Boot](http://projects.spring.io/spring-boot/)的印象非常深刻。它的主要目的就是减少以往需要的大部分初始配置，使用一些默认的设置来代替。可能有人会说这将会使开发者失去控制，而将控制权让给了黑暗魔法。当然这也许对，但是在这里仙子却是来帮忙的，而且他们也可以很容易从特殊的事情去除。它只是像以往你亲历而为所做的那样，由于`@ConditionalOn...`的行为，Spring Boot的自动配置将不会发动。

In the following article I will explore the way of employing Spring Boot to create a very basic, restful web service. As usual the source code can be found [here on GitHub](https://github.com/bkielczewski/example-spring-boot-rest) to play around. 

在接下来的文章里，我将会探索一种采用Spring Boot的方式，创建一个基本的RESTful Web服务。像往常一样，源代码在[GitHub](https://github.com/bkielczewski/example-spring-boot-rest)

## Service overview

The goal will be to create a simple web service with the following requirements:

目标就是构建一个简单的web服务，满足一下需求：

* Given no user with same id exists, it should store a new user in the database and immediately return the stored object.
    - 给一个具有ID的不存在的用户，这将在数据库中存储一个新的用户，并立即返回存储的对象。
* Given there exists a user with same id, it should not store, but return error status with the message.
    - 给一个具有ID的已存在的用户，将不会存储，而是返回错误码信息。
* Given there are previously stored users, it should be able to retrieve the list of them.
    - 给定已存储的用户，将拿到用户的列表。

## Maven

Let's start with creating a `pom.xml`.

让我们从创建`pom.xml`开始。

```xml
 <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://maven.apache.org/POM/4.0.0"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
     <modelVersion>4.0.0</modelVersion>

     <groupId>eu.kielczewski.example.spring</groupId>
     <artifactId>example-spring-boot-rest</artifactId>
     <version>1.0-SNAPSHOT</version>
     <packaging>jar</packaging>

     <parent>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-parent</artifactId>
         <version>1.0.1.RELEASE</version>
     </parent>

     <name>Example Spring Boot REST Service</name>

     <properties>
         <java.version>1.7</java.version>
         <guava.version>16.0.1</guava.version>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
     </properties>

 </project>
```

The relevant thing here is `<parent>` tag - we will be inheriting from Spring Boot's parent POM. It's not absolutely necessary, however it provides a very useful thing which is the dependency management. It has already defined many artifacts we might find useful to use, together with their recent versions supported by Spring, so it really saves up the hassle of tracking them down yourself. We just need to override `java.version` property, which defaults to 1.6, and add version property for [Guava](https://code.google.com/p/guava-libraries/), which is a nice thing to have in handy - but both of these are a matter of personal preference.

需要关注的就是`<parent>`标签————我们将会从Spring Boot的父POM继承而来。这当然不是必须的，但是这有一个非常有用的好处就是依赖管理。它已经定义了很多我们可能会用到的有用的东西，可以被Spring所支持的最新版本，所以真的解决了很多需要自己搜寻的麻烦事。我们需要重写`java.version`属性，默认是1.6，添加[Guava](https://code.google.com/p/guava-libraries/)版本信息，这都是非常好用的东西。————但这都属于个人喜好。

Next thing to do will be to specify the dependencies, which will decide upon the technology stack we will be using:
    
下一件事就是指定我们将会用到的技术栈依赖：

```xml
<dependencies>

    <!-- Spring Boot -->

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Hibernate validator -->

    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
    </dependency>

    <!-- HSQLDB -->

    <dependency>
        <groupId>org.hsqldb</groupId>
        <artifactId>hsqldb</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Guava -->

    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>${guava.version}</version>
    </dependency>

    <!-- Java EE -->

    <dependency>
        <groupId>javax.inject</groupId>
        <artifactId>javax.inject</artifactId>
        <version>1</version>
    </dependency>

</dependencies>
```

When it comes to Spring Boot its functions are spread between the starter modules. The `spring-boot-starter` is the main one, followed by `spring-boot-starter-test` which pulls some nice tools for unit testing including JUnit4 and Mockito. Next comes `spring-boot-starter-web` that pulls Spring MVC dependencies, but also Jackson which will be used for JSON, and most importantly Tomcat, which act as embedded Servlet container. Finally `spring-boot-starter-data-jpa` which is responsible for setting up Spring Data JPA, and comes bundled with Hibernate.



Additional dependencies include Hibernate Validator, as we will be doing some validation. HSQLDB will be the database engine, chosen here because it can be easily embedded and has in memory database feature which is handy for tutorial purposes. Notice I haven't specified versions for these - they are managed by `spring-boot-starter-parent`.

The rest is something of my personal preference - Guava, because it's cool ;) and JSR-330 API to replace `@Autowired` annotation with `@Inject`, which I like better.

Last thing that's left is to add Spring Boot Maven Plugin:
    
    
    <build>
        <plugins>
    
            <!-- Spring Boot Maven -->
    
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
    
        </plugins>
    </build>
    

This plugin does two things:

  * It provides spring-boot:run goal for Maven, so the application can be easily run without packaging.
  * It hooks into `package` goal to produce executable JAR file with all the dependencies included, similar to maven-shade-plugin, but in less messy way.

## Writing the main() method

The execution will start by firing up the main() method, so let's write a class to hold it:
    
    
    @Configuration
    @EnableAutoConfiguration
    @ComponentScan
    public class Application extends SpringBootServletInitializer {
    
        public static void main(final String[] args) {
            SpringApplication.run(Application.class, args);
        }
    
        @Override
        protected final SpringApplicationBuilder configure(final SpringApplicationBuilder application) {
            return application.sources(Application.class);
        }
    }
    

This class has the following features:

  * It acts as a `@Configuration` class for Spring.
  * As such it has `@ComponentScan` annotation that enables scanning for another Spring components in current package and its subpackages.
  * Another annotation is `@EnableAutoConfiguration` which tells Spring Boot to run autoconfiguration.
  * It also extends `SpringBootServletInitializer` which will configure Spring servlet for us, and overrides the `configure()` method to point to itself, so Spring can find the main configuration.
  * Finally, the `main()` method consists of single static call to `SpringApplication.run()`.

At this point this is all that is required to configure the application, so we can start implementing.

## Adding UserController

Let's start with writing a test case for creating a new user through the UserController.
    
    
    @RunWith(MockitoJUnitRunner.class)
    public class UserControllerTest {
    
        @Mock
        private UserService userService;
    
        private UserController userController;
    
        @Before
        public void setUp() {
            userController = new UserController(userService);
        }
    
        @Test
        public void shouldCreateUser() throws Exception {
            final User savedUser = stubServiceToReturnStoredUser();
            final User user = new User();
            User returnedUser = userController.createUser(user);
            // verify user was passed to UserService
            verify(userService, times(1)).save(user);
            assertEquals("Returned user should come from the service", savedUser, returnedUser);
        }
    
        private User stubServiceToReturnStoredUser() {
            final User user = new User();
            when(userService.save(any(User.class))).thenReturn(user);
            return user;
        }
    
    }
    

From this test case we see that in order to create a new user, we need to have `UserController` with `createUser()` method, that takes the `User` object and passes it to the `UserService`, that will be responsible for doing the actual work.

Both `MockitoJUnitRunner.class` and `@Mock` annotation come from Mockito and their purpose is to inject mocked object instead of real implementation of UserService interface. Thanks to this, without the need for a real implementation of `UserService`, I can simulate returning a stored `User` object that comes from the service and verify that it will be exactly the object the `UserController` is going to return. I also check that the `UserService` is going to be called exactly once.

From the REST point of view it will hook up to the POST http method of the `/user` resource.

So let's implement it:
    
    
    @RestController
    public class UserController {
    
        private final UserService userService;
    
        @Inject
        public UserController(final UserService userService) {
            this.userService = userService;
        }
    
        @RequestMapping(value = "/user", method = RequestMethod.POST)
        public User createUser(@RequestBody @Valid final User user) {
            return userService.save(user);
        }
    
    }
    

Points to notice:

  * It's annotated with `@RestController`. The difference between this and `@Controller` annotation is the former also implies `@ResponseBody` on every method, which means there is less to write since from a RESTful web service we are returning JSON objects anyway.
  * `@RequestMapping` maps the `createUser()` to the POST request on the `/user` url.
  * Method takes the `User` object as a parameter. It is created from the body of the request thanks to `@RequestBody` annotation. It is then validated, which is enforced by `@Valid`.
  * The `UserService` will be injected to the constructor, and `User` object is passed to its `save()` method for storage.
  * After storing, the stored `User` object will be returned. Spring will convert it back to JSON automatically, even without `@ResponseBody` annotation which is default for `@RestController`.

How about UserService and User then? For the test to pass we need only an interface for UserService, because it is not even created, but merely mocked. It should take `User` objects into the `save()` methods which will be used to save them, then it should return saved `User` object back to the caller.
    
    
    public interface UserService {
    
        User save(User user);
    
    }
    

As for `User` object it can be anything at this stage, for example:
    
    
    public class User {
    
        @NotNull
        @Size(max = 64)
        private String id;
    
        @NotNull
        @Size(max = 64)
        private String password;
    
        // getters
    
    }
    

Where `@NotNull` and `@Size` are validation constraints the object will be checked against while being deserialized from the request body.

## Adding UserService

Having an interface for `UserService` it would be useful to have an implementation of this `save()` method to do the work for us. Let's start with the test case to see how it might work:
    
    
    @RunWith(MockitoJUnitRunner.class)
    public class UserServiceImplTest {
    
        @Mock
        private UserRepository userRepository;
    
        private UserService userService;
    
        @Before
        public void setUp() {
            userService = new UserServiceImpl(userRepository);
        }
    
        @Test
        public void shouldSaveNewUser() {
            final User savedUser = stubRepositoryToReturnUserOnSave();
            final User user = new User();
            final User returnedUser = userService.save(user);
            // verify repository was called with user
            verify(userRepository, times(1)).save(user);
            assertEquals("Returned user should come from the repository", savedUser, returnedUser);
        }
    
        private User stubRepositoryToReturnUserOnSave() {
            User user = new User();
            when(userRepository.save(any(User.class))).thenReturn(user);
            return user;
        }
    
    }
    

The assumption is that the `UserService` will delegate actual storage to the `UserRepository`, which is mocked, and later stubbed to return a stored `User` object. Then I'm checking whether the object returned from `save()` is the stored one, and that `UserRepository` is called exactly once.

As previously, at this point all we need is the interface for `UserRepository`, but thanks to Spring Data JPA it's also everything that we'll ever need for this project. The interface looks like this:
    
    
    public interface UserRepository extends JpaRepository<User, String> {
    }
    

It just extends `JpaRepository` generic interface with `User` and `String` as type parameters. The former indicates that there will be `User` objects in this repository, latter that it's primary key will be of the `String` type. The `save()` method we need is already there inherited, among other basic CRUD methods.

We won't need to implement this interface, because that's how Spring Data JPA works - it generates the implementation for us.

We need however to make the test to pass by implementing `UserService`:
    
    
    @Service
    public class UserServiceImpl implements UserService {
    
        private final UserRepository repository;
    
        @Inject
        public UserServiceImpl(final UserRepository repository) {
            this.repository = repository;
        }
    
        @Override
        @Transactional
        public User save(final User user) {
            return repository.save(user);
        }
    
    }
    

It's a real simple implementation. Thing to notice is `@Transactional` annotation, that starts the transaction when the method is called as we are going to change the database by inserting a new `User`.

Now the test should pass, but we are inserting `User` object into the database so it needs to be made a proper `@Entity`:
    
    
    @Entity
    public class User {
    
        @Id
        @Column(name = "id", nullable = false, updatable = false)
        @NotNull
        @Size(max = 64)
        private String id;
    
        @Column(name = "password", nullable = false)
        @NotNull
        @Size(max = 64)
        private String password;
    
        // getters
    
    }
    

Notice the `@Entity`, `@Column`, and `@Id` annotations that appeared. The first one tells the object is a JPA entity. The second tells JPA how fields should be mapped to a column and what can be done with them - what the column name will be, whether it's allowed for a column to be updated or have null value. Whereas the `@Id` indicates a primary key for database record - it needs to be non null and unique, so our `User.id` fits here perfectly.

## Getting this to work

It's now possible to run and test the whole thing. Type:
    
    
    mvn spring-boot:run
    

And you should have the web service running on the default port from the current compiled sources.

Alternatively you can build and run the package:
    
    
    mvn package
    java -jar target/example-spring-boot-rest-1.0-SNAPSHOT.jar
    

Having done that now you can:
    
    
    curl -X POST -d '{ "id": "test_id", "password": "test_password" }' http://localhost:8080/user
    

And see whether the response from <http://localhost:8080/> will be like:
    
    
    { "id": "test_id", "password": "test_password" }
    

Which should be our inserted object.

## Checking for duplicate Users

Another requirement for our service is to prevent inserting users if another user already exists with the same id.

Lets add a test case to the `UserServiceTest`:
    
    
    @Test
    public void shouldSaveNewUser_GivenThereExistsOneWithTheSameId_ThenTheExceptionShouldBeThrown() throws Exception {
        stubRepositoryToReturnExistingUser();
        try {
            userService.save(UserUtil.createUser());
            fail("Expected exception");
        } catch (UserAlreadyExistsException ignored) {
        }
        verify(userRepository, never()).save(any(User.class));
    }
    
    private void stubRepositoryToReturnExistingUser() {
        final User user = UserUtil.createUser();
        when(userRepository.findOne(user.getId())).thenReturn(user);
    }
    

That assumes we will ask `UserRepository` about existing user with the same id by calling its `findOne()` method. If such user will be found, the `UserAlreadyExistsException` should be thrown, and the `save()` method on the repository should never be called.

Now we need to change `save()` method in `UserService` implementation, that becomes:
    
    
    @Override
    @Transactional
    public User save(final User user) {
        User existing = repository.findOne(user.getId());
        if (existing != null) {
            throw new UserAlreadyExistsException(
                    String.format("There already exists a user with id=%s", user.getId()));
        }
        return repository.save(user);
    }
    

It clearly has the logic the test requires. The `findOne(id)` method already exists in `UserRepository` and returns null if no object could be found.

The web service will be working fine after that, but once `UserAlreadyExistsException` will be thrown, it will cause INTERNAL SERVER ERROR response for the client using it. We should make sure the response will be different, so the client can more clearly see the situation occurred.

Let's say it should be a response with CONFLICT status with a meaningful error message in the body. To do that we need to add `@ExceptionHandler` to the `UserController`:
    
    
    @ExceptionHandler
    @ResponseStatus(HttpStatus.CONFLICT)
    public String handleUserAlreadyExistsException(UserAlreadyExistsException e) {
        return e.getMessage();
    }
    

It's just a method that intercepts `UserAlreadyExistsException`, returns its message and sets the response status to CONFLICT.

## Closing remarks

I hope that was helpful to get a grasp on what it takes to do a simple web service the Spring Boot way. The implementation of the third requirement about returning all stored users could be a way to spend an evening instead of doing all the things normal people usually do on evenings. If not, take a peek at the [source code](https://github.com/bkielczewski/example-spring-boot-rest) I have prepared sacrificing mine for you.


  

