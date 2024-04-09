# DevSecOps

这篇文档尚未更新完成，陆陆续续更新中...

## 持续集成 {id="continuous-integration"}

什么是持续集成（Continuous Integration，CI）？

**它是一种软件开发实践，鼓励开发人员频繁地（一天多次）将代码集成到代码仓库中，确保每次合并都不会破坏已有功能，并自动化执行构建和测试**。这样做的目的是快速发现并修复集成错误，提升软件质量，加快发布速度、减少发布风险。

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/edh5ky9taI3YmciIeWce.png)

## 持续交付 {id="continuous-delivery"}

什么是持续交付（Continuous Delivery，CD）？

它是持续集成的延伸，**包含了测试、配置、发布，以及自动化部署，保障软件随时都处于可发布的状态** 。是敏捷开发和 DevOps 文化的关键组成部分。

那么持续交付和持续集成的区别是什么？

**持续部署其实包含了持续集成，但是两者的关注点不同。持续集成关注的是代码频繁合并到仓库，而持续部署则关注通过持续集成让项目代码时刻处于可发布的状态**。

