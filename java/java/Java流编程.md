## Java流编程

### 流是什么

- JDK1.8引入的新成员，以声明式方式处理集合数据
- 将基础操作链接起来，完成复杂的数据处理流水线
- 提供透明的并行处理
- 从支持**数据处理操作**的**源**生成的**元素序列**

### 流和集合的区别

- 时间与空间，流（空间）面向的是计算集合（时间）面向的是存储
- 流只能遍历一次，集合可以遍历多次。
- 集合只能外部迭代，流可以内部迭代。 

### 流的组成

1. 数据源
2. 中间操作：业务处理
3. 终端操作：流数据的收集

![image-20221230101210588](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221230101210588.png)

### 流操作分类

- 中间操作（Intermediate）

  - 无状态操作 — filter / map / peek 等

    不基于全部数据，只需要根据当前数据就能进行的操作。

  - 有状态操作 — dictinct / sorted / limit 等

     基于流中全部数据才能进行的操作，例如去重，排序等。

- 终端操作（Terminal）

  - 非短路操作 — forEach / collect / count 等

    数据遍历的过程中不管是否满足条件，所有数据遍历完成再返回结果。

  - 短路操作 — anyMatch / findFirst / findAny 等

    数据遍历的过程中遇到符合条件的数据就返回结果。

  ![12121](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221230103644908.png)

代码地址：https://github.com/Wxdddd/efficiently

#### 中间操作 - 无状态操作

```java
	/**
     * filter使用：过滤掉不符合断言判断的数据
     */
    @Test
    public void filterTest() {
        list.stream()

                // filter
                .filter(sku ->
                        SkuCategoryEnum.BOOKS
                                .equals(sku.getSkuCategory()))

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }

    /**
     * map使用：将一个元素转换成另一个元素
     */
    @Test
    public void mapTest() {
        list.stream()

                // map
                .map(sku -> sku.getSkuName())

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }

    /**
     * flatMap使用：将一个对象转换成流
     */
    @Test
    public void flatMapTest() {
        list.stream()

                // flatMap
                .flatMap(sku -> Arrays.stream(
                        sku.getSkuName().split("")))

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }

    /**
     * peek使用：对流中元素进行遍历操作，与forEach类似，但不会销毁流元素
     */
    @Test
    public void peek() {
        list.stream()

                // peek
                .peek(sku -> System.out.println(sku.getSkuName()))

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }
```

#### 中间操作 - 有状态操作

```java
    /**
     * sort使用：对流中元素进行排序，可选则自然排序或指定排序规则。有状态操作
     */
    @Test
    public void sortTest() {
        list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                //sort
                .sorted(Comparator.comparing(Sku::getTotalPrice))

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }

    /**
     * distinct使用：对流元素进行去重。有状态操作
     */
    @Test
    public void distinctTest() {
        list.stream()
                .map(sku -> sku.getSkuCategory())

                // distinct
                .distinct()

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));


    }

    /**
     * skip使用：跳过前N条记录。有状态操作
     */
    @Test
    public void skipTest() {
        list.stream()

                .sorted(Comparator.comparing(Sku::getTotalPrice))

                // skip
                .skip(3)

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }

    /**
     * limit使用：截断前N条记录。有状态操作
     */
    @Test
    public void limitTest() {
        list.stream()
                .sorted(Comparator.comparing(Sku::getTotalPrice))

                .skip(2 * 3)

                // limit
                .limit(3)

                .forEach(item ->
                        System.out.println(
                                JSON.toJSONString(
                                        item, true)));
    }
```

#### 终端操作 - 短路操作

```java
	/**
     * allMatch使用：终端操作，短路操作。所有元素匹配，返回true
     */
    @Test
    public void allMatchTest() {
        boolean match = list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                // allMatch
                .allMatch(sku -> sku.getTotalPrice() > 100);

        System.out.println(match);
    }

    /**
     * anyMatch使用：任何元素匹配，返回true
     */
    @Test
    public void anyMatchTest() {
        boolean match = list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                // anyMatch
                .anyMatch(sku -> sku.getTotalPrice() > 100);

        System.out.println(match);
    }

    /**
     * noneMatch使用：任何元素都不匹配，返回true
     */
    @Test
    public void noneMatchTest() {
        boolean match = list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                // noneMatch
                .noneMatch(sku -> sku.getTotalPrice() > 10_000);

        System.out.println(match);
    }

    /**
     * 找到第一个
     */
    @Test
    public void findFirstTest() {
        Optional<Sku> optional = list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                // findFirst
                .findFirst();

        System.out.println(
                JSON.toJSONString(optional.get(), true));
    }

    /**
     * 找任意一个
     */
    @Test
    public void findAnyTest() {
        Optional<Sku> optional = list.stream()

                .peek(sku -> System.out.println(sku.getSkuName()))

                // findAny
                .findAny();

        System.out.println(
                JSON.toJSONString(optional.get(), true));
    }
```

#### 终端操作 - 非短路操作

