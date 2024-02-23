# Spring State Machine

在开发中，我们总是要和各种状态打交道，比如游戏状态、订单状态。在处理这些状态的编码中，经常容易出错，并且状态和业务逻辑耦合在一起，并不容易扩展。所以这篇文档介绍了在项目中使用
Spring State Machine 这个库，引入状态机的概念去管理业务中的各种状态。

## 什么是状态机 {id="what"}

状态机（State Machine）是计算机科学中的一个抽象概念，用于描述对象在不同状态之间的转换。状态机主要由三个部分组成：

1. 状态（States）：这是系统在任何给定时刻可以存在的一种或多种条件。例如，一个简单的交通灯状态机可能有三个状态：红、黄和绿。

2. 转换（Transitions）：这是从一个状态到另一个状态的逻辑或条件。继续使用交通灯的例子，当红灯亮起时，经过一段时间后会转变为绿灯，这种状态的变化就是一个转换。

3. 事件（Events）：这是触发转换的外部因素。在某些状态机中，事件是驱动状态更改的主要因素。

状态机在很多领域都有应用，如电路设计、软件工程、网络协议和计算机算法等。设计状态机的目的通常是为了对复杂系统的行为进行建模和分析，以确保系统能够以预期的方式工作。

其中，有限状态机（Finite State Machine, FSM）是状态机中的一个特殊子集，其中的状态数量是有限的。FSM在计算机科学和电气工程中有广泛的应用，包括在硬件设计、解析器、正则表达式和某些类型的算法中。

简单来说，**在软件领域，所谓的状态机就是将状态、状态的转换从业务逻辑中抽离出来，然后通过事件驱动的方式去触发状态的流转。使得状态和业务代码松耦合、可扩展。
**

## 状态机和状态设计模式 {id="diff"}

在软件领域，状态机常常是通过“状态模式”实现的。

状态模式是一种设计模式，它允许对象在其内部状态改变时改变它的行为，好像对象的类似乎被改变了一样。通过使用状态模式，可以将与特定状态相关的行为封装在单独的类中，并使用这些类对象来表示当前的状态。

状态模式的主要组成部分如下：

1. 上下文（Context）：定义客户感兴趣的接口并维护一个具体状态实例，这个实例定义了当前的状态。

2. 抽象状态（State）：定义一个接口，用来封装与上下文的一个特定状态相关的行为。

3. 具体状态（Concrete State）：每个子类实现与上下文的一个状态相关的行为。

状态模式的好处：

1. 封装性：每个状态的行为都被封装在它自己的类中，使得状态的转换变得明确和易于管理。

2. 更好的组织：状态模式提供了一个更好的组织结构来管理一个对象在不同状态下的行为，特别是当有很多状态时。

3. 扩展性：当需要添加新的状态或者更改状态间的转换逻辑时，状态模式使得这些更改变得更加简单，因为每个状态都是独立的类。

状态模式与状态机的不同之处在于，状态模式主要关注如何在软件设计中组织和管理对象的状态，而状态机主要关注状态之间的转换和如何执行这些转换。但是，两者可以结合使用，以提供一个完整、结构化和灵活的系统来处理对象的状态和行为。

## Spring State Machine 的应用 {id="usage"}

在 SpringBoot 项目中, 可以使用 Spring 官方的状态机的实现，当然相对于其他实现来说，它的实现更复杂也更强大，需要一定的学习成本。

### Install {id="install"}

Spring State Machine 主要有 v2 和 v3 两个版本。主要的区别是 v3 加入了响应式的实现。v2 使用的最为广泛，而 v3 在部分 API
上并不兼容 v2 版本，且网上几乎没有参考资料，但是官网文档写的非常详细了。这篇文档使用的是 v3 版本，下面介绍使用 Maven 来安装:

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.statemachine/spring-statemachine-starter -->
<dependency>
    <groupId>org.springframework.statemachine</groupId>
    <artifactId>spring-statemachine-starter</artifactId>
