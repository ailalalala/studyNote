## 1.基础

### 1.1、string

```java
@Test
public void test1(){
    String s3 = "abcd";
    String s5 = "ab";
    String s6 = "cd";
    String s7 = s5+s6;
    final String s1 = "ab";
    final String s2 = "cd";
    String s4 = s1+s2;
    System.out.println(s3 == s7);//结果为false
    System.out.println(s3 == s4);//结果为true
}
```

s5+s6：

- 两个或者两个以上的字符串常量相加，在预编译的时候“+”会被优化，相当于把字符串常量自动合成一个字符串常量
- 字符串的+操作其本质是new了StringBuilder对象进行append操作，拼接后调用toString()返回String对象

使用final修饰String，相当于将s1与s2设置为常量。当final变量是基本数据类型以及String类型时，如果在编译期间能知道它的确切值，则编译器会把它当做编译期常量使用。也就是说在用到该final变量的地方，相当于直接访问的这个常量，不需要在运行时确定。

**intern()方法问题：**

```java
String s1 = new StringBuilder("go").append("od").toString();
System.out.println(s1.intern() == s1);//true
String s2 = new StringBuilder("ja").append("va").toString();
System.out.println(s2.intern() == s2);//false
```

解释：

intern方法是一个native方法，它会去字符串常量池去找这个常量，如果有则返回这个字符串常量池的地址。如果没有则把这个值放在字符串常量池中并返回新字符串的引用。

因为"Java"这个字符串在System类中已经存在了，所以不是第一次创建。

### 1.2、int

如果整型字面量的值在-128到127之间，那么不会new新的Integer对象，而是直接引用常量池中的Integer对象。

```java
Integer i1 = 130;
Integer i2 = 130;
i1 == i2;//false
Integer i3 = 80;
Integer i4 = 80;
i3 == i4;//true
```

### 1.3、&与&&，|与||

&&与||是短路与/或。

### 1.4、关于hashCode()



### 1.5、Collections常用方法

```java
Collections.sort(list);//排序
Collections.shuffle(list);//随机排序
int max = Collections.max(list);
int min = Collections.min(list);
Collections.binarySearch(list2, "Thursday");//查找集合指定元素，返回元素所在索引
Collections.indexOfSubList(list2, subList);//查找子串在集合中首次出现的位置
Collections.lastIndexOfSubList(list2, subList);//查找子串在集合中最后一次出现的位置
Collections.replaceAll(list2, "Sunday", "tttttt");//替换集合中指定的元素，若元素存在返回true，否则返回false
Collections.reverse(list2);//反转集合中的元素的顺序
Collections.rotate(list2, 3);//集合中的元素向后移动k位置，后面的元素出现在集合开始的位置
Collections.copy(list2, list3);//将集合list3中的元素复制到list2中，并覆盖相应索引位置的元素
Collections.swap(list2, 0, 3);//交换集合中指定元素的位置
Collections.fill(list2, "替换");//替换集合中的所有元素，用对象object
Collections.nCopies(5, "哈哈");//生成一个指定大小与内容的集合
```

### 1.6、容器

![image-20210406164945031](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/java面试/java面试.assets/image-20210406164945031.png)

HashMap底层实现原理：数组+链表

put：

![image-20210406172425149](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/java面试/java面试.assets/image-20210406172425149.png)

关于第4步：会先判断单链表节点书是否达到了8，如果达到了8还会判断当前数组的容量是否达到了64，如果没有则扩容，如果是则转为红黑树

get：

![image-20210406172739780](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/java面试/java面试.assets/image-20210406172739780.png)

HashSet底层实现就是HashMap，将值作为HashMap的Key进行存储而实现不可重复性。HashMap的Value都为一个Object对象。





## 2.多线程

### 2.1、多线程基础

线程的操作方法：

**join方法：**

理解为只有调用join方法的线程执行完毕之后再执行主线程。

下面例子只有当AA线程结束后才会执行main线程，如果去掉join方法那么主线程不会等待2秒钟。

```java
public class JoinTest implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"开始执行");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+"执行完毕");
    }
    public static void main(String[] args) throws InterruptedException {
        JoinTest test = new JoinTest();
        Thread thread = new Thread(test,"AA");
        thread.start();
        thread.join();
        System.out.println("主线程运行");
    }
}
```

**sleep方法：**

如果运行中的线程执行了sleep方法，那么回释放cpu资源sleep参数的毫秒数，因为此方法为线程的静态方法，所以该方法不会释放任何锁。**Sleep方法不会放弃 monitor 锁的所有权，会释放CPU资源。**

```java
Thread.sleep(2000);
//也可以使用TimeUnit方法进行休眠
TimeUnit.SECONDS.sleep(2);//HOURS,MINUTES,SECONDS,MILLISECONDS
```

例子证明sleep不释放锁：

```java
public class SleepTest implements Runnable{
    private static final Object locker = new Object();
    @Override
    public void run() {
        synchronized(locker){
            System.out.println(Thread.currentThread().getName()+"开始运行");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"运行结束");
        }
    }
    public static void main(String[] args) throws InterruptedException {
        SleepTest test = new SleepTest();
        Thread thread = new Thread(test);
        Thread thread2 = new Thread(test);
        thread.start();
        thread2.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("main线程运行");
    }
}
```

