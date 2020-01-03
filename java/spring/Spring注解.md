## Spring 注解

### Spring注解的三种方式

1. XML
2. 注解

3. 编程方式

### 模式注解（stereotype annotations）

- @Component
- @Service
- @Controller
- @Repository
- @Configuration

@Service、@Controller、@Repository、@Configuration 这四种注解都是以`@Component`注解为基础的。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any
	 */
	String value() default "";

}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

	/**
	 * Explicitly specify the name of the Spring bean definition associated
	 * with this Configuration class.  If left unspecified (the common case),
	 * a bean name will be automatically generated.
	 * <p>The custom name applies only if the Configuration class is picked up via
	 * component scanning or supplied directly to a {@link AnnotationConfigApplicationContext}.
	 * If the Configuration class is registered as a traditional XML bean definition,
	 * the name/id of the bean element will take precedence.
	 * @return the specified bean name, if any
	 * @see org.springframework.beans.factory.support.DefaultBeanNameGenerator
	 */
	String value() default "";

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any
	 */
	String value() default "";

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```

@Component 为最基础的注解，主要将 组件/类/bean 实例化加入到IOC容器中。

@Service、@Controller、@Repository 与 @Component 在当前的spring版本的作用是没有区别的，可以任意使用。他们的作用主要是标明一个类的不同的作用的。

例如：@Service 标明当前类是一种服务。@Controller 标明当前类是一个控制器。@Repository 标明当前类是一个仓储。他们都有 @Component 的功能。

@Configuration 更加灵活强大

- 注入IOC

  ```java
  @Configuration
  public class TestConfig {
      @Bean(name = "testInterface")
      public TestInterface testC() {
          return new TestCImpl();
      }
  }
  ```

- 注入IOC私有成员变量赋值 - 有参构造函数

  ```java
  @Configuration
  public class TestConfig {
      @Bean(name = "testInterface")
      public TestInterface testC() {
          return new TestCImpl("winn","24");
      }
  }
  ```

- 可以替代bean的xml配置

- @Configuration当成配置类去使用

  

### Spring依赖注入时机与延迟实例化

- @Autowired 依赖注入允许null

  `@Autowired(required = false)` required = false 允许注入null

- @Lazy 延迟实例化



### 成员变量注入、Setter注入与构造注入

- 成员变量注入

  ```java
  @RestController
  @RequestMapping("v1/test")
  public class Test2Controller {
  
      @Autowired
      private TestInterface testInterface;
  
      @RequestMapping(value = "list",method = RequestMethod.GET)
      public String count() {
          String disCount = testInterface.disCount(10);
          return disCount;
      }
  }
  ```

- 构造注入

  ```java
  @RestController
  @RequestMapping("v1/test")
  public class Test2Controller {
  
      //@Autowired
      private TestInterface testInterface;
  
      //构造注入
      @Autowired
      public Test2Controller(TestInterface testInterface) {
          this.testInterface = testInterface;
      }
  
      @RequestMapping(value = "list",method = RequestMethod.GET)
      public String count() {
          String disCount = testInterface.disCount(10);
          return disCount;
      }
  }
  ```

- Setter 注入

  ```java
  @RestController
  @RequestMapping("v1/test")
  public class Test2Controller {
  
      //@Autowired
      private TestInterface testInterface;
  
      //Setter 注入
      @Autowired
      public void setTestInterface(TestInterface testInterface) {
          this.testInterface = testInterface;
      }
  
      @RequestMapping(value = "list",method = RequestMethod.GET)
      public String count() {
          String disCount = testInterface.disCount(10);
          return disCount;
      }
  }
  ```

### 包扫描机制

```java
@ComponentScan(value = {"com.siti"})
```