</dependency>
```

### 业务说明 {id="business"}

我们以游戏场景为例，用户从开始游戏到游戏结束, 状态机图如下:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/UNC5AJDsYgo2PgcOVR8D.png" alt="state-machine-draw" />

很简单的状态的单向流转。其实在 Spring State Machine 中，状态的流转并不是难点，难点在于持久化。后文详细说明。

### 状态以及事件的定义 {id="defined"}

Spring State Machine
支持通过字符串和枚举两种方式定义状态以及事件。这里采用枚举的方式，相对于字符串更方便维护。首先我们需要定义状态以及事件的枚举类:

```Java
@Getter
@AllArgsConstructor
public enum GameStatusEnum {

    /**
     * 从未开始
     */
    NEVER_START("NEVER_START"),

    /**
     * 已开始
     */
    STARTED("STARTED"),

    /**
     * 永远结束(通关或者超时，且游戏不可以重玩)
     */
    END("END"),
    ;

    @EnumValue
    private final String value;
}
```

接着定义事件的枚举:

```Java
@Getter
@AllArgsConstructor
public enum PromotionGameEventEnum {
    STARTED, COMPLETED;
}
```

我们定义了两个事件， STARTED 表示游戏开始，此时状态将会从NEVER_START 流转到 STARTED 。COMPLETED 表示游戏结束，此时状态将会从
STARTED 流转到 END 。

### 状态机的配置 {id="configure"}

首先我们需要创建一个配置类:

```Java
@Configuration
@AllArgsConstructor
@EnableStateMachineFactory
public class GameStatemachineConfiguration 
    extends EnumStateMachineConfigurerAdapter<GameStatusEnum, GameEventEnum> {

    @Override
    public void configure(StateMachineConfigurationConfigurer<PromotionUserGameStatusEnum, PromotionGameEventEnum> config) throws Exception {
        // 此处为状态机ID,其他情况下为状态机实例ID，注意区分
        config.withConfiguration().machineId("GAME")
                .listener(new GameStateMachineListener())
                .autoStartup(true);
    }
}
```

我们定义了一个配置类，名为 `GameStatemachineConfiguration` ，并且继承了 `EnumStateMachineConfigurerAdapter` ，接受两个泛型参数,
分别是状态以及事件的枚举。

- `@EnableStateMachineFactory` 这个注解表示启用状态机工厂，当我们需要在代码中创建多个状态机实例的时候，就需要使用这个注解，比如根据不同的游戏创建不同的状态机、根据不同的订单创建不同的状态机。
- `machineId` 表示配置状态机的ID。
- `autoStartup` 表示状态机随着应用启动而自动启用。
- `listener` 表示指定一个全局监听器，注册状态机的生命周期中的各个事件 `Hander`。

<note>
需要强烈说明的是，在 Spring State Machine 中，以及平时的表述中，不怎么区分状态机和状态机的示例的概念，这个在编码中极容易造成错用。这就像如同 OOP 中类和对象的概念一般。
</note>

比如我的应用中有订单的状态、游戏的状态，这时候需要通过状态机来区分。而当需要根据订单 ID 或者游戏 ID 创建状态机的时候，指的都是状态机实例。

### 状态流转的配置 {id="transfer"}

配置了状态机之后，我们需要对状态机中状态的流转进行配置，如下:

```Java
/**
 * 配置状态机状态转换
 *
 * @param transitions the {@link StateMachineTransitionConfigurer}
 */
@Override
public void configure(StateMachineTransitionConfigurer<PromotionUserGameStatusEnum, PromotionGameEventEnum> transitions) throws Exception {
    transitions.withExternal()
            .source(GameStatusEnum.NEVER_START).target(GameStatusEnum.STARTED)
            .event(GameEventEnum.STARTED)
            .action(context -> { System.out.println("Never -> Started") })
            
            .and().withExternal()
            .source(GameStatusEnum.STARTED).target(GameStatusEnum.COMPLATED)
            .event(GameEventEnum.COMPLATED);
}
```

通过 `source` 和 `target` 我们定义了一组状态的流转，第 9 行，定义了状态从 `NEVER_START` 流转到 `STARTED`
，通过 `event(GameEventEnum.STARTED)` 事件触发，并且在状态流转之后执行 `action` 操作，输出一行打印。

这几行配置可以清楚的展示使用状态机的优势：

- 状态配置化，原先状态的配置都是在业务逻辑中，需要深入业务逻辑才能理清楚状态的变化，而状态的变化又是极容易出错的地方。

- 状态可扩展，通过配置，我们可以快速、安全的配置状态的流转、状态流转后的事件。

上面的代码展示了当我们开始游戏之后，触发了 `STARTED` 事件，然后向状态机发送了一个 `Message` ,这个 `Message` 中包含了必要的参数，游戏
ID 和用户 ID。

需要着重说明的是下面这两个方法:

- `acquireStateMachine` 这个方法表示恢复状态实例的状态，这个状态可能是保存在内存中，保存在数据库中，这个在后续的持久化中再说。
- `releaseStateMachine` 这个方法会调用状态实例的持久化，也就是将状态持久化道内存或者数据库中。

### 状态机的持久化 {id="persistence"}

上面以及提到了状态机实例的状态需要持久化。官方给出了 JPA 的实现。这个就不说了，可以参考官方文档。这里给出在 MyBatis 、或者其他
ORM 中的实现。持久化是最容易出错，也是最难排查错误的地方。

配置持久化，上文我们已经配置了状态机，然后在上文的配置中，加入对持久化的配置:

```Java
private final StateMachineRuntimePersister<GameStatusEnum, GameEventEnum, String>
     stateMachinePersist;