```java
    /**
     * max使用：
     */
    @Test
    public void maxTest() {
        OptionalDouble optionalDouble = list.stream()
                // 获取总价
                .mapToDouble(Sku::getTotalPrice)

                .max();

        System.out.println(optionalDouble.getAsDouble());
    }

    /**
     * min使用
     */
    @Test
    public void minTest() {
        OptionalDouble optionalDouble = list.stream()
                // 获取总价
                .mapToDouble(Sku::getTotalPrice)

                .min();

        System.out.println(optionalDouble.getAsDouble());
    }

    /**
     * count使用
     */
    @Test
    public void countTest() {
        long count = list.stream()
                .count();

        System.out.println(count);
    }
```

### 流的构成

- 由值创建流
- 由数组创建流
- 由文件生产流
- 由函数生成流（无限流）

```java
/**
 * 流的四种构建形式
 */
public class StreamConstructor {

    /**
     * 由数值直接构建流
     */
    @Test
    public void streamFromValue() {
        Stream stream = Stream.of(1, 2, 3, 4, 5);

        stream.forEach(System.out::println);
    }

    /**
     * 通过数组构建流
     */
    @Test
    public void streamFromArray() {
        int[] numbers = {1, 2, 3, 4, 5};

        IntStream stream = Arrays.stream(numbers);
        stream.forEach(System.out::println);
    }

    /**
     * 通过文件生成流
     * @throws IOException
     */
    @Test
    public void streamFromFile() throws IOException {
        // TODO 此处替换为本地文件的地址全路径
        String filePath = "";

        Stream<String> stream = Files.lines(
                Paths.get(filePath));

        stream.forEach(System.out::println);
    }

    /**
     * 通过函数生成流（无限流）
     */
    @Test
    public void streamFromFunction() {

//        Stream stream = Stream.iterate(0, n -> n + 2);

        Stream stream = Stream.generate(Math::random);

        stream.limit(100)
                .forEach(System.out::println);

    }

}
```

### 流收集器

- 将流中的元素累积成一个结果
- 作用于终端操作collect()上
- collect / Collector / Collectors
  - collect：作为一个终端操作出现，是流收集的最后一个步骤，是一个方法。
  - Collector：是一个接口，Collector方法需要一个实现了Collector接口的收集器。
  - Collectors：是一个工具类，它帮助我们提前封装了一些实现了Collector方法的收集器。

Collectors中预定义的收集器有：

- 将流元素归约和汇总为一个值
- 将流元素分组
- 将流元素分区

```java
/**
 * 描述: Collector 接口
 * @param <T> 流中要收集的项目的泛型
 * @param <A> 累加器的类型，累加器是在收集过程中用于累积部分结果的对象
 * @param <R> 收集操作得到的对象的类型
 * 例如: public class ToList<r> implements Collector<T，List<T>，List<T>>
 *
 * @author wenxudong
 * @date 2022/12/30 16:06
 */
public interface Collector<T, A, R>  {
    // 建立新的结果容器
    Supplier<A> supplier();
    
    // 将元素添加到结果容器
    BiConsumer<A, T> accumulator();

    // 对结果容器应用最终转换
    BinaryOperator<A> combiner();

    // 合并两个结果容器
    Function<A, R> finisher();
    
    // 定义收集器行为，如：是否可以并行，可以使用哪些优化
    Set<java.util.stream.Collector.Characteristics> characteristics();
}
```



```java
/**
 * 常见预定义收集器使用
 */
public class StreamCollector {

    /**
     * 集合收集器
     */
    @Test
    public void toList() {

        List<Sku> list = CartService.getCartSkuList();

        List<Sku> result = list.stream()
                .filter(sku -> sku.getTotalPrice() > 100)

                .collect(Collectors.toList());

        System.out.println(
                JSON.toJSONString(result, true));

    }

    /**
     * 分组
     */
    @Test
    public void group() {
        List<Sku> list = CartService.getCartSkuList();

        // Map<分组条件，结果集合>
        Map<Object, List<Sku>> group = list.stream()
                .collect(
                        Collectors.groupingBy(
                                sku -> sku.getSkuCategory()));

        System.out.println(
                JSON.toJSONString(group, true));
    }

    /**
     * 分区
     */
    @Test
    public void partition() {
        List<Sku> list = CartService.getCartSkuList();

        Map<Boolean, List<Sku>> partition = list.stream()
                .collect(Collectors.partitioningBy(
                        sku -> sku.getTotalPrice() > 100));

        System.out.println(
                JSON.toJSONString(partition, true));
    }

}
```

### 流的归约与汇总

