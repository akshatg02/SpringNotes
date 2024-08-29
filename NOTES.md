# Spring Framework

Servlets - Java technology that you use to handle incoming requests on a server.
Its a java object that has special methods for handling incoming HTTP methods. doGet, doPost, doDelete, doPut

**Spring Framework**

Concepts:
1. IOC Containers
2. Dependency Injection
3. Bean/Beans
4. BeanFactory
5. ApplicationContext/(Aware)
6. Constructor/Setter Injection
7. Object Injection
8. Inner Beans, Aliases, idref
9. Initializing collections
10. Auto-wiring
11. Bean Scopes: Singleton, Prototype

* Main goal is to decouple relationship between dependent objects
* Dependency injection means separate the class which is dependent and inject them from outside when needed
 
 Questions
1. It is necessary to define setter while injecting properties using ApplicationContext???
2. spring.xml needs to be in the src folder while using ApplicationContext
3. Injection can be done either through constructor or setter
4. In case we need objects that are not likely to be used more than once or very frequently, then we use inner beans to define them. In such cases, there is no need to specify an id as the object will never be referenced from the outside world. This does not give any performance related improvement.
5. Beans is analogous to group of class blueprints

* BeanFactory and ApplicationContext are IOC containers. BeanFactory is the most basic version of IOC containers and ApplicationContext extends it.
* BeanFactory - OnDemandLoading, hence lightweight
* ApplicationContext - Lazy Loading (Loads all the beans at the startup)


To conclude, beanPostProcessor is some common code to execute for all type of beans after initialization, while init-methods are only for individual beans.
What confuses me is the name, even though it's called postProcessBeforeInitialization, actually the bean object passed into this method is already initialized and properties set. "BeforeInitialization" here only means before other init logic like init-methods.

The order is as below FYI
1. postProcessBeforeInitialization (Note that here, the bean object itself is already initialized and properties set)
2. InitializingBean
3. init method
4. postProcessAfterInitialization

For Spring Framework, spring-core contains mainly core utilities and common stuff (like enums) and because it's really critical for Spring, probably all other Spring modules depend on it (directly or transitively).
In turn spring-context provides Application Context, that is Spring's Dependency Injection Container and it is probably always defined in POMs of artifacts that use Spring Framework somehow. In fact, spring-context depends on spring-core so by defining spring-context as your dependency, you have spring-core in your classpath as well.

**Tight Coupling**

When objects are highly dependent on one another.

Let's say initially we have just one game Mario and one runner platform.
```
class GameRunner{
	private MarioGame game;	
	
	public GameRunner(MarioGame game){
		this.game = game;
	}
	
	public void play(){
		game.up();
		game.down();
		game.left();
		game.right();
	}
}
```
```
class MarioGame{
	public void up(){
		print("mario up")
	}
	public void down(){
		print("mario down")
	}
	public void left(){
		print("mario left")
	}
	public void right(){
		print("mario right")
	}
}
```
```
public class Main{
	MarioGame game = new MarioGame();
	GameRunner runner = new GameRunner(game);
	runner.play();
}
```

Here if we want to add a new game, Contra, then we will have to either change the argument type in the GameRunner constructor from MarioGame to Contra or add another constructor that accepts Contra object (This is bad as for every new addition of game, we need to change/add code).

This is an example of tight coupling as GameRunner is tightly dependent on the MarioGame class.

We can make this little loose by using an **Interface**.

**Loose Coupling (Using interface)**

```
public interface GamingConsole{
	void up();
	void down();
	void left();
	void right();
}
```

```
@Component
class MarioGame implements GamingConsole{
	public void up(){
		print("mario up")
	}
	public void down(){
		print("mario down")
	}
	public void left(){
		print("mario left")
	}
	public void right(){
		print("mario right")
	}
}
```

```
@Component
class Contra implements GamingConsole{
	public void up(){
		print("contra up")
	}
	public void down(){
		print("contra down")
	}
	public void left(){
		print("contra left")
	}
	public void right(){
		print("contra right")
	}
}
```

