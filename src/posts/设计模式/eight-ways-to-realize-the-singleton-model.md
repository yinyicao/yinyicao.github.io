---
title: 实现单例模式的8种方法
date: 2022-02-08 14:54:01
tags:
  - 单例模式
  - 软件设计
  - 饿汉式
  - 懒汉式
categories: 设计模式
---

> 本篇是在学习[Java内存模型 (ladybug.top)](https://notes.ladybug.top/#/Java/Java线程/Java内存模型)时对JMM的实例记录。

以下记录了八种实现单例模式的方式，这多种方法中有的<span style="color:green">推荐使用</span>、有的虽然不存在线程安全问题但是效率比较低<span style="color:orange">可以使用但不推荐</span>、有的则存在线程安全问题不可使用。

◆单例模式的作用：节省内存和计算、保证结果正确、方便管理

- 节省内存和计算：实例对象创建时需要消耗大量资源，比如计算结果、从数据库读取等，并且这些内容一般不需要改变，对象创建时就固定。
- 保证结果正确：比如在比较耗时的统计计算时，利用了多线程，多线程中的某个对象利用单例模式可保证结果的正确性。
- 方便管理：比如工具类只需要一个实例，更方便管理。

◆单例模式适用场景

1.无状态的工具类：比如日志工具类，不管是在哪里使用，我们需要的只是它帮我们记录日志信息，除此之外，并不需要在它的实例对象上存储任何状态，这时候我们就只需要一个实例对象即可。

2.全局信息类：比如我们在一个类上记录网站的访问次数，我们不希望有的访问被记录在对象A上，有的却记录在对象B上，这时候我们就让这个类成为单例

## 一、饿汉式（静态变量）（可用）

使用静态变量方法实现的饿汉式单例模式是**线程安全**的，可以使用。但是缺点在于，如果单例对象的创建过程比较耗时，那么应用程序的启动将会比较慢，尽早的创建对象会造成资源的浪费。

```java
public class Singleton1 {

    private final static Singleton1 INSTANCE = new Singleton1();


    private Singleton1(){

    }

    public static Singleton1 getInstance(){
        return INSTANCE;
    }
}
```

## 二、饿汉式（静态代码块）（可用）

使用静态代码块方法实现与使用静态变量实现实际上是差不多的，同样是线程安全的。

```java
public class Singleton2 {
    private final static Singleton2 INSTANCE;

    static {
        INSTANCE = new Singleton2();
    }

    public static Singleton2 getInstance(){
        return INSTANCE;
    }
}
```

## 三、懒汉式（不可用）

在不加任何锁的懒汉式单例模式是**线程不安全**的，在多线程并发的情况下具有较大的概率产生多个实例对象。

```java
public class Singleton3 {

    private static Singleton3 instance;

    private Singleton3(){

    }

    public static Singleton3 getInstance(){
        if (instance == null){
            instance = new Singleton3();
        }
        return instance;
    }
}
```

假设在单例类被实例化之前，有两个线程同时在获取单例对象：

线程1在执行完`if (instance == null)` 后，线程调度机制将 CPU 资源分配给线程2，

此时线程2在执行  `if (instance == null)`时也发现单例类还没有被实例化，这样就会导致单例类被实例化两次。

## 四、懒汉式(加方法锁)（不用）

在懒汉式的基础上对获取实例的方法上加synchronized锁，虽然保证了**线程安全**，但是效率极低（每次获取对象都会加锁带来性能损失 ），不推荐使用。

```java
public class Singleton4 {

    private static Singleton4 instance;


    private Singleton4(){

    }

    public synchronized static Singleton4 getInstance(){
        if (instance == null){
            instance = new Singleton4();
        }
        return instance;
    }
}
```

## 五、懒汉式(加对象锁)（不可用）

第四种方式效率太低是因为将synchronized锁加到了方法上，这种方式将synchronized锁加到方法内部，对当前对象加锁。虽然效率得到提示，但是**线程不安全**。与纯懒汉式的单例模式没有太大的区别。

```java
public class Singleton5 {

    private static Singleton5 instance;


    private Singleton5(){

    }

    public static Singleton5 getInstance(){
        if (instance == null){
            synchronized (Singleton5.class){
                instance = new Singleton5();
            }
        }
        return instance;
    }
}
```

> 可能有的会有疑问，如果把synchronized锁放到if外面行不行，就像这样：
>
> ```java
>     public static Singleton5 getInstance(){
>         synchronized (Singleton6.class){
>             if (instance == null) {
>                instance = new Singleton6();
>             }
>          }
>         return instance;
>     }
> ```
>
> 这样也是不行的，这和加方法锁是一样的效果，虽然保证了**线程安全**，但是效率极低，不推荐使用

## 六、懒汉式(双重检查锁)（推荐使用）

使用双重检查锁的方式实现的单例模式，既保证了效率又保证**线程安全**。但是写起来稍微比较复杂，需要注意对实例变量的声明必须使用**volatile**关键字。

```java
public class Singleton6 {

    private volatile static Singleton6 instance;


    private Singleton6(){

    }

    public static Singleton6 getInstance(){
        if (instance == null){
            synchronized (Singleton6.class){
                if (instance == null) {
                    instance = new Singleton6();
                }
            }
        }
        return instance;
    }
}
```

就算在单例类准备被实例化时有多个线程同时通过了第一个`if (instance == null)`的判断，但同一时间也只有一个线程获得锁后进入临界区。通过第一个if检查判断的每个线程会依次获得锁进入临界区，进入临界区后还要再判断一次单例类是否已被其它线程实例化，以避免多次实例化。

由于双重加锁实现仅在实例化单例类时需要加锁，所以相较于加方法锁的实现方式会带来性能上的提升。

另外需要注意的是双重加锁要对 `instance` 域加上 `volatile`修饰符。由于 `synchronized` 并不是对 `instance` 实例进行加锁（因为现在还并没有实例），所以线程在执行`instance = new Singleton6();`完成实例创建，修改 `instance` 的值后，应该将修改后的 `instance` 立即写入主存（`main memory`），而不是暂时存在寄存器或者高速缓冲区（`caches`）中，以保证新的值对其它线程可见。

## 七、懒汉式(静态内部类)（可用）

使用静态内部类的方式创建单例对象，同样可保证**线程安全**和效率并且还是延迟加载，不会像饿汉式那样最开始就创建对象，但是写法比较奇怪。一般也不会直接使用。

```java
public class Singleton7 {

    private Singleton7(){

    }

    public static Singleton7 getInstance(){
       return SingletonInstance.INSTANCE;
    }

    private static class SingletonInstance {
        public static final Singleton7 INSTANCE = new Singleton7();
    }
}
```

## 八、懒汉式(枚举类)（推荐使用）

使用枚举类创建单例模式，写法简单，同时保证**线程安全**。

```java
public enum Singleton8 {
    /**
     * 对象实例
     */
    INSTANCE;

    public void whatever(){
        // 对象的方法、操作
    }

}
```

使用：

```java
public class Singleton8Main {

    public static void main(String[] args) {
        Singleton8.INSTANCE.whatever();
    }
}
```

## 几种实现方式的对比

◆饿汉：简单，但是如果一直不使用该对象造成资源浪费，没有lazy loading

◆懒汉：写法复杂，比较容易造成线程安全问题（如果不对其改进使用双重检查锁或静态内部类就会产生线程安全问题）

◆静态内部类：可用

◆双重检查：面试用

◆枚举：最好

## 各种写法的适用场合

◆最好的方法是利用枚举，因为还可以防止反序列化重新创建新的对象；

◆非线程同步（非线程安全）的方法不能使用；

◆如果程序一开始要加载的资源太多，那么就应该使用懒加载，提升程序启动速度；

◆饿汉式如果是对象的创建需要配置文件等耗时的操作就不适用。

◆懒加载虽然好，但是静态内部类这种方式会引入编程复杂性。

## 思考

1. 哪种方法最好？
   Joshua Blochi大神在《Effective Java》中明确表达过的观点："使用**枚举实现单例**的方法虽然还没有广泛采用，但是单元素的枚举类型已经成为实现Singleton的最佳方法。具有**写法简单**、**线程安全有保障**、**避免反序列化破坏单例（其它方法可以使用反射调用私有构造方法、可以利用反序列化构造多个实例）**等优点。

2. 双重检查锁方式的优点？

   **线程安全；延迟加载；效率较高**

3. 为什么要double-check，只检查一次行不行?

   双重检查是为了**保证线程的安全性**，如果只想检查一次也是可以的，需要将synchronized加到方法上，但是这样效率就会降低。

4. 为何双重检查锁要加volatile关键字？

   这是为了**防止指令重排序**，因为新建对象实际上分为3个步骤：创建空对象、调用构造方法、赋值引用。如果不使用volatile关键字，很有可能会对这三个步骤进行重排序：比如重排序为创建空对象、赋值引用、调用构造方法，就会导致多线程不安全、空指针等问题。