- 归约（reduce）：将Stream流中元素转换成一个**值**

  ```java
      /**
       * 描述: 归约操作接口定义
       *
       * @param identity  初始值
       * @param accumulator  计算逻辑
       * @param combiner  并行执行时多个部分结果的合并方式
       * @return {@link U}
       * @author wenxudong
       * @date 2022/12/30 15:31
       */
      <U> U reduce(U identity,
                   BiFunction<U, ? super T, U> accumulator,
                   BinaryOperator<U> combiner);
  ```

  ![image-20221230153415769](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221230153415769.png)

  ```java
      @Test
      public void reduceTest() {
  
          /**
           * 订单对象
           */
          @Data
          @AllArgsConstructor
          class Order {
              /**
               * 订单编号
               */
              private Integer id;
              /**
               * 商品数量
               */
              private Integer productCount;
              /**
               * 消费总金额
               */
              private Double totalAmount;
          }
  
          /*
              准备数据
           */
          ArrayList<Order> list = Lists.newArrayList();
          list.add(new Order(1, 2, 25.12));
          list.add(new Order(2, 5, 257.23));
          list.add(new Order(3, 3, 23332.12));
  
          /*
              以前的方式：
              1. 计算商品数量
              2. 计算消费总金额
           */
  
          /*
              汇总商品数量和总金额
           */
          Order order = list.stream()
                  .parallel()
                  .reduce(
                          // 初始化值
                          new Order(0, 0, 0.0),
  
                          // Stream中两个元素的计算逻辑
                          (Order order1, Order order2) -> {
                              System.out.println("执行 计算逻辑 方法！！！");
  
                              int productCount =
                                      order1.getProductCount()
                                              + order2.getProductCount();
  
                              double totalAmount =
                                      order1.getTotalAmount()
                                              + order2.getTotalAmount();
  
                              return new Order(0, productCount, totalAmount);
                          },
  
                          // 并行情况下，多个并行结果如何合并
                          (Order order1, Order order2) -> {
                              System.out.println("执行 合并 方法！！！");
  
                              int productCount =
                                      order1.getProductCount()
                                              + order2.getProductCount();
  
                              double totalAmount =
                                      order1.getTotalAmount()
                                              + order2.getTotalAmount();
  
                              return new Order(0, productCount, totalAmount);
                          });
  
          System.out.println(JSON.toJSONString(order, true));
      }
  ```

  

- 汇总（collect）：将Stream流中元素转换成一个**容器**

  ```java
  	/**
       * 描述: 汇总操作接口定义
       *
       * @param supplier  初始化结果容器
       * @param accumulator  添加元素到结果容器逻辑
       * @param combiner  并行执行时多个结果容器的合并方式
       * @return {@link R}
       * @author wenxudong
       * @date 2022/12/30 15:49
       */
      <R> R collect(Supplier<R> supplier,
                    BiConsumer<R, ? super T> accumulator,
                    BiConsumer<R, R> combiner);
  
  	/**
       * 描述: 汇总操作接口定义
       *
       * @param collector  Collector接口实现类
       * @return {@link R}
       * @author wenxudong
       * @date 2022/12/30 16:03
       */
      <R, A> R collect(Collector<? super T, A, R> collector);
  ```

  ```java
  	@Test
      public void collectTest() {
          /**
           * 订单对象
           */
          @Data
          @AllArgsConstructor
          class Order {
              /**
               * 订单编号
               */
              private Integer id;
              /**
               * 用户账号
               */
              private String account;
              /**
               * 商品数量
               */
              private Integer productCount;
              /**
               * 消费总金额
               */
              private Double totalAmount;
          }
  
          /*
              准备数据
           */
          ArrayList<Order> list = Lists.newArrayList();
          list.add(new Order(1, "zhangxiaoxi", 2, 25.12));
          list.add(new Order(2, "zhangxiaoxi",5, 257.23));
          list.add(new Order(3, "lisi",3, 23332.12));
  
          /*
              Map<用户账号, 订单(数量和金额)>
           */
  
          Map<String, Order> collect = list.stream()
                  .parallel()
                  .collect(
                          () -> {
                              System.out.println("执行 初始化容器 操作！！！");
  
                              return new HashMap<String, Order>();
                          },
                          (HashMap<String, Order> map, Order newOrder) -> {
                              System.out.println("执行 新元素添加到容器 操作！！！");
  
                              /*
                                  新元素的account已经在map中存在了
                                  不存在
                               */
                              String account = newOrder.getAccount();
  
                              // 如果此账号已存在，将新订单数据累加上
                              if (map.containsKey(account)) {
                                  Order order = map.get(account);
                                  order.setProductCount(
                                          newOrder.getProductCount()
                                                  + order.getProductCount());
                                  order.setTotalAmount(
                                          newOrder.getTotalAmount()
                                                  + order.getTotalAmount());
                              } else {
                                  // 如果不存在，直接将新订单存入map
                                  map.put(account, newOrder);
                              }
  
                          }, (HashMap<String, Order> map1, HashMap<String, Order> map2) -> {
                              System.out.println("执行 并行结果合并 操作！！！");
  
                              map2.forEach((key, value) -> {
                                  map1.merge(key, value, (order1, order2) -> {
  
                                      // TODO 注意：一定要用map1做合并，因为最后collect返回的是map1
                                      return new Order(0, key,
                                              order1.getProductCount()
                                                      + order2.getProductCount(),
                                              order1.getTotalAmount()
                                                      + order2.getTotalAmount());
                                  });
                              });
                          });
  
          System.out.println(JSON.toJSONString(collect, true));
      }
  ```

  