运行结果：每次都是等到thread线程运行完毕才会运行thread2线程。但是main线程不会受影响该运行还是运行，因为main线程是一个jvm的线程，不需要持有SleepTest的locker锁才可以运行。

**wait方法：**

当我们调用wait方法的时候，它就进入道与该对象相关的等待池，同时释放对象锁，使得其他线程可以访问。直到另一线程调用notify或notifyAll方法来唤醒。

例子说明wait方法释放锁：

```java
public class WaitTest{
    private static Object locker = new Object();
    public static void main(String[] args) throws InterruptedException {
        WaitTest test = new WaitTest();
        Thread thread = new Thread(()->{
            try {
                test.waitTest();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"AA");
        Thread thread2 = new Thread(()->{
            test.notifyTest();
        },"BB");
        thread.start();
        TimeUnit.SECONDS.sleep(2);
        thread2.start();
    }
    private static void waitTest() throws InterruptedException {
        synchronized (locker){
            System.out.println(Thread.currentThread().getName()+"开始等待");
            locker.wait();
            System.out.println(Thread.currentThread().getName()+"等待结束");
        }
    }
    private static void notifyTest(){
        synchronized (locker){
            System.out.println(Thread.currentThread().getName()+"开始唤醒");
            locker.notify();
            System.out.println(Thread.currentThread().getName()+"唤醒结束");
        }
    }
}
```

说明：waitTest与notifyTest方法，要使用同一锁进行wait与notify。并且在同步代码块中也要持有同一锁。

**synchronized持有的各种锁：**





### 2.2、volatile

volitile：保证可见性，禁止指令重排，不保证原子性

可见性：多个线程访问同一共享变量的时候，共享变量放在主内存空间，每个线程需要操作的时候把共享变量拷贝一份放在线程自己的工作内存中，对工作内存的拷贝进行操作。如果变量的拷贝发生了变化，在此线程解锁前会把变化后的内容更新到主内存中，并且对于其他线程的工作内存中的拷贝也同时更新。也就是当前线程的修改对其他线程是可见的。

原子性：是指一个操作或多个操作要么全部执行，且执行的过程不会被任何因素打断，要么就都不执行。

有序性：即程序执行的顺序按照代码的先后顺序执行。

验证可见性代码：

```java
public class MyData {
    volatile int num = 0;
    public void addTo60(){
        num = 60;
    }
}
```

如果num不加volatile修饰，那么下面的main方法中会一直在while循环。

```java
public class MyTest {

    public static void main(String[] args) {
        MyData mydate = new MyData();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+" comming in-------");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mydate.addTo60();
            System.out.println(Thread.currentThread().getName()+" has beenupdate values="+mydate.num);
        },"aaa").start();
        while(mydate.num == 0){

        }
        System.out.println(Thread.currentThread().getName()+" mydate.num="+mydate.num);
    }

}
```

aaa线程修改了num之后，只有添加volatile修饰才会对其他线程立马可见。

不保证原子性验证：

```java
public class MyData {
    volatile int num = 0;
    public void add1(){
        num++;
    }
}
```

```java
public static void main(String[] args) {
        MyData mydate = new MyData();
        for (int i = 0; i < 20; i++) {
            new Thread(()->{
                for (int j = 0; j < 10000; j++) {
                    mydate.add1();
                }
            },String.valueOf(i)).start();
        }

        while(Thread.activeCount()>2){
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName()+" mydate.num="+mydate.num);

  }
```

结果每次都不一致

使用AtomicInteger来实现原子性的操作。

### 2.3、单例模式现下的多线程问题

```java
public class SingleDemo {

    //懒汉式
    private static SingleDemo singleDemo;

    private SingleDemo(){
        System.out.println(Thread.currentThread().getName()+"进入了构造器");
    }

    public static SingleDemo getSingleDemo(){
        if(singleDemo == null){
            singleDemo = new SingleDemo();
        }
        return singleDemo;
    }

}
```

测试

```
public class SingleDemoTest {
    public static void main(String[] args) {
//        System.out.println(SingleDemo.getSingleDemo() == SingleDemo.getSingleDemo());
//        System.out.println(SingleDemo.getSingleDemo() == SingleDemo.getSingleDemo());
//        System.out.println(SingleDemo.getSingleDemo() == SingleDemo.getSingleDemo());

        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                SingleDemo.getSingleDemo();
            }).start();
        }
    }
}
```

上面注释的全为true，并且只进入了一次构造器，这是在单线程的情况。

在下面多线程一起访问的情况下出现了多次进入构造器的情况，这种情况就需要解决了。

DCL（Double Check Lock）双端检锁机制方式解决：

```java
public static SingleDemo getSingleDemo(){
    if(singleDemo == null){
        synchronized (SingleDemo.class){
            if(singleDemo == null){
                singleDemo = new SingleDemo();
            }
        }
    }
    return singleDemo;
}
```

在private static SingleDemo singleDemo;上添加volatile防止指令重排。

### 2.4、CAS

cas：比较并交换