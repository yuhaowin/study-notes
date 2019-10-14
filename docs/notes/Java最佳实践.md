#### JAVA 中的最佳实践

########应该使用Collection.isEmpty()检测空

逻辑上可以使用 collection.size() == 0 判断集合为空，Collection.isEmpty()检测空可以使得代码更容易阅读，并且任何Collection.isEmpty()的实现时间复杂度都是 O(1),但是某些 Collection.size() 实现的时间复杂度可能是 O(n) 。


########集合初始化尽量指定大小

在使用集合框架时，可以尽可能的指定集合的大小，这样会减少集合扩容的次数。

```java
int[] arr = new int[]{1, 2, 3};
List<Integer> list = new ArrayList<>(arr.length);
```

######## for 循环中的中 字符串连接 使用 StringBuilder

一般的字符串拼接在编译期 java 会进行优化，但是在循环中字符串拼接， java 编译期无法做到优化，所以需要使用 StringBuilder 进行替换。

```java
String str = "a" + "b" + "c"; //编译器会优化

String s = "";
for (int i = 0; i < 10; i++) {
    s += i; //编译器无法优化
}
```

########List 的随机访问

数组实现的List 如：ArrayList 数组的随机访问效率更高，可以判断它是否实现 *RandomAccess* 接口判断是否可以提供高效的随机访问能力。

```java
// 调用别人的服务获取到list
List<Integer> list = otherService.getList();
if (list instanceof RandomAccess) {
    // 内部数组实现，可以随机访问
    System.out.println(list.get(list.size() - 1));
} else {
    // 内部可能是链表实现，随机访问效率低
}
```

########频繁调用 Collection.contains 方法请使用 Set

在 java 集合类库中，List 的 contains 方法普遍时间复杂度是 O(n) ，如果在代码中需要频繁调用 contains 方法查找数据，可以先将 list 转换成 HashSet 实现，将 O(n) 的时间复杂度降为 O(1) 

```java
ArrayList<Integer> list = otherService.getList();
for (int i = 0; i <= Integer.MAX_VALUE; i++) {
    // 时间复杂度O(n)
    list.contains(i);
}
```

```java
ArrayList<Integer> list = otherService.getList();
Set<Integer> set = new HashSet(list);
for (int i = 0; i <= Integer.MAX_VALUE; i++) {
    // 时间复杂度O(1)
    set.contains(i);
}
```


######## 静态的集合成员变量赋值

```java
private static Map<String, Integer> map = new HashMap<>();
static {
    map.put("a", 1);
    map.put("b", 2);
};
```



########使用String.valueOf(value)代替""+value

```java
int i = 1;
String s = String.valueOf(i);
//不用使用
String s = "" + i;
```





