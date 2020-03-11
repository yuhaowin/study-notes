### 为什么要引入 lambda 表达式



#### 函数式编程的演化历程

+ 将可变的条件直接固化在调用的方法内部 （X）
+ 将可变的条件作为参数传入方法中，在方法内部根据参数进行业务逻辑的实现
+ 将业务逻辑封装为一个实体类，方法接受实体类作为参数，在方法内部调用实体类处理业务逻辑
+ 调用方法时不在传入创建的实体类，而是传入匿名类。
+ 是 Lambda 表达式代替匿名类，从而实现业务逻辑的参数化传递。



```java
/**
 * Sku选择谓词接口
 */
public interface SkuPredicate {
    /**
     * 选择判断标准
     * @param sku
     * @return
     */
    boolean test(Sku sku);
}

/**
 * 对Sku的总价是否超出2000作为判断标准
 */
public class SkuTotalPricePredicate implements SkuPredicate {
    @Override
    public boolean test(Sku sku) {
        return sku.getTotalPrice() > 2000;
    }
}
```



```java
    /**
     * Version 4.0.0
     * 根据不同的Sku判断标准，对Sku列表进行过滤
     *
     * @param cartSkuList
     * @param predicate   - 不同的Sku判断标准策略
     * @return
     */
    public static List<Sku> filterSkus(
            List<Sku> cartSkuList, SkuPredicate predicate) {

        List<Sku> result = new ArrayList<Sku>();
        for (Sku sku : cartSkuList) {
            // 根据不同的Sku判断标准策略，对Sku进行判断
            if (predicate.test(sku)) {
                result.add(sku);
            }
        }
        return result;
    }
```



```java
    
    @Test
    public void filterSkus2() {
        List<Sku> cartSkuList = CartService.getCartSkuList();

        // 过滤商品总价大于2000的商品
        List<Sku> result = CartService.filterSkus(
                cartSkuList, new SkuTotalPricePredicate());

        System.out.println(JSON.toJSONString(
                result, true));
    }

		@Test
    public void filterSkus3() {
        List<Sku> cartSkuList = CartService.getCartSkuList();

        // 过滤商品单价大于1000的商品
        List<Sku> result = CartService.filterSkus(
                cartSkuList, new SkuPredicate() {
                    @Override
                    public boolean test(Sku sku) {
                        return sku.getSkuPrice() > 1000;
                    }
                });

        System.out.println(JSON.toJSONString(
                result, true));
    }

    @Test
    public void filterSkus4() {
        List<Sku> cartSkuList = CartService.getCartSkuList();

        // 过滤商品单价大于1000的商品
        List<Sku> result = CartService.filterSkus(
                cartSkuList,
                (Sku sku) -> sku.getSkuPrice() > 1000);

        System.out.println(JSON.toJSONString(
                result, true));
    }
```



利用设计模式中的策略模式，将行为的具体实现通过参数的形式传递到方法中。



### Lambda 简单介绍

#### Java 8 引入函数式编程的风格

> 函数式编程是一种编程的范式、一种编程的风格和思想。在Java8中函数也是可以作为变量、参数、返回值进行传递的。Lambda 表达式可以代替面向对象风格下的匿名函数, 以前是基于类或匿名类对行为进行传递的，现在是基于函数的， 直接将方法作为入参进行传递。

#### Lambda 表达式的表现形式

+ 参数 + 箭头 + 表达式  (parameters) -> expresson
+ 参数 + 箭头 + 大括号  (parameters) -> {statement;}

Lambda 表达式特点：可选的类型声明，可以不声明参数类型，可选的参数圆括号，一个参数时不需要写圆括号。没有参数或者多个参数需要写圆括号，可选的大括号，如果主体只包含一个语句时不需要大括号，包含多条语句需要大括号。可选的返回关键字，如果主体只有一个语句返回值，会自动返回， 

##### 形式一 没有参数，没有返回值

```java
    () -> System.out.println("Hello world")
    
    @Test
    public void test1(){
        Runnable runnable = () -> System.out.println("Hello world");
        runnable.run();
    }
```



##### 形式二 一个参数，没有返回值

```java
    (name) -> System.out.println(name) 
     name -> System.out.println(name)
    
    @Test
    public void test2(){
        Consumer consumer = (name) -> System.out.println(name);
        consumer.accept("yuhao");
    }
```



##### 形式三 没有参数，但主体有多条语句，没有返回值

```java
    () -> {
            System.out.print("Hello");
            System.out.print("world");
        };

		@Test
    public void test3() {
        Runnable runnable = () -> {
            System.out.print("Hello");
            System.out.print("world");
        };
        runnable.run();
    }
```



##### 形式四 两个参数,有返回值

```
    @Test
    public void test4() {
        IntBinaryOperator operator = (x, y) -> x + y;
        int sum = operator.applyAsInt(1, 2);
        System.out.println(sum);
    }
```



以上4个例子都是使用的 JDK 内置的4个函数式接口，如何定义函数式接口才能符合 Lambda 规范？

要求：

+ 接口只有一个抽象方法 
+ 用 @FunctionalInterface 注解修饰接口 （非必须只是提供给编译器校验该接口是否符合规范）

#### JDK 常用的函数式接口



|        接口         | 参数  | 返回值  |                    描述                    | 类型 |
| :-----------------: | :---: | :-----: | :----------------------------------------: | :--: |
|    Predicate<T>     |   T   | boolean |              用于判断一个对象              | 判断 |
|     Consumer<T>     |   T   |  void   |  用于接收一个对象并处理，但是没有返回值。  | 消费 |
|    Function<T R>    |   T   |    R    |      用于将一个对象转换为另一个对象。      | 函数 |
|     Supplier<T>     |  无   |    T    |                提供一个对象                | 供给 |
|  UnaryOperator<T>   |   T   |    T    |   接收一个类型对象并返回一个同类型的对象   |  -   |
| BinaryOperator<T T> | (T,T) |    T    | 接收两个同类型的对象并返回一个原类型的对象 |  -   |



#### 方法引用

 一种简写的 Lambda 表达式

目标引用 :: 方法名

如 Sku::getSkuPrice

指向静态方法的方法引用

```java
    (args) -> ClassName.staticMethod(args);
		ClassName::staticMethod;
		@Test
    public void test5() {
        Consumer<String> consumer = str -> Integer.parseInt(str);
        consumer.accept("100");
        Consumer<String> consumer1 = Integer::parseInt;
        consumer1.accept("100");
    }
```



指向实例方法的方法引用

```java
	  //(args) -> args.instanceMethod();
    //ClassName::instanceMethod;
    @Test
    public void test6() {
        Consumer<Integer> consumer = integer -> integer.intValue();
        consumer.accept(new Integer(100));
        Consumer<Integer>  consumer1 = Integer::intValue;
        consumer1.accept(new Integer(100));
    }
```



指向现有对象实例方法的方法引用

```java
    @Test
    public void test7() {
        //(args) -> instance.instanceMethod();
        //instance::instanceMethod;
        StringBuilder builder = new StringBuilder();
        Consumer<String> consumer = str -> builder.append(str);
        consumer.accept("100");
        Consumer<String> consumer1 = builder::append;
        consumer1.accept("100");
    }
```



