# API 设计

这篇文档介绍了什么是 API、API 的分类、作用，以及如何设计 API。

## 什么是 API {id="what-is-api"}

API是应用程序编程接口（Application Programming Interface）的缩写。它是一组定义了软件组件之间交互的规范和约定。API定义了如何请求或响应数据、访问功能或服务，并规定了数据的格式和传输方式。通过API，不同的软件组件、不同的系统之间可以进行交互和通信。

API可以用于不同的目的，如访问操作系统功能、访问硬件设备、访问第三方服务、实现不同软件模块之间的通信等。API可以以不同的形式存在，如函数库、类库、操作系统API、Web服务API等。

使用API可以实现软件的模块化和可重用性，提高开发效率和灵活性。同时，API也提供了一种标准化的接口，使得不同的软件组件可以更容易地集成和共享数据和功能。

## API 的分类 {id="categories"}

<code-block lang="plantuml">
@startmindmap
* API 分类
    * 根据使用者不同
        * API
        * SPI(Service Provider Interface)
    * 根据使用目的不同
        * 对内接口
        * 对外接口
    * 根据用途场景不同
        * 通用API
        * 框架级API
        * 应用级API
@endmindmap
</code-block>

## API 的作用 {id="use-case"}

<code-block lang="plantuml">
@startmindmap
* API 的作用
    * 作为系统提供功能的对内或对外的契约
    * 封装系统、程序模块化，使得系统更好的进行依赖选择
    * 分离使用者和具体实现
    * 面向抽象编程
    * 提供系统的可维护性和可扩展性
@endmindmap
</code-block>

## API 如何设计 {id="how-to-design"}

<code-block lang="plantuml">
@startmindmap
* API 如何设计
    * 1. 用例驱动设计
        * 按照业务功能需要进行初步设计
        * 根据功能点列表翻译成API接口
    * 2. 对初步设计的API进行综合权衡和调整
        * 类似功能合并
        * 功能过大进行拆分
        * 考虑业务内聚、职责单一
        * 考虑接口设计粒度粗细
        * 考虑参数列表的合理性，能少则不多
        * 返回的参数的数量和易用性
        * 向前兼容性、向后扩展性
    * 3. 对照业务场景进行API的功能走查
        * 是否满足页面展示需要
        * 需要考虑页面中隐含的数据回显以及连带操作
        * 页面提交的功能是否满足
    * 4. 逐步改善、增量改进
        * 从API实现来验证设计并完善
@endmindmap
</code-block>

## API 设计的建议 {id="api-design-advice"}

<code-block lang="plantuml">
@startmindmap
+ 设计API的最佳实践
++ 团队设计应该保持一致性
++ 隐藏内部实现细节
++ RPC 接口使用粗力度
++ 不同目标用户不同的接口
++ API 设计永不完美，需不断重构
-- 好好取名字
-- 少就是多，最少需要原则
-- 对外接口使用粗粒度
-- 使用对象进行输入输出
-- 符合以往经验以及使用习惯
@endmindmap
</code-block>

## 参考资料

如果你设计一套对外的 API，那么建议参考 [《Google API 设计指南》](http://file-linker.oss-cn-hangzhou.aliyuncs.com/oHOx84Lb9BYQdkX40PpG.png)。

也可以参考 [《Microsoft API 设计最佳实践》](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)。