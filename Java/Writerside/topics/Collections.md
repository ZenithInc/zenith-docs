# 集合

集合在 Java 中是非常重要的一部分，常用来代替数组。集合中包括 List、Set、Map 等数据结构，这篇文档将详细描述集合中不同的数据结构的使用以及原理。

## 概述

在 Java 以及其他的编程语言中，都会有数组的概念，用来表达一组数据。那么为什么有了数组，我们还要多一个集合的概念呢？这就要说数组的一些不方便之处了:数组必须在声明的时候指定元素的个数，但更多的时候吗，我们并不知道会有多少的元素。

```plantuml
@startmindmap
* 集合
    * List
        * 有序可以重复的集合
    * Set
        * 无序不可重复集合
    * Map
        * 代表映射关系
    * Queue
        * 队列,FIFO
@endmindmap
```


如上图所示，Java 提供了一系列的基于不同的数据结构的结合类，他们有着不同的应用场景，可用来存储数量不等、不确定的对象。上面四种类型都是基于 Collection 和  Map 两个接口派生而来, 如下图所示:

```plantuml
@startuml
interface Collection
interface Set
interface Queue
interface List

Collection <|-down- List 
Collection <|-down- Set
Collection <|-down- Queue

class EnumSet
interface SortedSet
class HashSet

EnumSet .up.> Set
SortedSet -up-> Set
HashSet .up.> Set

interface Deque
class PriorityQueue

Deque -up-> Queue
PriorityQueue .up.> Queue


class ArrayList
class Vector

ArrayList .up.> List
Vector .up.> List

class TreeSet

TreeSet .up.> SortedSet

class LinkedHashSet

LinkedHashSet -up-> HashSet

class ArrayDeque

ArrayDeque .up.> Deque

class LinkedList

LinkedList .up.> Deque
LinkedList .up.> List

class Stack

Stack .up.> Vector

@enduml
```

接着我们在看看 Map 接口的图示:

```plantuml
@startuml
interface Map

class EnumMap
class WeakHashMap
class IdentitityHashMap
class HashMap
class HashTable
interface SortedMap

EnumMap .up.> Map
WeakHashMap .up.> Map
IdentitityHashMap .up.> Map
HashMap .up.> Map
HashTable .up.> Map
SortedMap -up-> Map

class LinkerHashMap
class Properties
class TreeMap

LinkerHashMap -up-> HashMap
Properties -up-> HashTable
TreeMap .up.> SortedMap
@enduml
```

## 最常用的 List ##

List 集合代表一个元素有序、可重复的集合，集合中每一个元素都有其对应的顺序索引。其允许使用重复元素，通过索引访问指定位置的元素，默认按照元素的添加顺序设置元素的索引。

### Array List ###

我们先来介绍 ArrayList，它是基于数组实现的 List 类，是 Java 数组的有效替代品。它会自动对容量进行扩容，多数情况下无序指定最大长度。基本的用法如下:

```Java
ArrayList<String> languages = new ArrayList<>();
languages.add("PHP"); // 数据的类型都要保持一致。
languages.add("Java");
// 如果超出了索引的范围，就会抛出 IndexOutOfBoundsException
System.out.println(languages.get(1)); // 输出: Java
```

如果希望在指定位置插入，同样可以使用`add`方法:
```Java
languages.add(1, "C#");  // 在指定位置插入
System.out.println(languages.get(1)); // 输出: C#
```
更新列表中的数据可以使用set 方法, 同样不能超过索引范围:
```Java
languages.set(1, "C++");
System.out.println(languages.get(1));  // 输出: C++
```

还剩下一个删除的操作，如下:
```Java
languages.remove(1);
System.out.println(languages); // 输出: [PHP, Java]
languages.remove("PHP");
System.out.println(languages); // 输出: [Java]
```

获取元素的个数:
```Java
System.out.println(languages.size()); // 输出: 1
```

排序操作如下:
```Java
List<Integer> list = new ArrayList<>();
list.add(2048);
list.add(1024);
list.add(4096);
list.sort(Comparator.naturalOrder());
System.out.println(list);   // 升序输出: [1024, 2048, 4096]
list.sort(Comparator.reverseOrder());
System.out.println(list);   // 倒序输出: [4096, 2048, 1024]
```

### LinkedList ###

