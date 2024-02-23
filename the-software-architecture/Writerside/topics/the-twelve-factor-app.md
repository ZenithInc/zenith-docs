# 构建SaaS应用的12个要素 

Heroku 是一个云平台，它的开发者提出了构建 SaaS 应用的 12 个要素。这些指引和具体的技术无关。

你可以访问 [The Twelve Factor App](https://12factor.net/)，看到具体的内容,提供了十几种语言的翻译，也包括中文。这些这些摘抄自这个网站:


<deflist collapsible="false">
<def title="1. 基准代码">
一份基准代码，多份部署
</def>

<def title="2. 依赖">
显式声明依赖关系
</def>

<def title="3. 配置">
在环境中存储配置
</def>

<def title="4. 后端服务">
把后端服务当作附加资源
</def>

<def title="5. 构建，发布，运行">
严格分离构建和运行
</def>

<def title="6. 进程">
以一个或多个无状态进程运行应用
</def>

<def title="7. 端口绑定">
通过端口绑定提供服务
</def>

<def title="8. 并发">
通过进程模型进行扩展
</def>

<def title="9. 易处理">
快速启动和优雅终止可最大化健壮性
</def>

<def title="10. 开发环境和线上环境等价">
尽可能的保持开发，预发布，线上环境相同
</def>

<def title="11. 日志">
把日志当作事件流
</def>

<def title="12. 管理进程">
后台管理任务当作一次性进程运行
</def>
</deflist>