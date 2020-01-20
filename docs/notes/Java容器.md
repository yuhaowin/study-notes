?> 对List中的元素进行更优雅的操作。

对元素某个属性进行分组

假设对用户的年龄进行分组

```java
	public class User {
	    private String name;
	    private Integer age;
	}
```

使用命令式编码：

```java
   Map<Integer, List<User>> result = new HashMap<>();
    for (User u : users) {
        Integer age = u.getAge();
        List<User> temp = result.get(age);
        if (temp == null) {
            temp = new ArrayList<>();
            result.put(age, temp);
        }
        temp.add(u);
    }
```

使用jdk8

```java
Map<Integer, List<User>> result = users.stream()
					.collect(groupingBy(User::getAge));
```

同时可以计算除每个分组的个数，只需传递一个计数收集器给 groupingBy 收集器。第二个收集器的作用是在流分类的同一个组中对每个元素进行递归操作

```java
Map<Integer, Long> result = users.stream()
					.collect(groupingBy(User:: getAge, counting()));
```


partitioningBy收集器对list进行分区，如：把年龄分为大于18和小于等于18的

```java
Map<Boolean, List<User>> collect2 = users.stream()
			.collect(partitioningBy(p -> p.getAge() > 18));
```


+ [参考资料](http://www.importnew.com/17313.html)

+ [参考资料2](http://www.importnew.com/17313.html)

*******


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

********
********

### 泛型

> 引入泛型的原因：主要是对一个集合内元素类型进行约束，以达到减少使用者的责任的目的，引入泛型可以借助编译器在编译时检查元素类型，防止程序在运行时出现 ClassCastException

```java
    @Test
    public void test3() {
        List list = new ArrayList();
        list.add("yuhao");
        list.add("Eric");
        list.add(new Integer(2));

        // 使用时需要进行强制转换
        String str1 = (String) list.get(0);
        String str2 = (String) list.get(1);
        String str3 = (String) list.get(2);
    }

    @Test
    public void test4() {
        List<String> list = new ArrayList<>();
        list.add("yuhao");
        list.add("Eric");

        // 编译器报错
        //list.add(new Integer(2));

        // 使用是无需做强制转换
        String str1 = list.get(0);
        String str2 = list.get(1);
    }
```

泛型的实现原理是：在编译后擦出具体传入的类型，相当于传入的是obj类型，然后在使用的地方编译器根据传入的类型进行强制转换，此时，进行强制转换不会报 ClassCastException，因为在添加的时候只能传入一种类型。

下面是test4根据 class 文件反编译的结果,可以看出编译器进行了强制转换。

```java
    public void test4()
    {
        ArrayList arraylist = new ArrayList();
        arraylist.add("yuhao");
        arraylist.add("Eric");
        String s = (String)arraylist.get(0);
        String s1 = (String)arraylist.get(1);
    }
```

一个更深刻的例子：

```java
public class GenericsTest<T> {

    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static <T, Eric, A, String> T get(T t, Eric eric, String string) {
        return t;
    }

    public static void main(String[] args) {

        GenericsTest<String> test1 = new GenericsTest<>();
        test1.setData("This is generics demo");

        GenericsTest<Long> test2 = new GenericsTest<>();
        test2.setData(2019L);

        Integer result = get(12, 34, "yuhao");
    }
}
```

反编译后的结果：

```java
public class GenericsTest
{
    public GenericsTest()
    {
    }

    private Object data;

    public Object getData()
    {
        return data;
    }

    public void setData(Object obj)
    {
        data = obj;
    }

    public static Object get(Object obj, Object obj1, Object obj2)
    {
        return obj;
    }

    public static void main(String args[])
    {
        GenericsTest genericstest = new GenericsTest();
        genericstest.setData("This is generics demo");
        GenericsTest genericstest1 = new GenericsTest();
        genericstest1.setData(Long.valueOf(2019L));
        Integer integer = (Integer)get(Integer.valueOf(12), Integer.valueOf(34), "yuhao");
    }
}
```

在这个例子中，类后面的 T 指代的是成员变量 data 的类型，在实例化该类时传入的 T 是什么类型，那么成员变量 data 就是什么类型。在这个类中还有个静态方法 get() 参数表后面的 T 和类名后面的 T 指代的并不相同。泛型 <T, Eric, A, String> 中的 Eric String 和 T A 一样，也只是类型的指代，尤其 String 不要被其外形迷惑，实际上在调用 get() 方法传入的 string 是什么类型，String 就代表什么类型。

>定义的泛型类，就一定要传入泛型类型实参么？并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

意思是我定义了一个泛型类,但是这个泛型不是必须使用的。

```java
@Data
public class Result<T> {

    private int code;

    private String message;

    private T data;
}
```


```java
    @Test
    public void test() {
        // 语法上没有错误，但是虽然定义的是泛型类，但是在使用时没有指定泛型
        // 等于就是object 没有发挥泛型的作用
        Result result1 = new Result();
        result1.setData("Eric");
        String data = (String) result1.getData();
        Assert.assertEquals("Eric", data);

        // 限制类类型只能是 String 类型，因此在取出来时可以确保一定是String 类型
        // 无需进行类型转换
        Result<String> stringResult = new Result<>();
        stringResult.setData("yuhao");
        String data1 = stringResult.getData();
        Assert.assertEquals("yuhao", data1);
    }
```

#### 限定泛型的上下界 --- 上界

假设定义一个方法展示:

```java
    public static void showData(Result<Number> result){
        System.out.println(result.getData());
    }
```

此时如果传入 Result<Integer> 的实例会报错，虽然 Integer 是 Number 的子类，但是Result<Integer> 不是 Result<Number> 的子类.

可以更改为：

```java
    public static void showData(Result<?> result){
        System.out.println(result.getData());
    }
```

虽然这样 Result<Integer> 和 Result<Number> 都可以传入，但是其他类型也可以传入如：Result<Person> 显然约束不够。

可以进一步限制：

```java
    public static void showData(Result<? extent Number> result){
        System.out.println(result.getData());
    }
```

这样就只能传入参数是 Number 和 它子类的类型了。



#### 泛型方法

> public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。


```java
    public  <T> T test4(Result<T> result){
        return result.getData();
    }
```


```java
    public  <T extends Number> T test5(Result<T> result){
        return result.getData();
    }
```
