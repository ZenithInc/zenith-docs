# PHP8 重要特性

这篇文档总结了 PHP8 中引入的新特性，我挑选了我觉得重要的几个特性来说一说，并给出了部分特性的示例。

### JIT {id="jit"}

PHP8 中引入的最大的一个特性就是  JIT（Just in time）编译器。当有人问，**PHP 是编译型语言还是解释型语言的时候，就可以回答解释+编译。**那么 Java 呢？编译，字节码在 JVM 中是解释加 JIT 编译。那么什么是 JIT 编译技术呢？

因为解释型语言有一个被诟病的地方就是性能不太好。比如 PHP 是由 PHP 解释器边解释边执行的，这个过程慢，而且无法引入很多编译优化技术。所以就引入了** JIT 技术，在运行阶段对热点代码（频繁执行的代码片段）进行编译，从而提升运行速度。**在这个过程中，还可以引入很多的编译优化技术，比如说去除无用的代码、内联替换、循环展开等。

但是需要注意的是，**JIT 编译器的只是提升了 PHP 对计算密集型任务的处理效率。对于 IO 密集型的应用，由于瓶颈还是在数据库访问上，所以优化不大。**IO 密集型的应用，需要引入协程技术来解决。比如 Swoole、Go。

> 协程是什么？基于多线程的用户态的任务调度。这样的说法可能不严谨，但是说明了协程两个重要的特点，基于多线程、相对于进程和线程都是在内核态的调度，它是用户态的任务调度。所以被称为轻量级的线程。


### 命名参数 {id="named-arguments"}

这个对代码的可读性来说，非常棒的功能。看下面的示例:

```php
function sum(int $a, int $b): int {
  return $a + $b;
}
echo sum(a: 1, b: 2);
```

所以，编程的一大难题在于如何给变量命名，好的变量命名（简短+含义清晰）是提高代码可读性的一大关键。可能也是国内程序员和国外的程序员的一大差距吧。

### 原生支持的注解 {id="new-attribute"}

这是我之前最期待的一个特性，早就厌倦了去解析 `/****/`注释中的注解了。**使用注解可以让代码更加优雅，面向切面编程（AOP）:**

```php
#[Attribute(Attribute::IS_REPEATABLE|Attribute::TARGET_METHOD)]
class Param {
    public function __construct(
        public string $name,
        public string $rule,
        public string $message
    ) {} 
}

class Controller
{
   
    #[Param(name: "username", rule: "NotEmpty", message: "username not empty!")]
    #[Param(name: "password", rule: "NotEmpty", message: "password not empty!")]
    public function register() {}
}
```

上面，我使用 PHP8 提供的注解实现了优雅的接口参数校验（注意，这里不包含利用反射解释注解的代码）。

另外，上面的代码我们也演示了在构造方法中使用  Constructor property promotion 这个特性，可以直接在构造方法的声明中加入属性的声明。不过我对这种写法有一定的疑虑，不觉得这可以提升代码的可读性，特别是如果一个类中部分属性在构造方法中声明的情况下，是严重降低可维护性、可读性的。

### 联合类型 {id="union-types"}

这个特性在 TypeScript 和 PHP 中都提供了，我觉得**联合类型是一种弱类型向强类型的过度做法。**

```php
class Test
{
  // 也可以在方法的参数、返回值声明
  public int|float $number;
}
```

由于 PHP 很多历史代码都是动态类型的，可以接受字符串的数字、可以接受整型、可以接受浮点型。由于近些年来强类型的趋势，所以这种特性我个人认为只是一种过度做法，兼容老的代码。新代码中尽量避免。

> 注意: 不是说弱类型不好，其实弱类型更加灵活，更加考验程序员对程序的掌控能力。但是对于企业级的项目，需要弱化程序员的随意性，需要规范性。这是一种矛盾。


### 更多特性 {id="more"}

更多的特性，请参考 [PHP 的官网](https://www.php.net/releases/8.3/en.php)。