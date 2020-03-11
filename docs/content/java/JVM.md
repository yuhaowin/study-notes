+ 生产环境内存溢出如何处理？
+ 生产环境服务器该分配多少内存合适？
+ 如何对垃圾回收器的性能进行调优？性能指标有哪些？
+ 生产环境 cpu 负载飙高如何处理？
+ 生产环境该给应用分配多少线程合适？

#### JVM 参数类型

 + 标准参数 所有 JVM 实现均有 如：java -version java -server java -client

 + -X 参数 如：-Xint -Xcomp -Xmixed

 + -XX 参数非标准化参数

    + boolean 类型 表示启用/禁用参数 

      -XX:[+]UseG1GC 启用 G1GC

      -XX:[-]UseCompressedClassPointers 禁用压缩类空间
      
    + key - value 类型
    
      -XX:GCTimeRatio=19
    
      -Xms -Xmx -Xss 都是 -XX 参数
    
      -Xms = -XX:InitialHeapSize 最小内存初始化的堆的大小
    
      -Xmx = -XX:MaxHeapSize 最大内存
    
      -Xss = -XX:ThreadStackSize 一个线程堆栈的大小
    
      
      
      ![144949](http://image.yuhaowin.com/2020/03/11/144949.jpg)

![145047](http://image.yuhaowin.com/2020/03/11/145047.jpg)

![145121](http://image.yuhaowin.com/2020/03/11/145121.jpg)

![LGTD2HmZkc7K3UB](https://i.loli.net/2020/03/11/LGTD2HmZkc7K3UB.jpg)

![LKc5Rbr9H31xQXM](https://i.loli.net/2020/03/11/LKc5Rbr9H31xQXM.jpg)

![N5z2hIu1XawgQx8](https://i.loli.net/2020/03/11/N5z2hIu1XawgQx8.jpg)

![XkOQENGMCUWTm7f](https://i.loli.net/2020/03/11/XkOQENGMCUWTm7f.jpg)

![fBdRT5tKIglMixe](https://i.loli.net/2020/03/11/fBdRT5tKIglMixe.jpg)

![vuyi8exXbtWwLjE](https://i.loli.net/2020/03/11/vuyi8exXbtWwLjE.jpg)

#### 查看 JVM 运行是的参数

-XX:+PrintFlagsFinal

-XX:+PrintFlagsInitial

结果中 = 表示默认值 := 表示修改后的值

![010251](http://image.yuhaowin.com/2020/03/08/010251.jpg)



jps 类似 Linux 的 ps 只查看 java 进程



jinfo 

jinfo -flag 参数名 pid

![010821](http://image.yuhaowin.com/2020/03/08/010821.jpg)



JVM 64bit 没有 client 模式 只有 server 模式



Java 虚拟机是一个抽象的机器，是一个虚拟机实现的规范，有很多根据规范实现的具体的虚拟机，如 Oracle 的 Hotspot。

JVM规范是一种高度抽象行为的描述，而不是具体虚拟机的实现。

Animorphic Smalltalk 虚拟机 -> Hotspot 虚拟机

Java 虚拟机是Java语言实现硬件无关、操作系统无关的关键部分。

Java 虚拟机和java 语言没有必然的联系，它只是与特定的二进制文件class文件有关联。

class 文件是一种能够被java 虚拟机所执行的二进制文件，常常以文件的形式存储。

jvm 的数据类型也分为两种：基本类型、引用类型。

Java 虚拟机希望尽可能多的类型检查能在程序运行之前完成，换句话说，编译器应当在编译
期间尽最大努力完成可能的类型检查，使得虚拟机在运行期间无需进行这些操作。

>Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁。



![195922](http://image.yuhaowin.com/2020/03/10/195922.jpg)

虚拟机定义的运行时数据区，就是我们常说的内存结构。

运行时数据区中的有些数据是一直存在的，被所有的线程共享，而有些数据是线程私有的，随着线程开始而创建，结束而销毁。

+ 公有部分包括：方法区、运行时常量池、Java堆。
+ 私有部分包括：Java虚拟机栈、本地方法栈、PC寄存器。

##### 方法区 （线程共享）**元空间**，还有个别名 non-heap（非堆）

在Java虚拟机中，方法区(Method Area)是可供各条线程共享的运行时内存区域。它存储了每一个类的结构信息，例如运行时常量 池(Runtime Constant Pool)、字段和方法数据、构造函数和普通方法的字节码内容、还包 括一些在类、实例、接口初始化时用到的特殊方。

在HotSpot虚拟机中，JDK1.7版本称其为永久代（Permanent Generation），而在JDK1.8则称之为元空间（Metaspace）。

可能有异常

+ OutOfMemoryError

如果方法区的内存空间不能满足内存分配请求，那Java虚拟机将抛出一个OutOfMemoryError 异常。

##### 运行时常量池 （分配在方法区内）

运行时常量池(Runtime Constant Pool)是每一个类或接口的常量池(Constant_Pool)的运行时表示形式，每一个运行时常量池都分配在 Java 虚拟机的方法区之中，在类和接口被加载到 虚拟机后，对应的运行时常量池就被创建出来。

可能有异常

+ OutOfMemoryError

当创建类或接口的时候，如果构造运行时常量池所需要的内存空间超过了方法区所能提供的最大值，那 Java 虚拟机将会抛出一个 OutOfMemoryError 异常。

##### JAVA 堆 （线程共享）

在 Java 虚拟机中，堆(Heap)是可供各条线程共享的运行时内存区域，也是供所有类实例和数组对象分配内存的区域。

可能有异常

+ OutOfMemoryError

如果实际所需的堆超过了能提供的最大容量，那Java虚拟机将会抛出一个OutOfMemoryError 异常。

##### JAVA 虚拟机栈 （线程私有）

每一个虚拟机线程都有自己私有的虚拟机栈，这个虚拟机栈和线程是同时创建的，用于存储栈帧，就是用于存储局部变量与一些过程结果的地方。

可能有两个异常

+ StackOverFlowError
+ OutOfMemoryError

Java8 中默认一个虚拟机栈的空间大小是 1MB 如果存放的栈帧太多导致1MB内存耗尽会 StackOverFlowError

如果在一开始申请分配 1MB 大小空间时内存不够，会OutOfMemoryError。

##### 本地方法栈（线程私有）

如果 Java 虚拟机不支持 natvie 方法，并且自己也不依赖传统栈的话，可以无需支持 地方法栈，如果支持本地方法栈，那这个栈一般会在线程创建的时候按线程分配。

可能有两个异常

+ StackOverFlowError
+ OutOfMemoryError

##### 程序计数器  PC 寄存器 （线程私有）

>Java 虚拟机可以支持多条线程同时执行(可参考《Java 语言规范》第 17 章)，每一条 Java 虚拟机线程都有自己的PC(Program Counter)寄存器。在任意时刻，一条Java虚拟机线程 只会执行一个方法的代码，这个正在被线程执行的方法称为该线程的当前方法(Current Method)。如果这个方法不是 native 的，那 PC 寄存器就保存 Java 虚拟机正在执行的 字节码指令的地址，如果该方法是 native 的，那 PC 寄存器的值是 undefined。



|     名称      |                           特征                           |                             作用                             |    配置参数     |     可能异常     |
| :-----------: | :------------------------------------------------------: | :----------------------------------------------------------: | :-------------: | :--------------: |
|    方法区     | 线程共享，生命周期与虚拟机相同，可以不使用连续的内存地址 | 存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据 | XX:PermSize=16M | OutOfMemoryError |
| 运行时常量池  |                方法区的一部分，具有动态性                |                     存放字面量及符号引用                     |        -        | OutOfMemoryError |
|    JAVA 堆    | 线程共享，生命周期与虚拟机相同，可以不使用连续的内存地址 |     保存对象实例，所有对象实例（包括数组）都要在堆上分配     | -Xms -Xsx -Xmn  | OutOfMemoryError |
| JAVA 虚拟机栈 |     线程私有，生命周期与线程相同，使用连续的内存空间     | Java 方法执行的内存模型，存储局部变量表、操作栈、动态链接、方法出口等信息 |      -Xss       |    OOME、SOE     |
|  本地方法栈   |                                                          |                                                              |                 |    OOME、SOE     |
|  程序计数器   |               线程私有, 生命周期与线程相同               |                    大致为字节码行号指示器                    |        -        |        -         |

[JVM 内存结构&内存模型](https://juejin.im/post/5d7d9204f265da03f33382dd)

[Oracle JVM 规范](https://docs.oracle.com/javase/specs/index.html)

[Oracle JAVA 文档](https://docs.oracle.com/en/java/index.html)

[博客园 JVM 规范解读](https://www.cnblogs.com/chanshuyi/p/jvm_specification_00_guide.html)

[JVM 内存结构快问快答](https://juejin.im/post/5e51f2eae51d4526e03f9da8)



https://heaphero.io/

https://gceasy.io/

https://fastthread.io/



https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html



Hotspot UseMaximumCompactionOnSystemGC

Openjdk UseParallelGC

https://blog.csdn.net/wd2014610/article/details/81664062



https://www.jianshu.com/p/f55ddf1e9839