# Lamdba

JDK1.8 之后，提供了 Lamdba 表达式，他可以让我们的程序编写更加简化、显得更加优雅。利用 Lamdba
可以更简洁地实现匿名内部类与函数声明调用。在这个基础上，还提供了 stream 流式处理极大地简化了对集合的操作。这篇文档将对
Lamdba 表示式做出详细的描述。

## 与传统写法的对比

我们先来看一个对集合排序的例子，直观地感受一下使用 Lamdba 表达式和之前传统的写法之间明显的区别。下面是传统的集合排序写法:

```Java
List<String> names = Arrays.asList("PHP", "C", "C++", "Java");
names.sort(new Comparator<String>() {
	@Override
	public int compare(String o1, String o2) {
		return o1.compareTo(o2);
	}
});
```

如果改成 Lamdba 表示式，上面的写法就简化成了如下形式：

```Java
List<String> names = Arrays.asList("PHP", "C", "C++", "Java");
names.sort(String::compareTo);
```

但从代码上看，就直接去掉了 6 行。

## 基础语法解析

一条 Lamdba 表示式由三部分组成，形式如下:

```Java
(参数列表) -> 实现语句
```

解释如下:

* 参数列表：可以由 1 到多个参数，中间使用逗号分隔。如果是单参数，括号可以省略。参数类型有可以省略。
* ->:  这是 Lmdba 表达式的操作符，固定写法，编译器据此来识别是否是 Lamdba 表达式。
* 实现语句: 如果是单行则直接写就行，如果是多行使用 {} 包裹。

## 方法引用语法解析

上文中，出现了`names.sort(String::compareTo);` 这样的写法，当中 `String::compareTo`
这种就叫做方法引用，表示用来直接访问类或者实例中已经存在的方法或者构造方法。其语法如下：

```Java
 String         ::         compareTo
目标引用  双冒号分隔符        方法名
```

方法引用一共分为四类，如下表所示:
|方法引用类型 |表现形式|
|---|---|
|指向静态方法| Class::staticMethod|
|指向现有对象的实例| Object::instanceMethod|
|指向任意类型的实例方法| Class::instanceMethod|
|指向构造方法| Class::new|

举例来说明，比如我要判断一个列表是否为空，如果为空的话给默认值:

```Java
Optional.of(numbers).orElseGet(ArrayList::new);
```

## 写法示例

Lamdba 表示式就是对实现类的简化写法。比如说定义一个四则运算的接口如下:

```Java
public interface MathOperation {
    public Float operate(Integer a, Integer b);
}
```

然后利用 Lamdba 表达式，就可以来实现这个接口:

```Java
MathOperation addition = (Integer a, Integer b) -> a + b + 0f;
System.out.println(addition.operate(1, 2)); // 输出:3.0
```

上面的写法还可以继续简化，如下:

```Java
MathOperation subtraction = (a, b) -> a - b + 0f;
System.out.println(subtraction.operate(4, 2)); // 输出: 2.0
```

<warning>
注意: Lamdba 表达式对实现的接口是有严格要求的: 只能实现有且只有一个抽象方法的接口，这种接口称之为 “函数式接口”。
</warning>

## @FunctionalInterface 接口

如果你去看 JDK 中的函数式接口的源码的话，都会在接口的上方看到一个名为`@FunctionalInterface`
的注解。这个注解对于代码本身来说并没有任何影响，加上或者不加上都是一样的。

```Java
@FunctionalInterface
public interface Function<T, R> {}
```

这个注解是给编译器看的，编译器看到之后就会去检查你这个接口是否为函数式接口，是否有且只有一个抽象方法。

## 函数式编程

函数式编程的核心在于，让函数称为和类一样的一等公民，可以定义为变量、可以在函数之间传递。关于函数式编程和面向对象编程的区别，如下表所示:

|       | 面向对象编程          | 函数式编程        |
|-------|-----------------|--------------|
| 设计思路  | 	侧重过程，重分析，重设计   | 	侧重结果，快速实现   |
| 可读性	  | 对于复杂的结构，可读性较差	  | 在特定场景下，可读性较好 |
| 代码量   | 	完全面向对象的实现，代码量多 | 	某些特定场景，代码量少 |
| 并发问题	 | 设计不当，会出现线程安全问题  | 	不会出现线程安全问题  |
| 健壮性   | 	好              | 	相对差         |
| 使用场景  | 	中大型多人协作的项目	    | 小型应用快速实现     |

<note>
注意：上面的这些对比都并不是非常严谨，都要视场景而定。比如使用场景，更多的时候其实都会混合多种编程范式，根据场景不同而定。抛开具体场景谈这些区别，就是在耍流氓。
</note>

而在 Java 中，函数式编程就是基于函数式接口并使用 Lamdba 表达式的编程方式，在上面的章节中我们已经演示了函数式接口和 Lamdba
表达式。

下面介绍一些常用的函数式接口:

