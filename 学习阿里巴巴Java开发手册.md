# 学习《阿里巴巴Java开发手册》

> 经验，是提高自己的重要途径之一。

# 学习笔记：

## 编程规约

### 命名风格

1. “骆驼拼写法”分为两种。第一个词的首字母小写，后面每个词的首字母大写，叫做“小骆驼拼写法”（lowerCamelCase）；第一个词的首字母，以及后面每个词的首字母都大写，叫做“大骆驼拼写法”（UpperCamelCase），又称“帕斯卡拼写法”。类名大骆驼，方法名、参数名、成员变量、局部变量都是小骆驼。
2. POJO类（只有getter和setter方法的简单类）：布尔类型变量不要加is，否则会引起部分框架解析引起序列化错误。
3. 接口类的方法和属性不要加任何修饰符号，包括public。JDK8允许接口有默认实现，default方法。
4. 基于SOA理念，暴露出来的一定是接口，内部实现类加Impl后缀。


### 常量定义

1. 常量的复用层次：
    跨应用共享常量
    应用内共享常量
    子工程内共享常量
    包内共享常量
    类内共享常量： private static final
   
2. 变量值在范围内变化，设置为枚举类。成员名称全部大写。

### OOP规约

1. 直接通过类名访问类的静态变量和静态方法，而不是通过类的对象，增加编译器的解析成本。

2. 所有覆写方法都必须加上@override注解。

3. 所有相同类型的包装对象之间的值的比较，通过equals方法。

    原因：Integer在128  127 分为内的复制，Integer对象是在IntegerCache.cache产生，会复用已有的对象，区间之外的所有数据都在堆上产生，并不会复用已有对象。
    == 不仅比较值的大小，还比较对象的地址。

4. POJO类属性必须使用包装类型，返回值和参数也必须使用包装数据类型。预防NPE现象。

| 基本类型    | 包装器类型     |
| ------- | --------- |
| boolean | Boolean   |
| char    | Character |
| int     | Integer   |
| byte    | Byte      |
| short   | Short     |
| long    | Long      |
| float   | Float     |
| double  | Double    |