```
@Component
class GameRunner{

	@Autowired
	private GamingConsole game;	
	
	public GameRunner(GamingConsole game){
		this.game = game;
	}
	
	public void play(){
		game.up();
		game.down();
		game.left();
		game.right();
	}
}
```

Here we introduced some extent of Loose coupling. By introducing the interface, we eliminated the need to make changes to our GameRunner class everytime a new game is added. Here GameRunner class is loosely coupled with all the games it supports.

**Loose Coupling (Using Spring)**

```
public interface GamingConsole{
	void up();
	void down();
	void left();
	void right();
}
```

```
class MarioGame implements GamingConsole{
	public void up(){
		print("mario up")
	}
	public void down(){
		print("mario down")
	}
	public void left(){
		print("mario left")
	}
	public void right(){
		print("mario right")
	}
}
```

```
class Contra implements GamingConsole{
	public void up(){
		print("contra up")
	}
	public void down(){
		print("contra down")
	}
	public void left(){
		print("contra left")
	}
	public void right(){
		print("contra right")
	}
}
```

```
class GameRunner{
	private GamingConsole game;	
	
	public GameRunner(GamingConsole game){
		this.game = game;
	}
	
	public void play(){
		game.up();
		game.down();
		game.left();
		game.right();
	}
}

public class Main{
	public static void main(String[] args){
		ConfigurableApplicationContext context = SpringApplication.run(Main.class, args);
		GameRunner runner = context.getBean(GameRunner.class);
		runner.play();
	}
}
```

* @Component above any class tells spring that a need may arise in future to create objects of this class which has been annotated as a Component. So, spring will create object of this class.

* @Autowired used before any member variable tells spring that somewhere this class would have been annotated as @Component and you would have created an object of it. Give me that.

So in simple terms @Component annotation is to make Spring Framework aware of a class and @Autowired is to get the instance of that class actually.

e.g.

```
@Component
public class MyCar{
}

public class YourCar{
}

public class Main{
	
	@Autowired
	private MyCar mycar;
	
	@Autowired
	private YourCar yourcar; // this will throw an exception as spring would not know how to create instance of YourCar since it has not been annotated with @Component
}
```

What is happening in the background?
Setting maven logging levels to DEBUG mode can help.

>Identified candidate component class: file [GameRunner.class]
Identified candidate component class: file [MarioGame.class]
Creating shared instance of singletone bean 'gameRunner'
Creating shared instance of singletone bean 'marioGame'
Autowiring by type from bean name 'gameRunner' via constructor to bean named 'marioGame'


**Jargons**

1. @Component/@Service: Class that needs to be managed by Spring. @Service is usually used at Service layer classes. Rest everything is same.
2. Depedency: Any external class that one class needs to function
3. Component Scan: The way in which spring finds the components needed in the current package and all its subpackages (default behaviour). We can specify other locations using @ComponentScan annotation above the main class.
4. Dependency Injection: Identify beans, their dependencies and wire them together. In short, we just tell spring what we want. Then spring takes care of how it is done. Also called inversion of control. As a programmer, we were responsible for creating all objects, wiring them and destroying them. But with IOC, we give that responsibility to the Spring framework.
5. Beans: Objects managed by Spring framework. Whenever spring encounters the component annotations, it creates instance of that class/interface. These instances of components created by Spring are also called Beans.
6. IoC Container: The backbone of spring that implements depdendency injection. e.g. BeanFactory and ApplicationContext(extension of BeanFactory) are two examples of IoC containers.
7. Autowiring: Whenever spring encounters the @Autowired annotation, it tries to find find the instance of the class following the annotation. In a correct implmentation, a class definition with @Component annotation should exist in the package.
8. @Repository: Same as component but used at persistence layer so that it can be used as database repository.
9. @Configuration: Tells spring that this java class has one or more Components.
10. @ComponentScan(basePackages="location where beans could be found")

Spring helps us to focus just on the business logic without worrying much about object lifecycle.