@Override
public void configure(StateMachineConfigurationConfigurer<PromotionUserGameStatusEnum, PromotionGameEventEnum> config) throws Exception {
    config.withPersistence().runtimePersister(stateMachinePersist);
}
```

其中 `stateMachinePersist` 的代码如下:

```Java
@Configuration
@AllArgsConstructor
public class MyBatisPersistingConfiguration {

    private final MyBatisStateMachineRepository<GameStatusEnum, GameEventEnum, String> myBatisStateMachineRepository;

    @Bean
    public StateMachineRuntimePersister<GameStatusEnum, GameEventEnum, String> stateMachineRuntimePersister() {
        return new MyBatisPersistingStateMachineInterceptor<>(myBatisStateMachineRepository);
    }

    @Bean
    public StateMachineService<UserGameStatusEnum, GameEventEnum> stateMachineService(
            StateMachineFactory<UserGameStatusEnum, GameEventEnum> stateMachineFactory,
            StateMachineRuntimePersister<UserGameStatusEnum, GameEventEnum, String> stateMachineRuntimePersister) {
        return new DefaultStateMachineService<>(stateMachineFactory, stateMachineRuntimePersister);
    }
}
```

它继承了StateMachinePersist 的接口。另外我们需要实现一个拦截器，当状态流转的时候，会调用改拦截器的 read 方法来恢复状态、调用write
方法来持久化状态。

```Java
public class MyBatisPersistingStateMachineInterceptor<S, E, T> extends AbstractPersistingStateMachineInterceptor<S, E, T> implements StateMachineRuntimePersister<S, E, T> {

    private final MyBatisStateMachineRepository<S, E, T> persist;

    public MyBatisPersistingStateMachineInterceptor(MyBatisStateMachineRepository<S, E, T> persist) {
        this.persist = persist;
    }

    @Override
    public void write(StateMachineContext<S, E> context, T contextObj) throws Exception {
        persist.write(context, contextObj);
    }

    @Override
    public StateMachineContext<S, E> read(T contextObj) throws Exception {
        return persist.read(contextObj);
    }

    @Override
    public StateMachineInterceptor<S, E> getInterceptor() {
        return this;
    }
}
```

它接收一个Persist 的实现，就是 `MyBatisStateMachineRepository` 的代码如下:

```Java
public interface MyBatisStateMachineRepository<S, E, T> extends StateMachinePersist<S, E, T> {
}
```

最后我们实现 MyBatis 持久化:

```Java
@Component
@AllArgsConstructor
public class InDatabaseUserStateMachinePersist implements MyBatisStateMachineRepository<PromotionUserGameStatusEnum, PromotionGameEventEnum, String> {

    /**
     * 写入状态机实例上下文
     *
     * @param context    状态机实例上下文
     * @param contextObj 活动ID_用户ID
     */
    @Override
    public void write(StateMachineContext<GameStatusEnum, EventEnum> context, String contextObj) {
        // 通过 MyBatis 持久化到数据库
        // 需要注意，需要将 context 写入到数据库...
    }

