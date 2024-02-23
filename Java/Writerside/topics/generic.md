# 泛型

从 Java5 开始引入了泛型的概念，其是对 C++ 中的模板的借鉴。之后，泛型成了 Java 中不可缺少的一部分，大量的库都使用了泛型去编写。这篇文档详细描述了 Java 泛型的使用。

## 为什么要有泛型

根本原因是因为 Java 是一种强类型的语言，所以在弱类型的语言中并不需要泛型。我们以 PHP 的语法举例说明：
```PHP
$arr = [];
function addToArray($element) {
  $arr[] = $element;
}
addToArray(1);
addToArray("Hello");
```
定义了一个函数，可以将一个元素加入到数组中去。我们可以看到加入一个数值类型的1 是可以的，加入一个字符串类型的 Hello 也是可以的。

但同样的做法在强类型的 Java 中是不被允许的，因为所有的类型都必须在编译时候确定。一方面，强类型可以让我们的程序更加的健壮，另一方面相对于弱类型的语言其灵活性就不够。

在 Java 中要实现相同的功能，有下面这些做法：

1. 定义多个类，每一个类负责一中类型。比如说你需要定义一个 StringList类，然后再定义一个 IntegerList 类，接着再定义一个 FloatList 类、ShortList、BooleanList......无穷无尽。
2. 使用继承，但限制是只能传入子类，当然你可以使用 Object 类，但是这样做就意味着传入的时候需要向下转型，取出的时候需要向上转型，有较大的转型成本。另外，转型的前提是你知道这个类的类型，如果类型编写错误，原本是编译时错误，就会退化为运行时错误。
3. 使用接口，但限制是只能传入接口的实现类。

所以，从第 1 点，我们可以看出泛型增强了代码的可复用性。从第 2 点我们可以看出泛型消除了转型成本，提升了性能，也增强了类型安全，避免了因为转型引起的类型错误退化为运行时错误。

## 基本用法 ##

我们还是以 List 举例, 在 Java 中使用泛型是容易的:
```Java
List<String> list = new ArrayList<String>();
// 在 Java1.7 之后，可以简化构造函数中的类型，因为可以自动推断
List<String> list = new ArrayList<>();
```
需要注意的是，下面这些写法都是错误的, 前后类型必须一致:
```Java
// 前后类型必须一致，子类也不行
List<Animal> list = new ArrayList<Dog>();
List<Number> list = new ArrayList<Integer>();
List<Object> list = new ArrayList<String>();
```

## 泛型作为方法参数 ##

首先我们来编写一个三个类，分为别 Goods 商品类、Books 书本类以及 Seller 销售员类，代码如下:
```Java
public class Goods {}
public class Books {}
public class Seller {
    public void sell(List<Goods> list) {
        list.forEach(System.out::println);
    }
}
```

然后我们来测试这个泛型作为传参的方法，将 `Books`类传入，没有意外的话就会循环商品输出类名:
```Java
public class Main {
    public static void main(String[] args) {
        List<Books> goods = new ArrayList<>();
        goods.add(new Books());
        goods.add(new Books());
        new Seller().sell(goods);		// 编译错误
    }
}
```
嗯，因为你传入的这个 Books 类型和方法中的 Goods 泛型是不一致的。如何让其一致呢?
```Java
List<Goods> goods = new ArrayList<>();	// 改为 Goods
goods.add(new Books());
new Seller().sell(goods);
```
但是这样有一个问题，就是所有 Goods 的子类都可以加入其中，违反了泛型的初衷。我们来修改泛型参数定义,  这样就解决了这个问题:
```Java
public class Seller {
	// 表示可以接收 Goods 以及其子类
    public void sell(List<? extends Goods> list) {
        list.forEach(System.out::println);
    }
}
// 修改 main 方法如下
List<Books> goods = new ArrayList<>();
```
## 范型通配符

`Class<?>` 中的 `<?>` 表示一个通配符，它在 Java 中称为无界通配符。这个通配符用于表示可以匹配任何类型的 `Class` 对象，但是具体是哪种类型要在使用时确定。

在 Java 中，Class  类用于表示类的元数据，可以通过它获取有关类的信息，如类名、字段、方法等。使用 `Class<?>` 表示你不确定具体的类类型，可以在使用时动态决定。