| 函数式接口               | 参数类型   | 返回类型      | 描述                    |
|---------------------|--------|-----------|-----------------------|
| `Consumer<T>`       | 	T     | 	void	    | 对应有一个输入参数无输出的功能代码     |
| `Function<T,R>`     | T      | 	R        | 对应有一个输入参数且需要返回数据的功能代码 |
| `Predicate<T>`      | 	T     | 	 Boolean | 	用于条件判断，固定返回 Boolean  |
| `Supplier<T>`       | 	None  | 	T        | 	提供一个对象               |
| `UnaryOperator<T>`  | 	T     | 	T	       | 接受对象并返回同类型的对象         |
| `BinaryOperator<T>` | 	(T,T) | 	T	       | 接收两个同类型的对象并返回一个原类型的对象 |

### Predicate 函数式接口 ###

在 Java1.8 以后，新增了很多的函数式接口，位于 `java.lang.function` 包下。其中就包括 Predicate接口，这个接口是为了测试传入的数据是否满足要求返回一个
Boolean 值，它只有唯一的方式名为`test`。

```Java
Predicate<Integer> less = (a) -> a > 30;
Boolean result = less.test(4);
System.out.println(result); // 输出: false
```

怎么如何应用于集合呢？

```Java
public List<Integer> filter(List<Integer> numbers, Predicate<Integer> tester) {
	List<Integer> list = new ArrayList<>();
	for (Integer number: numbers) {
		if (tester.test(number)) {
			list.add(number);
		}
	}
	return list;
}
```

测试代码如下:

```Java
List<Integer> numbers = Arrays.asList(87, 29, 37, 98, 82, 58, 73);
// 输出: [29]
System.out.println(filter(numbers, n -> n < 30));
// 输出: [87, 37, 98, 82, 58, 73]
System.out.println(filter(numbers, n -> n > 30));
```

### Consumer 函数式接口 ###

还有就是 Consumer 函数式接口，也比较常用。它用于在只有一个参数并且不返回任何参数的接表达式。此外还有
IntConsumer这样的接口，其用于只有一个整数型参数的表达式，类似的还有 DoubleConsumer。

```Java
public void output(Consumer<String> consumer) {
	String text = "hello world";
	consumer.accept(text);
}
output(text -> System.out.println("Output:" + text));
```

### Function 函数式接口 ###

Function 接口接受一个参数，并且可以返回结果。我们以生成随机字符串为例演示其用法:

```Java
Function<Integer, String> randomString = length -> {
	String chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
	StringBuilder buffer = new StringBuilder();
	Random random = new Random();
	for (int i = 0;i < length; i++) {
		int index = random.nextInt(chars.length());
		buffer.append(chars.charAt(index));
	}
	return buffer.toString();
};
System.out.println(randomString.apply(10)); // 输出: trDOPBrGDk
System.out.println(randomString.apply(16)); // 输出: SCqhTQDXjBLbllVZ
```

## Stream 流 {id="stream"}

在正式说 Stream 流之前，着重强调一下：Lamdba 中所说的 Stream 和 I/O Stream 没有任何关系。这很容易让初学者感到迷惑。

Stream 流式处理是建立在 Lamdba 表示式基础上的多数据处理技术。它对集合数据处理进行了高度抽象，极大简化了处理集合数据的代码量。包含了对集合进行迭代、去重、筛选、聚合、排序等一系列的操作。

```plantuml
@startmindmap

+ Stream
++ 终端操作 Terminal
+++_ 非短路操作
++++_ count 计数
++++_ max 最大值
++++_ min 最小值
++++_ forEach 遍历
++++_ collect 聚合
++++_ reduce 归约
+++_ 短路操作
++++_ allMatch 全部匹配
++++_ noneMatch 不匹配
++++_ findFirst 查找首个
++++_ findAny 查找任意一个
++++_ anyMatch 任意匹配
++ 中间操作 Intermediate
+++_ 有状态操作
++++_ sorted 排序
++++_ limit 截断
++++_ distinct 去重
++++_ skip 跳过
+++_ 无状态操作
++++_ map 映射
++++_ peek 遍历
++++_ filter 过滤
++++_ flatMap 扁平化
@endmindmap
```

    
### 使用流的示例 {id="stream_example"}

下面的例子展示了从一组数据中筛选出最大的一个偶数:

```Java
Optional<Integer> op = Stream.of(1, 2, 3, 4, 5)
	.filter(n -> n % 2 == 0)
	.max(Comparator.comparingInt(a -> a));
op.ifPresent(System.out::println);      // 输出: 4
```

通过这个例子我们可以看到使用 Stream 可以对我们的集合处理代码极度的简化，代码量少了可读性高了。通过流失处理，我们可以一步一步得到我们最终的结果。

### 创建 stream 的5种方法 {id="the_way_of_create_stream"}

在 Java 中，提供了 5 种创建流的方式，第一种是基于数组创建：

```Java
String[] languages = {"Java", "python", "Shell", "PHP"};
Stream<String> s = Stream.of(languages);
s.forEach(System.out::println);
```

第二种方法是基于 List 进行创建:

```Java
List<String> list = new ArrayList<>();
list.add("Java");
list.add("c");
list.add("VB");
list.stream().sorted().forEach(System.out::println); // 输出: Java VB c
```

