---
title: 【JAVA】面试总结——Java基础篇
date: 2020-08-10 17:37:15
tags:
  - Java
  - 面试
top: true
cover: true
img: /images/timg.jpg
categories: Java面试
---
#### 本文目的
总结面试常见Java基础面试题  

#### Java基础

##### 1、封装、继承、多态

- 封装

  封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，那我们大可不必提供方法给外界访问。如果一个类没有提供给外界访问的方法，那么这个类也没什么意义了。

- 继承

  继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码。

  - 子类拥有父类对象所有的属性和方法(包括私有属性和私有方法)，但是父类中的私有属性和方法子类是无法访问，只是拥有。
  - 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。 
  - 子类可以用自己的方式实现父类的方法。

- 多态

  - 所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并 不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出 的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

  - 在 Java 中有两种形式可以实现多态:继承(多个子类对同一方法的重写)和接口(实现接口并覆盖接口中同一方法)。

##### 2、Java自动装箱与拆箱

- 装箱：自动将基本数据类型转换为包装器类型（int -> Integer) 
- 拆箱：自动将包装器类型转换为基本数据类型（Integer -> int）

##### 3、Java的四种引用

- 强引用

  强引用在OOM时也不会被回收。

  ```java
  String str = new String("");
  ```

- 软引用

  软引用在OOM时会被回收。

  适用场景：实现缓存技术时。

  ```java
  􏰍􏰇􏱕// 注意：wrf这个引用也是强引用。
  // 软引用指的是指向new String("str")的引用，也就是SoftReference类中T
  SoftReference<String> wrf = new SoftReference<String>(new String("str"));
  ```

- 弱引用

  弱引用无论是否OOM，只要JVM开始进行垃圾回收，就会回收。

  ```java
  WeakReference<String> srf = new WeakReference<>(str);
  ```

- 虚引用

  虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收。

  ```java
  ReferenceQueue<String> queue = new ReferenceQueue<String>();  
          PhantomReference<String> pr = new PhantomReference<String>(new String("str"), queue);  
  ```

##### 4、Java对象创建方式

- new对象
- 反射创建
- 采用clone
- 序列化

##### 5、hashcode相关

hashcode：根据对象的内存地址换算出来的一个值。

- 直接定址法

  - 取关键字或关键词的某个线性函数值为散列表地址。

  - f(key) = key 或 f(key) = a*key + b，其中a和b为常数。

- 除留余数法

  - 取关键字被某个不大于散列表长度 m 的数 p 求余，得到的作为散列地址。

  - f(key) = key % p, p < m。这是最为常见的一种哈希算法。

- 数字分析法

  - 当关键字的位数大于地址的位数，对关键字的各位分布进行分析，选出分布均匀的任意几位作为散列地址。

  - 仅适用于所有关键字都已知的情况下，根据实际应用确定要选取的部分，尽量避免发生冲突。

- 平方取中法

  - 先计算出关键字值的平方，然后取平方值中间几位作为散列地址。

  - 随机分布的关键字，得到的散列地址也是随机分布的。

- 随机数法

  - 选择一个随机函数，把关键字的随机函数值作为它的哈希值。

  - 通常当关键字的长度不等时用这种方法。

在hash冲突的时候，有可能两个不相等的对象有相同的hashcode值，当产生hash冲突时，[一般采用](https://www.cnblogs.com/tag6254/p/9416946.html "Hash冲突解决方案")：

- 拉链法

  每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表继续存储。

- 开放定址法

  一旦发生冲突，就去寻找下一个空的散列表地址，只要散列表足够大，空的散列表地址总能找到并记录。

- 再哈希

  又叫双哈希法，有多个不同的Hash函数，当发生冲突时，使用第二个，第三个...等哈希函数计算地址，直到无冲突。

##### 6、重写和重载

1. 重写（Override）
   - 不同类之间，子类重写父类的方法。
   - 方法名、参数列表、返回值类型必须相同
   - 访问权限修饰符一定要大于被重写的访问权限修饰符
2. 重载（Overload）
   - 在同一个类中，同名不同参数列表。
   - 它是一个类的多态性的一种表现
   - 返回值类型可以相同也可以不相同

> 构造器Constructor不能被override（重写），但是可以overload（重载），所以可以看到一个类中有多个构造函数的情况。

##### 7、深拷贝和浅拷贝

- 浅拷贝

  能复制变量，如果对象内还有对象，则只能复制对象的地址

- 深拷贝

  能复制变量，也能复制当前对象的 内部对象

##### 8、final相关

- 被final修饰的类不可以被继承
- 被final修饰的方法不可以被重写
- 被final修饰的变量不可以被改变，如果修饰引用，那么表示引用不可变，引用指向的内容可变
- 被final修饰的方法，JVM会尝试将其内联，以提高运行效率
- 被final修饰的常量，在编译阶段会存入常量池中。
- 编译器对final域要遵循两个重排规则
  - 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序
  - 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序

##### 9、接口和抽象类的区别

- 接口的方法默认是public，所有的方法在接口中不能有实现（Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
- 接口中除了static、final变量，不能有其他变量，而抽象类中则不一定。
- 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。
- 接口方法默认修饰符public，抽象方法可以有public、protected和default这些修饰符。（抽象方法就是为了被重写所以不能使用private关键字修饰）
- 从设计层面来说，抽象是对类的抽象，是一种模版设计，而接口上对行为的抽象，是一种行为的规范。

> 1. 在jdk 7 或更早的版本中，接口里面只能有常量、变量和抽象方法。这些接口方法必须由选择实现接口的类实现。
> 2. jdk 8 的时候接口可以有默认方法和静态方法功能。
> 3. jdk 9 在接口中引入了私有方法和私有静态方法。

##### 10、==和equals()

- == 对于基本类型来说是值比较，对于引用类型来说是比较的是引用（内存地址）
- equals() 默认情况下是引用比较，只是很多类重写了equals方法，比如String、Integer等把它变成了值比较。

##### 11、重写equals必须重写hashcode。

重写了equeals，而没有重写hashcode的话，就会不能保证相等对象必须拥有相等的hashcode。则该类无法结合所有的散列集合（HashMap，HashSet）一起正常运作。

##### 12、BIO、NIO、AIO

- **BIO (Blocking I/O):** 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高(小于单机1000)的情况下，这种模型是比􏱁不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚 至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

- **NIO (Non-blocking/New I/O):** NIO 是一种同步非阻塞的 I/O 模型，在 Java 1.4 中引入了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可 以理解为 Non-blocking，不单纯是 New。它支持面向缓冲的，基于通道的 I/O 操作方法。 NIO 提供了与传统 BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模 式。阻塞模式使用就像传统中的支持一样，比􏱁简单，但是性能和可靠性都不好;非阻塞模式正 好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞 I/O 来提升开发速率和更好 的维护性;对于高负载、高并发的(网络)应用，应使用 NIO 的非阻塞模式来开发

- **AIO (Asynchronous I/O):** AIO 也就是 NIO 2。在 Java 7 中引入了NIO的改进版 NIO 2,它是异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步 IO 的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO 操作本身是同步的。