LinkedList 同时实现了 List 和 Deque 两个接口，在保障有序、允许重复的前提下，也可以作为列表在队首、队尾快速追加数据。 List 中的数据在内存中是分散存储的，基于链表数据结构，所以拥有良好的差距插入数独，但数据访问速度是低于 ArrayList 的。

大部分的操作和 ArrayList 是一样的，所以下面只是演示一些不同的用法。添加元素:
```Java
LinkedList<String> languages = new LinkedList<>();
languages.addFirst("PHP");
languages.addLast("Java");
System.out.println(languages);
```

### ArrayList 和 LinkedList 的区别 ###

ArrayList 底层是基于 Array 的，在 Array 的基础上添加了 add、set 等方法，相比 Array 会更好用。所以数据的存储和数组是一样的，元素在内存中都是连续的，如下表所示:

|0|	1|	2|	3|	4|
|---|---|---|---|---|
|PHP	|Java|	C|	C++|	C#|

**这样的存储结构决定了访问某个元素、遍历元素的效率是非常高的。但是如果想要在中间插入元素，那么效率是很差的，因为中间并没有空闲的空间。比如在 Java 的后面插入 Shell, 那么 C、C++以及 C# 这些位于后面的元素都需要向后移动，空出一个位置给插入的元素。其他元素在内存上需要重新调度、分配，所以这样的效率是十分低下的。**

而 LinkedList 在内存中是分散存储的，并不连续，如下图所示:

```plantuml
@startuml
usecase "PHP"
usecase "Java"
usecase "C"
usecase "C++"
usecase "C#"

"PHP" -> "Java"
"Java" -> "C"
C -> "C++"
"C++" -> "C#"

"C#" -> "C++"
"C++" -> "C"
"C" -> "Java"
"Java" -> "PHP"
@enduml
```

元素与元素之间是通过双向链表连接在一起的，可以从 PHP 找到 Java ，也可以从C#一路找到 PHP。**所以在遍历元素上，是比 ArrayList 要低效的。而因为并不是连续的，如果要插入一个元素是高效的，只是更改前后元素的指针即可。**

所以，**如果你的场景是大量的读取操作的话，应该选择 ArrayList。如果你的场景是大量的插入操作的话，就需要选择 LinkedList。**

### List 集合的遍历 {id="the_list"}

对 List 集合进行遍历有三种方式，分别是通过 for循环遍历、通过 forEach 方法遍历、通过 Iterator 迭代器遍历。

第一种方法是通过for 循环进行遍历，这是平时开发中最常用的方法:
```Java
LinkedList<String> languages = new LinkedList<>();
languages.addFirst("PHP");
languages.addLast("Java");
for (String language : languages) {
	System.out.println(language);
}
// 从 Java9 开始，可以使用下面的方式初始化
List<Integer> list = List.of(1, 2, 3);
System.out.println(list);
```

第二种方式是使用forEach,  如果只是简单的操作，那么使用基于 Lamdba 表达式的方式更加简洁:
```Java
// 完整的写法
languages.forEach(language -> {
	// 循环体
	System.out.println(language);
});
// 简写
languages.forEach(System.out::println);
```

第三种是使用Iterator 方式，这种方式现在是很少使用了， 迭代器是一次性的:

```Java
Iterator<String> iterator = languages.iterator();
while (iterator.hasNext()) {
	System.out.println(iterator.next());
}
```

<warning>
注意: 迭代器中的元素只能迭代一次，第二次迭代就不存在了。这是因为迭代器中使用指针会顺着元素不断向下移动，但是这个指针在移动之后，是无法自动复位的。在移动到了底部之后，就没有元素了。
</warning>

## Set ##

Set 集合代表一个元素无序且不可重复的集合。和 List 集合的使用方法基本相同，只是处理行为略有不同。平时比较常用的是 `HashSet` 和 `TreeSet` 两种，此外还有 `LinkedHashSet` 和 `EnumSet` 。

### 基本用法 ###

首先，我们添加了两个 C#， 但是最终输出的只有一个。另外，它没有按照我们代码添加的顺序输出。这就演示了 HashSet 的两个特性，无序以及去重。因为是无序的，所以你不能像 List 那样通过 get 传递索引获取元素。

```Java
Set<String> languages = new HashSet<>();
languages.add("PHP");
languages.add("C#");
languages.add("Java");
languages.add("C#");
System.out.println(languages);  // 输出: [C#, Java, PHP]
```

下面的代码演示了如何判断元素在 Set 中是否已经存在，以及如何获取 Set 中元素的个数:
```Java
System.out.println(result);     // 输出: true
System.out.println(languages.size());   // 输出: 3
```

