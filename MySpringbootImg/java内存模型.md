# java内存模型

## 1. 程序执行流程

![Snip20200724_1](/Users/luo/Documents/开发笔记/images/Snip20200724_1.png)

* 首先Java源代码文件(.java后缀)会被Java编译器编译为字节码文件(.class后缀)，
* 然后由JVM中的类加载器加载各个类的字节码文件，
* 加载完毕之后，交由JVM执行引擎执行。
* Java内存模型指的就是Runtime Data Area（运行时数据区），即程序执行期间用到的数据和相关信息保存区。

## 2. java内存模型

根据 JVM 规范，JVM 内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。

![Snip20200724_2](/Users/luo/Documents/开发笔记/images/Snip20200724_2.png)

### （1）PC程序计数器

* 每个线程对应有一个程序计数器。
* 各线程的程序计数器是线程私有的，互不影响，是线程安全的。
* 程序计数器记录线程正在执行的内存地址，以便被中断线程恢复执行时再次按照中断时的指令地址继续执行

### （2）Java栈JavaStack（虚拟机栈JVM Stack）

* 每个线程会对应一个Java栈；
* 每个Java栈由若干栈帧组成；
* 每个方法对应一个栈帧；
* 栈帧在方法运行时，创建并入栈；方法执行完，该栈帧弹出栈帧中的元素作为该方法返回值，该栈帧被清除；
* 栈顶的栈帧叫活动栈，表示当前执行的方法，才可以被CPU执行；
* 线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；
* 栈扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常；

### （3）方法区MethodArea

* 方法区是Java堆的永久区（PermanetGeneration）
* 方法区存放了要加载的类的信息（名称、修饰符等）、类中的静态常量、类中定义为final类型的常量、类中的Field信息、类中的方法信息，
* 方法区是被Java线程共享的
* 方法区要使用的内存超过其允许的大小时，会抛出OutOfMemoryError: PremGen space的错误信息。

### （4）常量池ConstantPool

* 常量池是方法区的一部分。
* 常量池中存储两类数据：字面量和引用量。
* 字面量：字符串、final变量等。
* 引用量：类/接口、方法和字段的名称和描述符，
* 常量池在编译期间就被确定，并保存在已编译的.class文件中

### （5）本地方法栈Native Method Stack

* 本地方法栈和Java栈所发挥的作用非常相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行Native方法服务。
* 本地方法栈也会抛出StackOverflowError和OutOfMemoryError异常。

### （6）Java内存模型工作示意图

![Snip20200724_3](/Users/luo/Documents/开发笔记/images/Snip20200724_3.png)

1)   首先类加载器将Java代码加载到方法区

2)   然后执行引擎从方法区找到main方法

3)   为方法创建栈帧放入方法栈，同时创建该栈帧的程序计数器

4)   执行引擎请求CPU执行该方法

5)   CPU将方法栈数据加载到工作内存（寄存器和高速缓存），执行该方法

6)   CPU执行完之后将执行结果从工作内存同步到主内存

## 2. 多线程特性

线程计算的时候，原始的数据来自内存，在计算过程中，有些数据可能被频繁读取，这些数据被存储在寄存器和高速缓存中，当线程计算完后，这些缓存的数据在适当的时候应该写回内存。

当个多个线程同时读写某个内存数据时，就会产生多线程并发问题，要解决这些问题就涉及到多线程编程三个特性：原子性，有序性，可见性。

### （1） 原子性

原子性，即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

### （2）可见性

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。显然，对于单线程来说，可见性问题是不存在的。

### （3）有序性

有序性即程序执行的顺序按照代码的先后顺序执行。

## 3. 多线程控制类

为了保证多线程的三个特性，Java引入了很多线程控制机制，下面介绍其中常用的几种

* ThreadLocal
* 原子类
* Lock类
* Volatile关键字

### （1）ThreadLocal

