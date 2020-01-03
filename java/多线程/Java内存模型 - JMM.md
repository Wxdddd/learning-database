## Java内存模型 - JMM

### 简介

- Java内存模型描述Java语言的特性，Jvm运行时数据区描述Jvm的特性。

- **`Java虚拟机`**可以同时支持多个执行线程，若未正确同步，线程的行为可能会出现混淆和违反直觉。

- Jvm内存模型描述了**`多线程程序的语义`**；它包含了，**`当多个线程修改了共享内存中的值时，应该读取到哪个值的规则`**。由于这部分规范类似于不同硬件体系结构的内存模型，因此这些语义称为Java编程语言内存模型。
- 这些语义**`没有规定如何`**执行多线程程序。相反，它们描述了允许多线程程序的**`合法行为`**。

> Java内存模型是描述多线程程序的合法行为，与Java虚拟机并没有关系。

### 提出JMM的原因

- 多线程中的问题

1. 所见非所得
2. 无法肉眼去检测程序的准确性
3.  不同的运行平台有不同的表现
4.  错误很难重现

```java
public class DemoVisibility {
    int i = 0;
    boolean isRunning = true;

    public static void main(String args[]) throws InterruptedException {
        DemoVisibility demo = new DemoVisibility();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("here i am...");
                while(demo.isRunning){
                    demo.i++;
                }
                System.out.println("i=" + demo.i);
            }
        }).start();

        Thread.sleep(3000L);
        demo.isRunning = false;
        System.out.println("shutdown...");
    }
}
```

预期结果：在不同的参数，不同版本的JDK下的结果。

| 参数（VM options） | 32位JDK     | 64位JDK     |
| ------------------ | ----------- | ----------- |
| -server            | 不打印i的值 | 不打印i的值 |
| -client            | 打印i的值   | 不打印i的值 |

### volatile关键字

**`可见性`：**让一个线程对共享变量的修改，能够及时的被其他线程看到 。

> Java内存模型中规定：
>
> 对某个 volatile 字段的写操作 happens-before 每个后续对该 volatile 字段的读操作。
>
> **对 volatile 变量 v 的写入，与所有其他线程后续对 v 的读同步**

- volatile如何实现它的语义：

  >- 禁止缓存；
  >
  >  volatile变量的访问控制符会加个ACC_VOLATILE
  >
  >  https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.5
  >
  >- 对volatile变量相关的指令不做重排序；

