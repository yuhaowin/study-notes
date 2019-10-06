#### String 类

+ String 对象的不可变性

对象不可变的原因有两个：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
String 类被 final 关键词修饰，所以 String 类禁止被继承，意味着其所有的方法不能被重写。

用来存储 String 对象的值的属性是 char 数组，该属性也是被 final 修饰，表明该属性一旦被赋值，其内存地址就无法修改了。加上value[]是private 变量。外部无法直接访问。

**被final修饰的变量的特性是：当该变量指向一个内存地址后，其内存地址将无法修改，但是对引用类型而言其值是可以修改的，由于String类是不可变的，不同的值就必然是不同的内存地址，因此被final修饰的String变量表现为值不可以再被修改**

查看对象的内存地址：`System.identityHashCode(object)`

+ 判断两个字符串字面量是否相等

String 类 equals 方法实现

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

先比较两个对象的内存地址是否一致，如果一致，则直接返回相等，否则依次比较两个对象的字符是否相等。

##### Long 类

Long 最被我们关注的就是 Long 的缓存问题，Long 自己实现了一种缓存机制，缓存了从 -128 到 127 内的所有 Long 值，如果是这个范围内的 Long 值，就不会初始化，而是从缓存中拿，缓存初始化源码如下：

Long 类中有一个 `LongCache ` 静态内部类，缓存范围为 -128 到 127

```java
    private static class LongCache {
        private LongCache(){}
        static final Long cache[] = new Long[-(-128) + 127 + 1];
        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
```


##### Volatile 关键词

volatile 的意思是反复无常的,常用来修饰某个共享变量,意思是当共享变量的值被修改后,会及时通知到其它线程上,其它线程就能知道当前共享变量的值已经被修改了。

在多核 CPU 下,为了提高效率,线程在拿值时,是直接和 CPU 缓存打交道的,而不是内存 主要是因为 CPU 缓存执行速度更快,比如线程要拿值 C,会直接从 CPU 缓存中拿, CPU 缓存中没有,就会从内存中拿,所以线程读的操作永远都是拿 CPU 缓存的值 这时候会产生一个问题,CPU 缓存中的值和内存中的值可能并不是时刻都同步,导致线程计算的值可能不是最新的,共享变量的值有可能已经被其它线程所修改了,但此时修改是机器内存的值,CPU 缓存的值还是老的,导致计算会出现问题 这时候有个机制,就是内存会主动通知 CPU 缓存 当前共享变量的值已经失效了,你需要重新来拉取一份,CPU 缓存就会重新从内存中拿取一份最新的值 

volatile 关键字就会触发这种机制,加了 volatile 关键字的变量,就会被识别成共享变量,内存中值被修改后,会通知到各个 CPU 缓存,使 CPU 缓存中的值也对应被修改,从而保证线程从 CPU 缓存中拿取出来的值是最新的。


![volatile 关键字](https://tva1.sinaimg.cn/large/006y8mN6gy1g7olqh42xoj30ze0kg0uh.jpg)


