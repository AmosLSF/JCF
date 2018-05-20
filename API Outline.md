# Outline of the Collections Framework 学习笔记

### [Annotated API Outline](https://docs.oracle.com/javase/9/docs/api/java/util/doc-files/coll-reference.html) - An annotated outline of the classes and interfaces comprising the collections framework, with links into the API Specification.

[参考翻译](https://blog.csdn.net/qq378532177/article/details/50542482)

## 补充：

### 1. [TransferQueue](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/TransferQueue.html)

中文参考：[TransferQueue](http://ifeve.com/java-transfer-queue/)

特点：在 BlockingQueue 基础上进一步提供了支持阻塞的添加元素接口，其阻塞持续到添加的元素被消费 (take/poll)。



### 2. [Collections.checkedInterface](https://docs.oracle.com/javase/9/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-) 

特点：生成支持动态类型检查的 collection 视图，当插入 object 类型与该视图的声明类型不匹配时，会马上扔出 ClassCastException

使用场景：对于可能绕过静态类型检查的集合使用，在插入式时就抛出ClassCastException
- 防止第三方接口/库函数时可能在无察觉的情况下插入一些静态类型检查无法发现的错误类型对象
```java
//reference https://blog.csdn.net/chang_harry/article/details/8759757

//: generics/CheckedList.java
   // Using Collection.checkedList().
   import typeinfo.pets.*;
   import java.util.*;

   public class CheckedList {
     @SuppressWarnings("unchecked")
     static void oldStyleMethod(List probablyDogs) {
       probablyDogs.add(new Cat());
     } 
     public static void main(String[] args) {
       List<Dog> dogs1 = new ArrayList<Dog>();
       oldStyleMethod(dogs1); // Quietly accepts a Cat
       List<Dog> dogs2 = Collections.checkedList(
         new ArrayList<Dog>(), Dog.class);
       try {
         oldStyleMethod(dogs2); // Throws an exception
       } catch(Exception e) {
         System.out.println(e);
       }
       // Derived types work fine:
       List<Pet> pets = Collections.checkedList(
         new ArrayList<Pet>(), Pet.class);
       pets.add(new Dog());
       pets.add(new Cat());
     }
   }
```

- 帮助已经出现 ClassCastException 的程序进行debug，将异常捕获时机提前到插入时刻，修复后再撤回该动态视图

### 3. [WeakHashMap](https://docs.oracle.com/javase/9/docs/api/java/util/WeakHashMap.html)

中文参考：[设计及实现机制](https://blog.csdn.net/u012129558/article/details/51980883)

特点：map 的 key 为 WeakReference，gc 后回收，回收 key 的 value 通过内部 referenceQueue 在后续每次访问时进行 join并删除以释放 对 value 的StrongRefence

使用场景：利用 gc 回收机制进行缓存控制的场景。

范例：

- ThreadLocal
- [Tomacat ConcurrentCache](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/collections/ConcurrentCache.java)


### 4. [IdentityHashMap](https://docs.oracle.com/javase/9/docs/api/java/util/IdentityHashMap.html)


特点：和通常使用 object.equals(objectTarget) 的 map 相比，IdentityHashMap 使用 object==objectTarget 来进行 key 的比较，附加优势就是更快的性能表现

### 5. [CopyOnWriteArrayList](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/CopyOnWriteArrayList.html)

特点：修改操作(remove, put)等是通过对内部存储先拷贝副本，后操作更新的方式完成。其 iterator 可见的是 该iterator 创建时刻的内容，因此是多线程安全的，但是其 iterator 不支持集合变更操作。多线程下的遍历性能高，但是修改操作成本很高。

使用场景：多线程安全下遍历频次远远高于集合变更操作


### 6. [DelayQueue](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/DelayQueue.html)

中文参考：[DelayQueue的使用场景](https://blog.csdn.net/u011001723/article/details/51882887)

特点：用于容纳实现了 Delayed 接口的对象，只有当前队列存在已经到期的对象时，poll() 才能拿到已经距离超时时间最远的对象

使用场景：需要延迟消费对象，比如超过30分钟的订单取消，比如订单确定短信延迟发送