    /**
     * 读取状态机实例上下文
     *
     * @param contextObj 活动ID_用户ID
     * @return 状态机实例上下文
     */
    @Override
    public StateMachineContext<GameStatusEnum, GameEventEnum> read(String contextObj) {
        // 需要从数据库中恢复 context 上下文
    }

}
```

## 难以排查的问题 {id="problems"}

使用 SSM，最多的时间用在了排查问题上。不像是自己在业务代码中维护状态，可以通过断点一步步排查出问题。SSM
优雅的背后是深度的抽象，这也导致了部分问题难以排查。

### 使用 Listener

配置 Listener 的代码如下,不管用不用到 Listener，都要配置好并输出每个状态流转的事件信息，在开发初期用以排查问题:

```Java
@Override
public void configure(StateMachineConfigurationConfigurer<PromotionUserGameStatusEnum, PromotionGameEventEnum> config) throws Exception {
    config.withConfiguration().machineId("USER")
            .listener(new GameStateMachineListener());
}
```

然后编写名为 `GameStateMachineListener` 的类，完整代码如下:
```Java
public class GameStateMachineListener extends StateMachineListenerAdapter<PromotionUserGameStatusEnum, PromotionGameEventEnum> {

    @Override
    public void stateChanged(State<PromotionUserGameStatusEnum, PromotionGameEventEnum> from, State<PromotionUserGameStatusEnum, PromotionGameEventEnum> to) {
        System.out.println("状态变更: " + from + " -> " + to);
    }

    @Override
    public void stateEntered(State<PromotionUserGameStatusEnum, PromotionGameEventEnum> state) {
        System.out.println("状态进入: " + state);
    }

    @Override
    public void stateExited(State<PromotionUserGameStatusEnum, PromotionGameEventEnum> state) {
        System.out.println("状态退出: " + state);
    }

    @Override
    public void transition(Transition<PromotionUserGameStatusEnum, PromotionGameEventEnum> transition) {
        System.out.println("状态转换: " + transition);
    }

    @Override
    public void transitionStarted(Transition<PromotionUserGameStatusEnum, PromotionGameEventEnum> transition) {
        System.out.println("状态转换开始: " + transition);
    }

    @Override
    public void transitionEnded(Transition<PromotionUserGameStatusEnum, PromotionGameEventEnum> transition) {
        System.out.println("状态转换结束: " + transition);
    }

    @Override
    public void stateMachineStarted(StateMachine<PromotionUserGameStatusEnum, PromotionGameEventEnum> stateMachine) {
        System.out.println("状态机启动: " + stateMachine);
    }

    @Override
    public void stateMachineStopped(StateMachine<PromotionUserGameStatusEnum, PromotionGameEventEnum> stateMachine) {
        System.out.println("状态机停止: " + stateMachine);
    }

    @Override
    public void eventNotAccepted(Message<PromotionGameEventEnum> event) {
        System.out.println("事件不被接受: " + event);
    }

    @Override
    public void extendedStateChanged(Object key, Object value) {
        System.out.println("扩展状态变更: " + key + " -> " + value);
    }

    @Override
    public void stateMachineError(StateMachine<PromotionUserGameStatusEnum, PromotionGameEventEnum> stateMachine, Exception exception) {
        System.out.println("状态机异常: " + stateMachine + " -> " + exception.getMessage());
    }

    @Override
    public void stateContext(StateContext<PromotionUserGameStatusEnum, PromotionGameEventEnum> stateContext) {
        System.out.println("状态上下文: " + stateContext);
    }
}
```

### 不被接受的事件 {id="Not-Accepted"}

在上面的事件中，要格外关注一个名为 `eventNotAccepted` 的事件，因为一旦触发这个事件，如果不知所以然就非常难排查，网上也没有更多资料可以参考。
可以从如下几个方面来排查:

* source 状态是否和预期一致
* guard 是否通过
* action 中是否存在异常

其中第三点最难排查，因为 SSM 在 action 出现异常的时候，也不会将异常信息抛出，而是不会执行状态的流转，触发 `eventNotAccepted` 事件。需要通过在源码中
打断点的方式，深入十几层的调用栈才能找到异常。

## 总结 {id="summary"}

这篇文档描述了什么是状态机，如何应用 Spring State Machine
到项目中去。遇到问题，多看官方文档，当发现解决不了的时候，总是什么地方自己理解错了，或者没有理解。这是我在应用状态机的过程中，最大的收获。