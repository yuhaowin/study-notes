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