```java
public class ThreadLocalDemo {

    /**
     * 用于存放线程的局部变量，同一个线程调用同一个ThreadLocal获取到的值都一样
     * */
    ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
        // 设置Integer的初始值
        protected Integer initialValue() {
            return 0;
        }
    };

    public Integer getThreadLocalValue() {
        return threadLocal.get();
    }

    public void addThreadLocalValue(Integer threadLocalValue) {
        this.threadLocal.set(this.threadLocal.get() + threadLocalValue);
    }

    public static void main(String[] args) {

        ThreadLocalDemo threadLocalDemo = new ThreadLocalDemo();

        Runnable runnable = () -> {
            for (int i = 0; i < 10; i++) {
                threadLocalDemo.addThreadLocalValue(10);
                String name = Thread.currentThread().getName();
                System.out.println(name + "当前的value是：" + threadLocalDemo.getThreadLocalValue());
            }
        };

        for (int i = 0; i < 3; i++) {
            new Thread(runnable).start();
        }
    }
}
```

原理：

* 在ThreadLocal类中定义了一个ThreadLocalMap内部类

  ```java
  public class ThreadLocal<T> {
    
    	static class ThreadLocalMap {
          static class Entry extends WeakReference<ThreadLocal<?>> {
              /** The value associated with this ThreadLocal. */
              Object value;
  
              Entry(ThreadLocal<?> k, Object v) {
                  super(k);
                  value = v;
              }
          }
      }
  }
  ```

  

* 每一个Thread都有一个ThreadLocalMap类型的变量threadLocals

  ```java
  public class Thread implements Runnable {
  		/* ThreadLocal values pertaining to this thread. This map is maintained
       * by the ThreadLocal class. */
      ThreadLocal.ThreadLocalMap threadLocals = null;
  }
  ```

  

* threadLocals内部有一个Entry，Entry的key是ThreadLocal对象实例，value就是共享变量副本

* ThreadLocal的get方法就是根据ThreadLocal对象实例获取共享变量副本

  ```java
  public class ThreadLocal<T> {    
  		public T get() {
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null) {
              ThreadLocalMap.Entry e = map.getEntry(this);
              if (e != null) {
                  @SuppressWarnings("unchecked")
                  T result = (T)e.value;
                  return result;
              }
          }
          return setInitialValue();
      }
      ThreadLocalMap getMap(Thread t) {
          return t.threadLocals;
      }
  }
  ```

  

* ThreadLocal的set方法就是根据ThreadLocal对象实例保存共享变量副本

  ```java
  public class ThreadLocal<T> {
  
  	    public void set(T value) {
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null)
              map.set(this, value);
          else
              createMap(t, value);
      }
  }
  ```

  

### （2）原子类

Java的java.util.concurrent.atomic包里面提供了很多可以进行原子操作的类，分为以下四类：

* 原子更新基本类型：AtomicInteger、AtomicBoolean、AtomicLong
* 原子更新数组：AtomicIntegerArray、AtomicLongArray
* 原子更新引用：AtomicReference、AtomicStampedReference等
* 原子更新属性：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater

提供这些原子类的目的就是为了解决基本类型操作的非原子性导致在多线程并发情况下引发的问题。

实例

```java
/**
 * 线程的原子性：可以看作是SQL的事务，要么全部执行，要么全都不执行
 * */
public class AtomicOperate {

    /**
     * 替换 int i;
     * */
    AtomicInteger atomicInteger;

    public AtomicOperate(AtomicInteger atomicInteger) {
        this.atomicInteger = atomicInteger;
    }

    public void plus100(){
        for (int i = 0; i < 100; i++) {
            /*
            * 替换 i++;
            * */
            atomicInteger.getAndIncrement();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        AtomicOperate atomicOperate = new AtomicOperate(atomicInteger);

        Runnable runnable = ()->{
          atomicOperate.plus100();
        };

        for (int i = 0; i < 100; i++) {
            Thread t1 = new Thread(runnable);
            Thread t2 = new Thread(runnable);

            t1.start();
            t2.start();

            // 使得主线程等待t1,t2的完成
            t1.join();
            t2.join();

            int result = atomicInteger.get();
            System.out.println("最终的结果是：" + result);
        }
    }
}

```