### HashSet 与 TreeSet 的存储原理 ###

HashSet 是 Set 接口的典型实现，大多数时候使用 Set 集合时就是使用这个实现类。它是使用 Hash 算法来决定集合元素的顺序，具有很好的查找性能。当向 HashSet 集合中存入一个元素时，根据该对象的  hashCode 值决定该对象在 HashSet 中的存储位置。

LinikedHashSet 是 HashSet 的子类，除 HashSet 的特性外，它同时使用链表维护元素的次序，可以保障按照插入顺序提取数据。LinkedHashSet 需要维护元素的插入顺序，因此性能上略低于 HashSet。但是当需要迭代 Set 里素有的元素时，因为有链表维护所以性能就很好。下面例子演示了使用 LinedHashSet 按照插入顺序输出:
```Java
Set<Integer> numbers = new LinkedHashSet<>();
numbers.add(2048);
numbers.add(1024);
numbers.add(4096);
System.out.println(numbers);   // 输出： [4096, 2048, 1024]
```

TreeSet 时 SortedSet 接口的实现类，TreeSet 可以确保元素处于排序状态。TreeSet 采用红黑树的数据结构来存储集合元素。默认情况下，采用自然排序对元素升序排列，也可以实现 Comparable 接口自定义排序方式。下面的例子展示了使用 TreeSet 进行自定义排序:

```Java
Set<Integer> numbers = new TreeSet<>(Comparator.comparingInt(o -> -o));
numbers.add(1024);
numbers.add(2048);
numbers.add(4096);
System.out.println(numbers);   // 输出： [4096, 2048, 1024]
```

## Map ##

Map 用于保存具有映射关系的数据，每组映射都是 Key 与 Value 的组合。Key 和 Value 可以是任何引用类型的数据，但是 Key 通常是 String 类型的。Map 中的 Key 不允许重复，重复为同一个 Key 设置 Value，后者 Value 会覆盖前者 Value。

在日常的开发中，比较常用的有 HashMap、LinkedHashMap 以及 TreeMap，它们之间的关系和前文中描述的 HashSet 、LinkedHashSet 以及 TreeHashSet 是类似的。需要说明的是，Java 中是先有 Map 后有 Set，HashSet 是从 HashMap 精简而来的。

不常用的还有 EnumMap 、IdentityHashMap 、HashTable 、Properties 和 WeakHashMap 。

### HashMap ###

下面的代码演示了 HashMap 的定义和遍历:
```Java
Map<String, Object> user = new HashMap<>();
user.put("Name", "ZhangSan");
user.put("Age", 18);
user.forEach((key, value) -> System.out.println("key:" + key + " value:" + value));
```
如果我们要判断一个 Key  在 Map 中是否存在，可以使用 containsKey 方法:
```Java
System.out.println(user.containsKey("Address"));        // 输出： false
System.out.println(user.containsValue("ZhangSan"));     // 输出： true
```
移除一个 Key 使用 remove 方法:
```Java
user.remove("Age");
System.out.println(user);   // 输出: {Name=ZhangSan}
```

### LinkedHashMap ###

另外，LinkedHashMap 和我们之前说的 LinkedHashSet 一样，只是通过链表来维护插入字段的顺序：
```Java
Map<String, Object> user = new LinkedHashMap<>();
user.put("Name", "ZhangSan");
user.put("Age", 18);
System.out.println(user);   // 输出: {Name=ZhangSan, Age=18}
```

### TreeHashMap ###

TreeMap 存储键值对时，需要根据 key 对节点进行排序，底层是根据红黑树实现的。它支持两种 Key 排序，分别是自然排序和自定义排序。

```Java
Map<String, Object> map1 = new TreeMap<>(Comparator.reverseOrder());
map1.put("b", 1);
map1.put("a", 2);
map1.put("c", 3);
System.out.println(map1);   // 倒序输出: {c=3, b=1, a=2}
```

## 集合中的错误用法 ##

下面针对一些集合中的错误用法以及最佳实践进行说明。

### 列表初始化导致内存泄漏 ###

列表可以通过匿名内部类完成初始化，这样的初始化方式有可能会导致内存泄漏，不建议采用这种方式:
```Java
List<Integer> numbers = new ArrayList<>(){
	{
		add(1);
		add(2);
	}
};
System.out.println(numbers);  // [1, 2]
```