5. 使用索引访问String的split方法，需要对最后一个分隔符后面有无内容进行检查，否则会报IndexOutOfBoundsException。

   public String[] split(String regex,int limit)方法：split(String regex) 方法，其实也就等同于split(String regex，0)方法，把结尾的空字符串丢弃！ 

   可以使用split("分隔符",1)或者是org.apache.commons.lang.StringUtils提供的split

   参考文章：[java 字符串split有很多坑，使用时请小心！！](http://yinny.iteye.com/blog/1750210)

6. 循环体内字符串的链接方式：

    用String和“+”：因为“+”拼接字符串，每拼接一次都是再内存重新开辟一个新的内存区域（堆里边）,然后把得到的新的字符串存在这块内存，很容易引起内存溢出。
    使用StringBuilder的append方法进行扩展。是在已有的内存空间追加的字符串。
    commonlang工具包的StringUtils.join(list,",");来一步实现这个拼接而且还能指定分隔的符号。

7. 对象的clone方法默认是前拷贝，实现深拷贝需要重写clone方法。

    基本数据类型的拷贝是没有意义的，String类型这样的引用的拷贝才是有意义的。
    需要注意的是，如果在拷贝一个对象时，要想让这个拷贝的对象和源对象完全彼此独立，那么在引用链上的每一级对象都要被显式的拷贝。所以创建彻底的深拷贝是非常麻烦的，尤其是在引用关系非常复杂的情况下， 或者在引用链的某一级上引用了一个第三方的对象， 而这个对象没有实现clone方法， 那么在它之后的所有引用的对象都是被共享的。
    参考文章：[详解Java中的clone方法 — 原型模式](http://www.importnew.com/16094.html)


### 集合处理


1. 只要重写equals方法，就必须重写hashCode。

    为了保证同一个对象，保证在equals相同的情况下hashcode值必定相同，如果重写了equals而未重写hashcode方法，可能就会出现两个没有关系的对象equals相同的（因为equal都是根据对象的特征进行重写的），但hashcode确实不相同的。
    Set存放不重复队形，先比较hashCode，再用equals比较，提高效率。
    String两个方法都重写了，放心使用。
    参考文章：[为什么重写equals时必须重写hashCode方法？](http://www.cnblogs.com/expiator/p/6064974.html)

2. ArrayList之subList：

    Java.util.List中有一个subList方法，用来返回一个list的一部分的视图。

   ```java
      List<E> subList(int fromIndex, int toIndex);
   ```

    它返回原来list的从[fromIndex, toIndex)之间这一部分的List(下面称之为sublist)，但是这个sublist是依赖于原来的List集合。

    在subList中进行了结构性修改（list大小修改），原来的list的大小也会发生变化，抛出一个ConcurrentModificationException。

3. 集合转为数组的方法，必须使用集合的toArray(T[] array)，类型与大小完全一致。

    不带参数的toArray方法，是构造的一个Object数组，然后进行数据拷贝，此时进行转型就会产生ClassCastException。

4. Arrays.asList()数组转为集合方法，不能使用挂起修改集合相关的方法，如add、remove、clear等，会抛出UnsupportedOperationException异常。

     设计模式：适配器模式。只是转换接口，后台的数据仍是数组。

      Arrays.asList方法返回的ArrayList是继承自AbstractList同时实现了RandomAccess和Serializable接口，AbstractList定义add等方法抛出异常。

      解决方法：

        ```java
        //1
        List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
        //2
        int i[]={11,22,33};  
        Arrays.asList(ArrayUtils.toObject(i));
        ```


5. Java泛型通配符PECS原则：

    如果要从集合中读取类型T的数据，并且不能写入，可以使用 ? extends 通配符；(Producer Extends)

    如果要从集合中写入类型T的数据，并且不需要读取，可以使用 ? super 通配符；(Consumer Super)

    如果既要存又要取，那么就不要使用任何通配符。

6. 不要在foreach循环中进行元素的remove/add操作。如果要remove，需在Iterator中。如果并发，需给Iterator加锁。

    List类会在内部维护一个modCount的变量，用来记录修改次数。

    每生成一个Iterator，Iterator就会记录该modCount，每次调用next()方法就会将该记录与外部类List的modCount进行对比，发现不相等就会抛出多线程编辑异常。

    ```java
     //foreach和迭代器的hasNext()方法，foreach这个语法糖，实际上就是
     while(itr.hasNext()){
         itr.next()
     }
     ```

    ```java
     public boolean hasNext() {
         return cursor != size;
     }
     ```

    cursor是用于标记迭代器位置的变量，该变量由0开始，每次调用next执行+1操作。

    > 你的代码在执行删除“1”后，size=1，cursor=1，此时hasNext()返回false，结束循环，因此你的迭代器并没有调用next查找第二个元素，也就无从检测modCount了，因此也不会出现多线程修改异常
     > 但当你删除“2”时，迭代器调用了两次next，此时size=1，cursor=2，hasNext()返回true，于是迭代器傻乎乎的就又去调用了一次next()，因此也引发了modCount不相等，抛出多线程修改的异常。
     >
     > 当你的集合有三个元素的时候，你就会神奇的发现，删除“1”是会抛出异常的，但删除“2”就没有问题了，究其原因，和上面的程序执行顺序是一致的。

    参考文章：[为什么java不要在foreach循环里进行元素的remove/add操作](https://segmentfault.com/q/1010000008858681)

7. 在 JDK 7 版本以上， Comparator 要满足自反性，传递性，对称性，不然 Arrays.sort ，Collections.sort 会报 IllegalArgumentException 异常。

     保证等于和大小与分开，要严格有序。

     Comparable 是排序接口。若一个类实现了Comparable接口，就意味着“**该类支持排序**”。

      Comparable 接口仅仅只包括一个函数：

     Comparator 是比较器接口。我们若需要控制某个类的次序，而该类本身不支持排序(即没有实现Comparable接口)，可以通过“**实现Comparator类来新建一个比较器**”，然后通过该比较器对类进行排序。

     Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。

      ```java
        package java.lang;
        import java.util.*;

        public interface Comparable<T> {
            public int compareTo(T o);
        }

        public interface Comparator<T> {
            int compare(T o1, T o2);
            boolean equals(Object obj);
        }
       ```



8. 集合初始化，指定集合的初始值大小。

    Collection的初始容量也显得异常重要。所以：对于已知的情景，请为集合指定初始容量。

     HashMap初始化容量计算 = （需要存储元素的个数 / 负载因子） + 1

    ```false
       // ArrayList新容量扩大到原容量的1.5倍，右移一位相关于原数值除以2。
       int newCapacity = oldCapacity + (oldCapacity >> 1);
       //Vector线程安全，速度慢。默认初始容量为10，加载因子为1：即当 元素个数 超过 容量长度 时，进行扩容扩容增量：原容量的1倍
       int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
       //HashMap
       static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
       static final float DEFAULT_LOAD_FACTOR = 0.75f;
       if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY) newThr = oldThr << 1;
       //HashTable
       int newCapacity = oldCapacity * 2 + 1;
       //StringBuilder
       void expandCapacity(int minimumCapacity) {  
               int newCapacity = value.length * 2 + 2;  
               if (newCapacity  minimumCapacity < 0)  
                   newCapacity = minimumCapacity;  
               if (newCapacity < 0) {  
                   if (minimumCapacity < 0) // overflow  
                       throw new OutOfMemoryError();  
                   newCapacity = Integer.MAX_VALUE;  
               }  
               value = Arrays.copyOf(value, newCapacity);  
           }  
     ```

| 集合名称                    | 默认容量 | 加载因子 | 扩容容量(扩大到原来的) |
| ----------------- | --------- | --------- | ----------- |
| ArrayList                  | 10   | 1    | 1.5          |
| Vector                         | 10   | 1    | 2            |
| HashSet                        | 16   | 0.75 | 2            |
| HashMap                        | 16   | 0.75 | 2            |
| HashTable                      | 11   | 0.75 | *2 +1        |
| StringBuilder(StringBuffer)| 16   | 条件判断 | *2+2      |

9. 遍历Map类，使用entrySet。

   同时遍历key和value时，keySet与entrySet方法的性能差异取决于key的具体情况，如复杂度（复杂对象）、离散度、冲突率等。换言之，取决于HashMap查找value的开销。entrySet一次性取出所有key和value的操作是有性能开销的，当这个损失小于HashMap查找value的开销时，entrySet的性能优势就会体现出来。

   同时遍历key和value时，与HashMap不同，entrySet的性能远远高于keySet。这是由TreeMap的查询效率决定的，也就是说，TreeMap查找value的开销较大，明显高于entrySet一次性取出所有key和value的开销。因此，遍历TreeMap时强烈推荐使用entrySet方法。

```java
//keySet遍历2次，一次转为Iterator对象，一次从HashMap中取出对应的value
for (String key : map.keySet()) {
    value = map.get(key);
}
//entrySet
for (Entry<String, String> entry: map.entrySet()) {
    key = entry.getKey();
    value = entry.getValue();
}
//for循环
for (String value : map.values()) {
}
//JDK8 增强for循环
Map<String, Integer> items = new HashMap<>();
items.forEach((k,v)>System.out.println("key : " + k + "; value : " + v));
```

 参考文章：[Java Map遍历方式的选择](http://www.cnblogs.com/fczjuever/archive/2013/04/07/3005997.html)
 
 10. Map类集合K/V为null需要注意的地方：

| 集合类               | Key       | Value     | Super       | 说明     |
| ----------------- | --------- | --------- | ----------- | ------ |
| Hashtable         | 不允许为 null | 不允许为 null | Dictionary  | 线程安全   |
| ConcurrentHashMap | 不允许为 null | 不允许为 null | AbstractMap | 分段所锁技术 |
| TreeMap           | 不允许为 null | 允许为 null  | AbstractMap | 线程不安全  |
| HashMap           | 允许为 null  | 允许为 null  | AbstractMap | 线程不安全  |

    由于 HashMap的干扰，很多人认为 ConcurrentHashMap是可以置入 null值，注意存储null值时会抛出 NPE异常。

11. 集合的有序性和稳定性：
    ArrayList是order/unsort，HashMap是unorder/unsort，TreeSet是order/sort

12. 利用Set唯一性去重，避免使用List的contains方法遍历去重。
    Collection的contains()和remove()操作都是线性时间复杂度，用set也会隐式的调用contains()方法，不过你用的是HashSet,这个contains()应该只会用常数时间，所以如果考虑平均时间复杂度，用set可能会占优；最坏情况下，两者可能差不多 。






### 并发处理

1. 获取单例对象，保证线程安全，以及方法的线程安全。

   ```java
   //立即加载（饿汉模式），在调用getInstance()方法前，实例就被创建了，getInstance()方法没有同步，所以可能出现非线程安全问题。
   public class Singleton {
      private final static Singleton INSTANCE = new Singleton();
      private Singleton() { }
      public static Singleton getInstance() {
         return INSTANCE;
      }
   }
   //延迟加载（懒汉模式），延迟加载就是在getInstance()方法中创建实例。在多线程的环境中，延迟加载中使用同步代码块，对类加锁。虽然做到了线程安全，并且解决了多实例的问题，但是它并不高效。因为在任何时候只能有一个线程调用 getInstance() 方法。
   public class Singleton {
       private static Singleton instance;
       private Singleton (){}
      
       public static synchronized Singleton getInstance() {
           if (instance == null) {
               instance = new Singleton();
           }
           return instance;
       }
   }


   //DCL双检查锁机制。 DCL双检查锁机制即使用volatile关键字（使变量在多个线程中可见）修改对象和synchronized代码块
   //两次检查 instance == null，一次是在同步块外，一次是在同步块内。为什么在同步块内还要再检验一次？因为可能会有多个线程一起进入同步块外的 if，如果在同步块内不进行二次检验的话就会生成多个实例了。
   //instance = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。
   //给 instance 分配内存
   //调用 Singleton 的构造函数来初始化成员变量
   //将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）
   //JVM 的即时编译器中存在指令重排序的优化，只需要将 instance 变量声明成 volatile 来避免指令重排序。
   public class Singleton {
       private static volatile Singleton singleton = null;
       private Singleton(){}
       public static Singleton getSingleton(){
           if(singleton == null){
               synchronized (Singleton.class){
                   if(singleton == null){
                       singleton = new Singleton();
                   }
               }
           }
           return singleton;
       }    
   }
   //【推荐】静态内部类
   //使用JVM本身机制保证了线程安全问题；由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候不会进行同步，没有性能缺陷；也不依赖 JDK 版本。
   //但是如果对象是序列化的就无法达到效果了。
   public class Singleton {  
       private static class SingletonHolder {  
           private static final Singleton INSTANCE = new Singleton();  
       }  
       private Singleton (){}  
       public static final Singleton getInstance() {  
           return SingletonHolder.INSTANCE; 
       }  
   }
   //枚举
   //枚举的缺点是它无法从另一个基类继承，因为它已经继承自java.lang.Enum。
   public enum FooEnumSingleton {
       INSTANCE;
       public static FooEnumSingleton getInstance() { return INSTANCE; }
       public void bar() { }
   }

   ```

2. 线程资源通过线城池提供，通过ThreadPoolExecutor方式创建。

    ThreadPoolExecutor作为java.util.concurrent包对外提供基础实现，以内部线程池的形式对外提供管理任务执行，线程调度，线程池管理等等服务

    Executors方法提供的线程服务，都是通过参数设置来实现不同的线程池机制。

    关系：Executors可以认为是封装好的线城池服务，ThreadPoolExecutor更加明确线程池的运行机制。

    `Executors.newCachedThreadPool();        ``//创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE，允许创建线程为Integer.MAX_VALUE,容易OOM`

     `Executors.newSingleThreadExecutor();   ``//创建容量为1的缓冲池，请求队列为长度为Integer.MAX_VALUE,容易OOM`，

     `Executors.newFixedThreadPool(``int``);    ``//创建固定容量大小的缓冲池请，求队列为长度为Integer.MAX_VALUE,容易OOM`

     ```java
     public ThreadPoolExecutor(int corePoolSize,  
                                   int maximumPoolSize,  
                                   long keepAliveTime,  
                                   TimeUnit unit,  
                                   BlockingQueue<Runnable> workQueue,  
                                   ThreadFactory threadFactory,  
                                   RejectedExecutionHandler handler) {  
             if (corePoolSize < 0 ||  
                 maximumPoolSize <= 0 ||  
                 maximumPoolSize < corePoolSize ||  
                 keepAliveTime < 0)  
                 throw new IllegalArgumentException();  
             if (workQueue == null || threadFactory == null || handler == null)  
                 throw new NullPointerException();  
             this.corePoolSize = corePoolSize;  
             this.maximumPoolSize = maximumPoolSize;  
             this.workQueue = workQueue;  
             this.keepAliveTime = unit.toNanos(keepAliveTime);  
             this.threadFactory = threadFactory;  
             this.handler = handler;  
         }  
     /*
     corePoolSize 核心线程池大小
     maximumPoolSize 线程池最大容量大小
     keepAliveTime 线程池空闲时，线程存活的时间
     TimeUnit 时间单位
     ThreadFactory 线程工厂
     BlockingQueue任务队列
     RejectedExecutionHandler 线程拒绝策略
     */
     ```


    ![](http://dl2.iteye.com/upload/attachment/0105/9641/92ad44092ab4388b9fb19fc4e0d832cd.jpg)


    ```java
     public class Test {
          public static void main(String[] args) {   
              ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 200, TimeUnit.MILLISECONDS,
                      new ArrayBlockingQueue<Runnable>(5));
      
              for(int i=0;i<15;i++){
                  MyTask myTask = new MyTask(i);
                  executor.execute(myTask);
                  System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
                  executor.getQueue().size()+"，已执行玩别的任务数目："+executor.getCompletedTaskCount());
              }
              executor.shutdown();
          }
     }
      
     class MyTask implements Runnable {
         private int taskNum;
      
         public MyTask(int num) {
             this.taskNum = num;
         }
      
         @Override
         public void run() {
             System.out.println("正在执行task "+taskNum);
             try {
                 Thread.currentThread().sleep(4000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println("task "+taskNum+"执行完毕");
         }
     }
     ```


    参考文章：[深入理解Java之线程池](http://www.importnew.com/19011.html)

3. SimpleDateFormat类线程不安全

    线程不安全原因：
    SimpleDateFormat(下面简称sdf)类内部有一个Calendar对象引用,它用来储存和这个sdf相关的日期信息,例如sdf.parse(dateStr), sdf.format(date) 。
    calendar这个共享变量的访问没有做到线程安全
    ![](http://static.oschina.net/uploads/space/2013/0813/033057_iid0_568818.jpg)


    解决方法：

    将SimpleDateFormat定义成局部变量：

    加一把线程同步锁：synchronized(lock)；

    【推荐】使用ThreadLocal: 每个线程都将拥有自己的SimpleDateFormat对象副本。

    ```java
     import java.text.ParseException;
     import java.text.SimpleDateFormat;
     import java.util.Date;
     import java.util.HashMap;
     import java.util.Map;

     public class DateUtil {
       /** 锁对象 */
       private static final Object lockObj = new Object();
       /** 存放不同的日期模板格式的sdf的Map */
       private static Map<String, ThreadLocal<SimpleDateFormat>> sdfMap = new HashMap<String, ThreadLocal<SimpleDateFormat>>();
       /**
        * 返回一个ThreadLocal的sdf,每个线程只会new一次sdf
        *
        * @param pattern
        * @return
        */
       private static SimpleDateFormat getSdf(final String pattern) {
         ThreadLocal<SimpleDateFormat> tl = sdfMap.get(pattern);
         // 此处的双重判断和同步是为了防止sdfMap这个单例被多次put重复的sdf
         if (tl == null) {
           synchronized (lockObj) {
             tl = sdfMap.get(pattern);
             if (tl == null) {
               // 只有Map中还没有这个pattern的sdf才会生成新的sdf并放入map
               System.out.println("put new sdf of pattern " + pattern + " to map");
               // 这里是关键,使用ThreadLocal<SimpleDateFormat>替代原来直接new SimpleDateFormat
               tl = new ThreadLocal<SimpleDateFormat>() {
                 @Override
                 protected SimpleDateFormat initialValue() {
                   System.out.println("thread: " + Thread.currentThread() + " init pattern: " + pattern);
                   return new SimpleDateFormat(pattern);
                 }
               };
               sdfMap.put(pattern, tl);
             }
           }
         }
         return tl.get();
       }
       /**
        * 是用ThreadLocal<SimpleDateFormat>来获取SimpleDateFormat,这样每个线程只会有一个SimpleDateFormat
        *
        * @param date
        * @param pattern
        * @return
        */
       public static String format(Date date, String pattern) {
         return getSdf(pattern).format(date);
       }
       public static Date parse(String dateStr, String pattern) throws ParseException {
         return getSdf(pattern).parse(dateStr);
       }
     }
     ```

4. 高并发的同步调用考虑锁的性能消耗，锁的粒度尽可能小。

    在获得锁之前做完所有需要做的事，只把锁用在需要同步的资源上，用完之后立即释放它。减少锁持有时间。
    减小锁粒度：ConcurrentHashMap。
    锁粗化：如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化 。
    锁消除：即时编译器时，如果发现不可能被共享的对象，则可以消除这些对象的锁操作。
    分别用不同的锁来保护同一个类中多个独立的状态变量，而不是对整个类域只使用一个锁。锁分离。ReadWriteLock。
    参考文章：[Java 高并发九：锁的优化和注意事项详解](http://www.jb51.net/article/92453.htm)

5. 对多个资源、库表、对象加锁，需要保持一致的加锁顺序。

    当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。
    此外还有加锁时限、死锁检测等方法预防死锁。

6. 并发修改同一记录必须加锁。

    乐观锁，大多是基于数据版本(Version)记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于[数据库](http://www.knowsky.com/sql.asp)表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来实现。 读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。 
    提交版本必须大于记录当前版本才能执行更新。
    悲观锁（Pessimistic Lock），正如其名，具有强烈的独占和排他特性。它指的是对数据被外界（包括本系统当前的其他[事务](http://baike.baidu.com/view/121511.htm)，以及来自外部系统的[事务处理](http://baike.baidu.com/subview/709594/709594.htm)）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的[排他性](http://baike.baidu.com/subview/118455/118455.htm)，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。

7. 多线程并行之Timer

    定时任务用Timer实现有可能出现异常，因为它是基于绝对时间而不是相对时间进行调度的。当环境的系统时间被修改后，原来的定时任务可能就不跑了。另外需要注意一点，捕获并处理定时任务的异常。如果在TimerTask里抛出了异常，那么Timer认为定时任务被取消并终止执行线程。

8. 异步转同步操作CountDownLatch

    CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。
    CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。
    ![](http://incdn1.b0.upaiyun.com/2015/04/f65cc83b7b4664916fad5d1398a36005.png)


    构造器中的**计数值（count）实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。这种通知机制是通过 **CountDownLatch.countDown()**方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。
    参考文章：[什么时候使用CountDownLatch](http://www.importnew.com/15731.html)

9. 多线程之Random

    任何情况下都不要在多个线程间共享一个*java.util.Random*实例，而该把它放入*ThreadLocal*之中。

    Java7在**所有**情形下都更推荐使用*java.util.concurrent.ThreadLocalRandom*——它向下兼容已有的代码且运营成本更低。

    ```java
     private static void testTL_Random( final int threads, final long cnt )
     {
         final CountDownLatch latch = new CountDownLatch( threads );
         final ThreadLocal<Random> rnd = new ThreadLocal<Random>() {
             @Override
             protected Random initialValue() {
                 return new Random( 100 );
             }
         };
         for ( int i = 0; i < threads; ++i )
         {
             final Thread thread = new Thread( new RandomTask( null, i, cnt, latch ) {
                 @Override
                 protected Random getRandom() {
                     return rnd.get();
                 }
             } );
             thread.start();
         }
     }
     ```

    参考文章：[多线程环境下生成随机数](http://www.importnew.com/12460.html)

10. volatile

    1. 可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
    2. 当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。
    3. 通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。
    4. **volatile不能确保原子性**
    5. 可以通过synchronized或lock，进行加锁，来保证操作的原子性。也可以通过AtomicInteger。
    6. java.util.concurrent.atomic包下提供了一些原子操作类，即对基本数据类型的 自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。atomic是利用CAS来实现原子性操作的（Compare And Swap），CAS实际上是利用处理器提供的CMPXCHG指令实现的，而处理器执行CMPXCHG指令是一个原子性操作。
    7. 好的一面是它通过一个直接机器码指令设置值时，能够最小程度地影响其他线程的执行。坏的一面是如果它在与其他线程竞争设置值时失败了，它不得不再次尝试。在高竞争下，这将转化为一个自旋锁，线程不得不持续尝试设置值，无限循环直到成功。
    8. JDK8中LongAdder实例，并使用intValue()和add()来获取和设置值。神奇的地方发生在幕后。这个类所做的事情是当一个直接CAS由于竞争失败时，它将delta保存在为该线程分配的一个内部单元对象中，然后当intValue()被调用时，它会将这些临时单元的值再相加到结果和中。这就减少了返回重新CAS或者阻塞其他线程的必要。
    9. 参考文章：
        [你真的了解volatile关键字吗？](http://www.importnew.com/24082.html)
        [Java 8 LongAdders：管理并发计数器的正确方式](http://www.importnew.com/11345.html)

11. HashMap在resize可能发生死链，加锁解决。

     当多个线程同时检测到总数量超过门限值的时候就会同时调用resize操作，各自生成新的数组并rehash后赋给该map底层的数组table，结果最终只有最后一个线程生成的新数组被赋给table变量，其他线程的均会丢失。而且当某些线程已经完成赋值而其他线程刚开始的时候，就会用已经被赋值的table作为原始数组，这样也会有问题。
     首先如果多个线程同时使用put方法添加元素，而且假设正好存在两个put的key发生了碰撞(hash值一样)，那么根据HashMap的实现，这两个key会添加到数组的同一个位置，这样最终就会发生其中一个线程的put的数据被覆盖。
     线程安全：
       Hashtable
       ConcurrentHashMap(性能优势)
       Synchronized Map
     参考文章：[如何线程安全的使用HashMap](http://www.importnew.com/21396.html)

12. ThreadLocal

    参考文章：[彻底理解ThreadLocal](http://www.cnblogs.com/xzwblog/p/7227509.html)




# 参考文章：



1. [【Java编码规范】《阿里巴巴Java开发手册（正式版）》更新（v1.2.0版）——迄今最完善版本](https://yq.aliyun.com/articles/69327?spm=5176.100241.0.0.3rSWCo#)
2. [白话阿里巴巴Java开发手册(编程规约)](http://www.jianshu.com/p/bc8fed863eca)
3. [白话阿里巴巴Java开发手册（异常日志）](http://www.jianshu.com/p/5b6d180bd1c2)
4. [白话阿里巴巴Java开发手册（安全规约）](http://www.jianshu.com/p/9528c4ea1504)