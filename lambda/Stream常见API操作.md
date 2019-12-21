## Stream常见API操作

### 1. 聚合操作

> 常规数据中针对批量数据的操作

### 2. Stream处理流程

- 获取数据源

- 数据转换

- 执行操作获取结果

### 3. 获取Stream对象

#### 从集合获取数组中获取

- `Collection.Stream()`
- `Collection.parallelStream()` 并行处理
- `Arrays.stream(T t)`

#### BufferReader获取数组对象

- BufferReader.lines()-> stream()

#### 静态工厂

- `java.util.stream.Instream.range()..`
- `java.nio.file.Files.walk()..`

#### 自定义构建

- `java.util.Spliterator`

#### 更多的方式

- `Random.ints()`
- `Pattern.splitAsStream()`等

### 4. 中间操作API {intermediate}

> 中间操作后的操作结果是一个Stream对象，中间操作可以有`一个或者多个连续的中间操作`，需要注意的是，中间操作`只记录操作方式不做具体执行`，直到结束操作发生时，才做数据的最终执行。

中间操作：就是业务逻辑处理

#### 中间操作过程

- 无状态：

  数据处理时，不受前置中间操作的影响。

  `map/filter/peek/parallel/sequential/unordered`

- 有状态：

  数据处理时，受到前置中间操作的影响。例如对集合截取时前置中间操作有一个排序操作，那么集合截取操作的是排序后的集合。

  `distinct/sorted/limit/skip`

### 5. 终结操作|结束操作{Terminal}

> 需要注意：一个Stream对象，只能有一个Terminal操作，这个操作一旦发生，就会遍历集合真实处理数据。就会完成他的最终执行过程，生成对应的处理结果。并且过程不可逆。

- 非短路操作

  当前的Stream对象必须处理完集合中所有数据，才能得到处理结果。

  `forEach/forEachOrdered/toArray/reduce/collect/min/max/count/iterator`

- 短路操作

  当前Stream对象在处理过程中，一旦满足某个条件，就可以得到结果。无限大的Stream-> 有限大的Stream。

  `anyMatch/allMatch/noneMatch/findFirst/findAny`

```java
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class Main {

    public static void main(String[] args) {
        // 1. 批量数据 -> Stream对象
        // 多个数据
        Stream stream1 = Stream.of("admin", "tom", "damu");

        // 数组
        String[] strArrays = new String[]{"xueqi", "biyao"};
        Stream stream2 = Arrays.stream(strArrays);

        // list
        List<String> list = new ArrayList<>();
        list.add("少林");
        list.add("武当");
        list.add("青城");
        list.add("崆峒");
        list.add("峨眉");
        Stream stream3 = list.stream();

        // set
        Set<String> set = new HashSet<>();
        set.add("少林罗汉拳");
        set.add("武当太极拳");
        set.add("青城剑法");
        Stream stream4 = set.stream();

        // map
        Map<String, Integer> map = new HashMap<>();
        map.put("tom", 1000);
        map.put("jerry", 1200);
        map.put("shuke", 1400);
        Stream stream5 = map.entrySet().stream();

        // 2. Stream 对于基本数据类型的功能封装 int / long / double
        // int
        IntStream.of(new int[]{10, 20, 30}).forEach(System.out::println);
        IntStream.range(1, 5).forEach(System.out::println);//截取1，5 不包含5
        IntStream.rangeClosed(1, 5).forEach(System.out::println);//截取1，5 包含5

        // 3. Stream对象 --> 转换得到指定的数据类型 （stream处理一遍数据就会清空 所以执行时请屏蔽其它stream的处理）
        // 数组
        Object[] objx = stream1.toArray(String[]::new);

        // 字符串
        String str = stream1.collect(Collectors.joining(",")).toString();
        System.out.println(str);

        // list
        List<String> listx = (List<String>) stream1.collect(Collectors.toList());
        System.out.println(listx);

        // set
        Set<String> setx = (Set<String>) stream1.collect(Collectors.toSet());
        System.out.println(setx);

        // map
        Map<String, String> mapx = (Map<String, String>) stream1.collect(Collectors.toMap(x -> x, y -> "value:" + y));
        System.out.println(mapx);

        // 4. Stream中常见的API操作
        List<String> accountList = new ArrayList<>();
        accountList.add("xongjiang");
        accountList.add("lujunyi");
        accountList.add("wuyong");
        accountList.add("linchong");
        accountList.add("luzhishen");
        accountList.add("likui");
        accountList.add("wusong");

        // map() 中间操作 map方法接受一个Function接口
        accountList = accountList.stream().map(x -> "梁山好汉：" + x).collect(Collectors.toList());

        // filter() 添加过滤条件，过滤条件符合的数据 filter中接收一个Predicate接口 若复杂的过滤可以实现Predicate接口来处理
        accountList = accountList.stream().filter(x -> x.length() > 5).collect(Collectors.toList());

        // forEach 增强型循环
        accountList.forEach(x -> System.out.println(x));

        // peek() 中间操作 迭代数据完成数据的依次处理过程（多次循环依次处理）
        accountList.stream()
                .peek(x -> System.out.println("peek 1:" + x))
                .peek(x -> System.out.println("peek 2:" + x))
                .forEach(System.out::println);

        // Stream中对数字运算的支持
        List<Integer> intList = new ArrayList<>();
        intList.add(20);
        intList.add(19);
        intList.add(17);
        intList.add(3);
        intList.add(4);
        intList.add(56);
        intList.add(20);
        intList.add(90);

        // skip() 中间操作， 有状态 跳过部分数据
        intList.stream().skip(3).forEach(System.out::println);

        // limit() 中间操作，有状态，限制输出数据量
        intList.stream().skip(3).limit(2).forEach(System.out::println);

        // distinct() 中间操作，有状态，删除重复数据
        intList.stream().distinct().forEach(System.out::println);

        // max() 中间操作，有状态，获取最大值
        Optional optional = intList.stream().max((x, y) -> x - y);
        // min() 中间操作，有状态，获取最小值

        // reduce() 中间操作，有状态，合并处理数据
        Optional optional2 = intList.stream().reduce((sum, x) -> sum + x);
    }
}
```