第三种方法是利用 Generate 方法来创建无限长度的流。下面的例子展现了生成无限随机数的例子:

```Java
Stream<Integer> generator = Stream.generate(() -> new Random().nextInt(100000));
generator.forEach(System.out::println); // 无限输出: 32135 40533 51978 .......
generator.limit(10).forEach(System.out::println); // 输出十个: 32135 40533 51978 .......
```

第四种方法是基于迭代器创建, 下面的例子输出了 10 个奇数:

```Java
Stream<Integer> iterator = Stream.iterate(1, n -> n + 2);
iterator.limit(10).forEach(System.out::println);
```

第五种方法是基于字符序列创建流, 一般用在字符加密或者对字符进行处理的时候:

```Java
String str = "abcdefg";
str.chars().forEach((i) -> System.out.println((char)i));
```

### 使用流的示例 {id="stream_usages"}

下面展示了一些上面提到的方法的示例，以供参考。

reduce 方法示例，它是将一组数据然后减少到单个数据，例如计算一组数据的和:

```Java
Integer result = Stream.generate(() -> new Random().nextInt(100))
	.limit(3)
	.parallel()			// 并行计算需要加上这个方法的调用
	.peek(System.out::println)
	.reduce(0, Integer::sum, Integer::sum);
System.out.println(result);		// 三个随机数的求和： 34 + 14 + 23 = 71
```

上面的代码随机生成三个数值，然后打印出来，最后使用 reduce 方法来求和。reduce 方法接受 3
个参数，第一个参数是初始值、第二个参数是两个参数之间的算法（示例中是求和）、第三个参数是并行计算下的算法。

和reduce 相反，collect
是在一组数据中收集符合要求或者经过处理的一组数据。但是用法是相似的，最多接受三个参数，分别是初始值、合并的方法以及并行计算的方法。下面的例子演示了，用户订单额的统计:

```Java
// 创建一个订单类，有两个属性，分别是用户名以及订单总金额
class Order {
	private final String user;
	private final Double total;
	// 省略全参构造函数以及 Getter 、Setter
}
// 构造测试数据
list.add(new Order("ZhangSan", 998D));
list.add(new Order("LiSi", 666D));
list.add(new Order("ZhangSan", 550D));
list.add(new Order("LiSi", 222D));
```

然后根据上面构造的订单数据，使用 collect 方法来分组统计每个用户下单总金额:

```Java
HashMap<String, Double> result = list.stream().parallel().collect(
	HashMap::new, 		// 初始值
	// 如何将订单数据汇总到 Map 中
	(HashMap<String, Double> map, Order order) -> {
		String user = order.getUser();
		if (map.containsKey(user)) {
			map.put(user, map.get(user) + order.getTotal());
		} else {
			map.put(user, order.getTotal());
		}
	},
	// 在并行计算下，如何合并两个计算后的 Map
	(HashMap<String, Double> map1, HashMap<String, Double> map2) -> {
		map2.forEach((user, total) -> {
			map1.merge(user, total, Double::sum);
		});
	}
);
```

上面的代码可以重构的更为简短:
```Java
Map<String, Double> result = list.stream().collect(
   Collectors.groupingBy(
       Order::getUser,
       Collectors.reducing(0.0, Order::getTotal, Double::sum)
   )
);
```

## 总结 {id="summary"}

这篇文档提供了关于 Java 语言中 Lambda 表达式的全面介绍，包括其在 JDK 1.8 之后的引入、基础语法、使用示例以及与传统编程方式的对比。文档首先展
示了 Lambda 表达式简化编程的优势，尤其是在集合排序等操作中，通过对比传统写法和 Lambda 表达式，突出了后者的简洁性。接着，文档详细解释了
Lambda 表达式的三个组成部分：参数列表、操作符（->）和实现语句。

文档还深入探讨了方法引用语法，解释了方法引用的四种类型，并通过实例加以说明。Lambda 表达式作为函数式编程的一个重要组成部分，文档阐述了其在 Java
 中的应用，特别是对函数式接口（只有一个抽象方法的接口）的要求，以及如何通过 `@FunctionalInterface` 注解标记这些接口。

进一步，文档比较了面向对象编程和函数式编程的不同特点，包括设计思路、可读性、代码量、并发问题处理、健壮性和使用场景。此外，还介绍了 Java 中常用
的几种函数式接口，如 `Consumer<T>`, `Function<T,R>`, `Predicate<T>`, `Supplier<T>`, `UnaryOperator<T>` 和 `BinaryOperator<T>`，
并通过示例展示了它们的应用。

最后，文档深入讲解了基于 Lambda 表达式的 Stream 流式处理，强调了其与 I/O Stream 无关。Stream 提供了对集合数据处理的高度抽象，包括迭代、
去重、筛选、聚合和排序等操作。文档通过丰富的示例展示了如何使用 Stream 流，如何创建流，以及如何利用流进行数据处理，最后通过几个具体的例子
（如 reduce 和 collect 方法）展示了 Stream 的强大功能。