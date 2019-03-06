#### Java 集合

> 先按直接前继和后继数对数据结构分为4大类：线性结构、树、图、hash<br>
> 线性结构：有明确的首尾元素，除首尾元素外，每个元素都有一个前继和后继。<br>
> hash：没有明确的前继和后继，是通过特定的哈希算法将索引和value关联起来。


java的集合是用来存放对象的容器,集合主要分为两类：

+ Collection (存放单元数据）
+ Map （存放K-V）


##### List
> List 是线性结构的实现

+ ArrayList

ArrayList 内部是通过数组实现的，是非线程安全的集合，通过数组的扩容，来更改（增加）ArrayList的容量，正是通过创建一个更大容量的新数组，再将原有的数据复制到新数组中，这里存在线程安全问题。

基于数组实现的必然导致查询很快，而插入和删除很慢。

ArrayList 默认容量为10个，在第一次add时会进行初始化为10

假设使用默认构造器 会给内部存储数据的elementData赋值一个空的obj的数组， 在add时会检查数组的容量是否能够存下当前的元素，在计算容量时如果是elementData是空数组，则需要保证容量至少为默认的10，显然当前的容量为0，需要进行扩容，扩容为当前的1.5倍，但是（0*1.5=0）即计算后需要扩容的值小于需要至少保证的值，则直接扩容至少保证的值10；

适当指定初始容量以避免过多的被动扩容和数组复制带来的额外开销。

ArrayList 线程不安全理解：
在add操作是可能出现：

+ 少值和null值
+ 数组下标越界

这是由于赋值时出现了覆盖。赋值语句为：elementData[size++] = e，这条语句可拆分为两条：
1. elementData[size] = e；
2. size ++；
假设A线程执行完第一条语句时，CPU暂停执行A线程转而去执行B线程，此时ArrayList的size并没有加一，这时在ArrayList中B线程就会覆盖掉A线程赋的值，而此时，A线程和B线程先后执行size++，便会出现值为null的情况。

至于出现的ArrayIndexOutOfBoundsException异常，
当list大小为9，即size=9
线程A开始进入add方法，这时它获取到size的值为9，调用ensureCapacityInternal方法进行容量判断。
线程B此时也进入add方法，它获取到size的值也为9，也开始调用ensureCapacityInternal方法。
线程A发现需求大小为10，而elementData的大小就为10，可以容纳。于是它不再扩容，返回。
线程B也发现需求大小为10，也可以容纳，返回。
线程A开始进行设置值操作， elementData[size++] = e 操作。此时size变为10。
线程B也开始进行设置值操作，它尝试设置elementData[10] = e，而elementData没有进行过扩容，它的下标最大为9。
因此便出现了ArrayIndexOutOfBoundsException异常。

既然ArrayList是线程不安全的，但如果需要在多线程中使用，可以采用list<Object> list =Collections.synchronizedList(new ArrayList<Object>)来创建一个ArrayList对象。

和 CopyOnWriteArrayList

两个的异同：

Arrays.copyOf返回的是原始对象、还是新对象？新对象