**@Component vs @Bean**
@Component is used for component scanning and automatic wiring. For example, if there is a class that needs to be made available for injection, then we annotate it with @Component. What happens then is, whenever the spring application is initialized, all the classes are scanned (@ComponentScan) and the classes with @Component tags are instantiated and stored in the IoC containers for future use. So, in future whenever we declare a class and annotate it with @Autowired, spring will look for the instance of that class in its IoC container and thereby eliminating the need to explicitly instantiate it.

@Bean annotation above a method tells spring that this method produces a bean to be managed by the spring container. This is a method level annotation. @Bean is used over @Component in situations wherein automatic configuration is not an option. Lets imagine that you want to wire components from a 3rd party library for which you don't have source code so you cannot annotate its classes with @Component. In such a case, we create a method and annotate it with @Bean. This method then returns an object required by the application. The body of the method bears the logic responsible for creating the instance.



### Spring Boot

* Allows us to bootstrap spring apps from scratch.
* It is opinionated
* Convention over configuration
* Stand-alone (ready to run app)


Initial code:

@SpringBootApplication : This annotation declares that CourseApiApp is a Spring Boot application


SpringApplication.run(App.class, args);
class - main class which is the entry point
run() - static method
e.g.
SpringApplication.run(CourseApiApp.class, args);

What the above code does?
1. Sets up default configuration
2. Starts spring application context (Its like a container that hosts different services like data, business logic, controllers, etc)
3. Performs class path scan
4. Starts tomcat server (We did not explicitly install tomcat server, it comes in built with spring boot. That is why spring boot is a stand-alone application)


Spring MVC - Spring framework leveraged by web layer. It is a child project of the Spring framework. It lets you build server side code which maps url with responses.


```
@RestController // comes from spring mvc
public class HelloController {
	
    @RequestMapping("/hello")
    public String sayHi() {
        return "Hi";
    }
}
```

@RestController indicates that the methods contained in that class can be mapped to URLs.

@RequestMapping("<url>") tells Spring that the method following this annotation needs to be executed whenever user hits url specified in the annotation. This is applicable only for GET methods.
	
**Spring jackson** converts the Java objects/collections to json automatically so that we don't have to serialize/desrialize. It is taken care by the @RestController annotation.
	
Bill of materials
	
Resources - Things in your domain.
	
Business services: In spring these are usually singletons. This is achieved using @Service annotation which marks the class as Spring Business Service.
	
	
```
@RequestMapping("/topics/{id}")
public Topic getTopic(@PathVariable String id) {
    // we are delegating this work to another method to make sure that 
    // rest controller does only the routing job and business logic is handled
    // by business service layer to ensure seperation of concern 		
    return topicService.getTopic(id);
}	
```
Here if we want to pass arguments in the url, then we need to catch it in method using @PathVariable annotation.
	
	
```
@RequestMapping(method=RequestMethod.POST, value="/topics")
public void addTopic(@RequestBody Topic topic) {
		
}
```
Here the @RequestBody is telling spring that the request body will be containing json. Take that and convert it into Topic object.
	
**Arrays.asList() creates an immutable list.**
	
Ways to bootstrap Spring Boot application:
1. Using Empty maven project and adding dependencies
2. Using Spring Initializr
3. Using Spring Boot CLI
4. Using Spring STS IDE
	
Customizing Spring Boot Application - This is done using application.properties file located in src/main/resources
e.g. Changing the port on which servelet container runs
	server.port=8081

	
Spring Data JPA (Java Persistence API)

Model - resources
Controller - action taken on resources
View - what the end user gets to see

Mapping models(Classes that defines entities/resources) in Java code

@Entity annotation is used above models. By doing this, a table is created in the DB with the class following @Entity as the table name and the member variables of that class as columns. The values that the class assumes comes in as row.
	
	
```
@Entity
public class Topic {
	
    @Id
	private String id;
	private String name;
	private String desc;
	
	public Topic() {
		
	}
	.
	.
	.

```
@Entity - A table named Topic will be created with 3 columns: id, name, desc.
@Id - This makes id the primary key of the Topic table.
	

Writing a class that talks with the database
	
```
public interface TopicRepository extends CrudRepository<Topic, String>{
	
}
```
