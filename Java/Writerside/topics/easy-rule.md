# 规则引擎:Easy-Rule

在代码中，经常需要指定代码指定的规则，而这些规则往往都内嵌在代码逻辑的内部。这使得代码和规则之间的耦合，并且难以扩展。这时候，我们可以引入规则引擎，将规则从业务中抽离出来，实现规则与业务的松耦合。这篇文档介绍了 Easy Rule 的应用。

### 什么是规则引擎

关于规则引擎，Martin Fowler 在 《[Should I use a Rules engine?](http://martinfowler.com/bliki/RulesEngine.html)》一文中，有这样的表述:

> You can build a simple rules engine yourself. All you need is to create a bunch of objects with conditions and actions, store them in a collection, and run through them to evaluate the conditions and execute the actions.


大意是您可以自己构建一个简单的规则引擎。您所需要的只是创建一堆具有条件和操作的对象，将它们存储在集合中，然后运行它们以评估条件并执行操作。

### 验证器的例子

举例来说，比如我需要对接口的传参进行校验。这时候有两种做法，一种是每个接口都写重复的校验规则来校验。伪代码如下:

```java
public Response register(Params params) {
    if (params.getUsername === '') {......}
	if (params.getPassword.length < 6) {......}
}
```

另一种方式是，将校验的规则抽离出来:

```java
class NotIsBlank implements Rule {
    public boolean check(Object value) {......}
}
class Length implements Rule {
	public boolean check(Object value) {......}
}
```

然后实现一个验证器:

```java
class Validator {
	private List<Rule> rules;
	// 注册规则
    public void registerRule(Rule rule) {
		this.rules.push(rule);
	}
	// 校验规则
	public void check(Object value) {
		rules.forEach(rule -> rule.check(value));
	}
}
```

然后重写之前的注册的接口:

```java
public Response register(Params params) {
	Validator validator = new Validator();
    validator.registerRule(new NotIsBlank());
    validator.registerRule(new Length());
    validator.check(params);
}
```

通过重构，规则从业务代码中抽离出来，变得可复用、可扩展。不变的是校验的行为逻辑、异常处理等，这些都封装在“引擎”中，这部分的行为逻辑也变得可复用。我们的代码中就不再有那么多的 `if...else...`。

比如当我们上传 Excel 的时候，往往需要对每一个 Sheet、每一行中的每一个单元格进行校验，如果使用 `if...else..`会有多么的麻烦和难以维护，如果使用规则引擎的写法，那么就会变得简单，你只需要聚焦于规则本身。

### 使用 Easy Rule 轻量级的规则引擎

为什么是轻量级的规则引擎，当规则引擎的规则变得复杂，也会增加学习成本和应用、维护成本。而轻量级的规则引擎其实在大多数场景下已经足够。故而这里推荐使用 Easy Rule 这个基于 Java 编写的规则引擎。

#### 安装

这里介绍通过 Maven 安装最新版本:

```xml
<!-- https://mvnrepository.com/artifact/org.jeasy/easy-rules-core -->
<dependency>
	<groupId>org.jeasy</groupId>
	<artifactId>easy-rules-core</artifactId>
	<version>4.1.0</version>
</dependency>
```
<note>

目前，这个项目已经稳定，只修复 Bug，不再增加新的功能特性。所以，不会有比 v4.1.x 更高的版本了。

</note>


#### 定义规则

通过上面我们验证器的例子，已经阐释了规则引擎的实现原理。那么首先我们需要定义规则, 如下:

```java
public class CustomRule {

    @Condition
    public boolean check(@Fact("params") Params params) {
		// 这里实现规则的校验
    }

    @Action
    public void toDoAnyThings() {
		// 这里实现校验后要做的一些事情
    }
}
```

上面的示例中，出现了三个注解, 也是三个概念:

- `@Condition`： 表示 `check`方法是一个定义了规则的条件，将会实现是否满足规则的逻辑，返回布尔值
- `@Action`：表示实现校验后做的动作
- `@Fact`： 指定规则验证所需要的参数，不同的规则需要的参数可能不一样

#### 校验规则

定义了规则之后，在需要的时候注册规则，并传入参数校验即可:

```java
Rules rules = new Rules();
rules.register(new CustomRule());
// 定义参数
Facts facts = new Facts();
facts.put("param1", "value1");
facts.put("param2", "value2");
// 创建默认的规则引擎使用注册的规则对参数进行校验
DefaultRulesEngine defaultEngine = new DefaultRulesEngine();
Map<Rule, Boolean> result = defaultEngine.check(rules, facts);
```

### 总结

这篇文档首先介绍了什么是规则引擎，他是一组条件判断的集合。通过将条件判断抽象为规则，并从业务中分离出来，由引擎实现通用的校验逻辑。

接着，我们通过验证器的例子，来阐述了规则引擎的实现逻辑。这种编码的思路也可以作用在代码的各个方面，比如当你在写多个`if...else...`分支的时候，就可以考虑是否存在可扩展，是否可以抽离变化的部分。

最后，我们介绍了轻量级规则引擎 Easy Rule 的简单应用。