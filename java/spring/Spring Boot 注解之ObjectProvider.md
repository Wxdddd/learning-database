## Spring Boot 注解之ObjectProvider



## 自动配置中的ObjectProvider

在阅读Spring Boot自动配置源码的配置时，看到这样如下的自动配置配置源代码。

```java
private final ResourceProperties resourceProperties;

private final WebMvcProperties mvcProperties;

private final ListableBeanFactory beanFactory;

private final WebMvcRegistrations mvcRegistrations;

private ResourceLoader resourceLoader;

public EnableWebMvcConfiguration(ResourceProperties resourceProperties,
                                 ObjectProvider<WebMvcProperties> mvcPropertiesProvider,
                                 ObjectProvider<WebMvcRegistrations> mvcRegistrationsProvider, ListableBeanFactory beanFactory) {
    this.resourceProperties = resourceProperties;
    this.mvcProperties = mvcPropertiesProvider.getIfAvailable();
    this.mvcRegistrations = mvcRegistrationsProvider.getIfUnique();
    this.beanFactory = beanFactory;
}
```

这就是一个常规的基于Java的配置类，那么你是否发现它在用法与其他的有所不同？是的，那就是二个ObjectProvider的参数。这也是本文要讲的内容。

## Spring的注入

在介绍ObjectProvider的使用之前，我们先来回顾一下注入相关的知识。

在Spring的使用过程中，我们可以通过多种形式将一个类注入到另外一个类当中，比如通过@Autowired和@Resources注解。

而@Autowired又可以注解在不同的地方来达到注入的效果，比如注解在构造函数上：

```java
@Service
public class FooService {
    private final FooRepository repository;
    @Autowired
    public FooService(FooRepository repository) {
        this.repository = repository
    }
}
```

注解在属性上：

```java
@Service
public class FooService {
    @Autowired
    private final FooRepository repository;
}
```

注解在setter方法上：

```java
@Service
public class FooService {
    private final FooRepository repository;
    @Autowired
    public void setFooRepository(FooRepository repository) {
        this.repository = repository
    }
}
```

## spring4.3新特性

上面是最常见的注入方式，如果忘记写@Autowired注解，那么在启动的时候就会抛出异常。

但在spring 4.3之后，引入了一个新特性：当构造方法的参数为单个构造参数时，可以不使用@Autowired进行注解。

因此，上面的代码可变为如下形式：

```java
@Service
public class FooService {
    
    private final FooRepository repository;
    
    public FooService(FooRepository repository) {
        this.repository = repository
    }
}
```

使用此种形式便会显得优雅一些。该特性，在Spring Boot的自动配置类中大量被使用。

## 依赖关系的改进

同样是在Spring 4.3版本中，不仅隐式的注入了单构造参数的属性。还引入了ObjectProvider接口。

ObjectProvider接口是ObjectFactory接口的扩展，专门为注入点设计的，可以让注入变得更加宽松和更具有可选项。

那么什么时候使用ObjectProvider接口？

如果待注入参数的Bean为空或有多个时，便是ObjectProvider发挥作用的时候了。

如果注入实例为空时，使用ObjectProvider则避免了强依赖导致的依赖对象不存在异常；如果有多个实例，ObjectProvider的方法会根据Bean实现的Ordered接口或@Order注解指定的先后顺序获取一个Bean。从而了提供了一个更加宽松的依赖注入方式。

Spring 5.1之后提供了基于Stream的orderedStream方法来获取有序的Stream的方法。

使用ObjectProvider之后，上面的代码便变为如下方式：

```java
@Service
public class FooService {
    private final FooRepository repository;
    public FooService(ObjectProvider<FooRepository> repositoryProvider) {
        this.repository = repositoryProvider.getIfUnique();
    }
}
```

这样的好处很显然，当容器中不存在FooRepository或存在多个时，可以从容处理。但坏处也很明显，如果FooRepository不能为null，则可能将异常从启动阶段转移到业务运行阶段。

## ObjectProvider源码

ObjectProvider的源码及解析如下：

```java
public interface ObjectProvider<T> extends ObjectFactory<T>, Iterable<T> {

    // 返回指定类型的bean, 如果容器中不存在, 抛出NoSuchBeanDefinitionException异常
    // 如果容器中有多个此类型的bean, 抛出NoUniqueBeanDefinitionException异常
    T getObject(Object... args) throws BeansException;

    // 如果指定类型的bean注册到容器中, 返回 bean 实例, 否则返回 null
    @Nullable
    T getIfAvailable() throws BeansException;

    // 如果返回对象不存在，则进行回调，回调对象由Supplier传入
    default T getIfAvailable(Supplier<T> defaultSupplier) throws BeansException {
        T dependency = getIfAvailable();
        return (dependency != null ? dependency : defaultSupplier.get());
    }

     // 消费对象的一个实例（可能是共享的或独立的），如果存在通过Consumer回调消耗目标对象。
    default void ifAvailable(Consumer<T> dependencyConsumer) throws BeansException {
        T dependency = getIfAvailable();
        if (dependency != null) {
            dependencyConsumer.accept(dependency);
        }
    }

    // 如果不可用或不唯一（没有指定primary）则返回null。否则，返回对象。
    @Nullable
    T getIfUnique() throws BeansException;

    // 如果存在唯一对象，则调用Supplier的回调函数
    default T getIfUnique(Supplier<T> defaultSupplier) throws BeansException {
        T dependency = getIfUnique();
        return (dependency != null ? dependency : defaultSupplier.get());
    }

    // 如果存在唯一对象，则消耗掉该对象
    default void ifUnique(Consumer<T> dependencyConsumer) throws BeansException {
        T dependency = getIfUnique();
        if (dependency != null) {
            dependencyConsumer.accept(dependency);
        }
    }

    // 返回符合条件的对象的Iterator，没有特殊顺序保证（一般为注册顺序）
    @Override
    default Iterator<T> iterator() {
        return stream().iterator();
    }

    // 返回符合条件对象的连续的Stream，没有特殊顺序保证（一般为注册顺序）
    default Stream<T> stream() {
        throw new UnsupportedOperationException("Multi element access not supported");
    }

    // 返回符合条件对象的连续的Stream。在标注Spring应用上下文中采用@Order注解或实现Order接口的顺序
    default Stream<T> orderedStream() {
        throw new UnsupportedOperationException("Ordered element access not supported");
    }
}
```

其中，在BeanFactory中也使用了该接口来定义方法的返回值：

```java
public interface BeanFactory {

    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
    <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
    ...
}
```

至此，关于ObjectProvider的使用和源码解析完成。