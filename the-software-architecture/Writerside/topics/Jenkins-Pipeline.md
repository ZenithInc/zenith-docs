# Jenkins Pipeline

这篇文档介绍了 Jenkins Pipeline 的相关内容，但是对于 Pipeline 以外其他的 Jenkins 内容就不多做说明。

## 什么是 Pipeline {id="what-is-pipeline"}

在 《持续交付-发布可靠软件的系统方法》一书中提到，部署流水线（Deployment pipeline）指的是从软件版本仓库到交付用户这个过程中的自动化表现形式。

对于 Jenkins 来说，就是编写 Jenkinsfile 存放在项目目录中，当中定义了一系列的自动化步骤（Build Steps），当提交代码的时候会按照当中的定义自动执行相应的任务。

> 在 Jenkins 中，有两种方式声明 Pipeline。一种是通过 UI 手动创建；一种是 pipeline as code，即上文提到的编写 Jenkinsfile。本文基于第二种，可以更好的版本化、更好的协作、更好的重用性。

**采用 pipeline as code 的做法促进了“基础设施即代码”（Infrastructure as code）和“配置即代码”（Configuration as code）的实践** ，通过这种方式可以减少环境之间的差异，提高开发和运维的效率。

在 Pipeline （Jenkinsfile）的编写上，也有两种选择，一种是编写 Groovy 脚本，一种是采用声明式语法。本文采用第二种，虽然第一种更加灵活、可扩展，但是第二种更结构化，并且更符合我们的阅读习惯、也更简单。

## Pipeline 的基础结构 {id="pipeline-structure"}

下面的配置展示了一个 Pipeline 的一个基础的结构:
```Groovy
pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```
对于上面示例中的一些关键词进行解释:

* `pipeline`: 代表了整条流水线按，声明了流水线的配置、任务。
* `stages` 和 `stage`: 流水线中的阶段的声明，一个 `stages` 中包含多个 `stage`，好比把大象装进冰箱需要几个 `stage` ？
* `steps`: 每个阶段需要做的步骤，比如 `echo` 就是一个步骤，输出 `Hello World`
* `agent`: 流水线的执行位置，比如物理机、虚拟机、Docker 容器等。`all` 表示在任何可用的节点上执行。

## Post {id="post"}

在 Jenkins Pipeline 中，`post` 部分允许你定义一组在 Pipeline 或阶段（Stage）完成后执行的步骤，无论这个 Pipeline 或阶段是成功了、失败了，还是遇到了其他的结束条件。通过使用不同的条件块，你可以针对不同的完成状态执行特定的操作，例如发送通知、清理工作空间或者进行部署。

下表概述了 `post` 部分中可用的几种完成状态及其描述：

| 状态           | 描述                                                           |
|--------------|--------------------------------------------------------------|
| `always`     | 不管当前完成状态是什么，都执行。                                             |
| `success`    | 只有当 Pipeline 或阶段成功完成时执行。                                     |
| `failure`    | 只有当 Pipeline 或阶段失败时执行。                                       |
| `unstable`   | 只有当 Pipeline 或阶段的结果被标记为不稳定时执行。这通常意味着构建过程中有测试失败。              |
| `changed`    | 只有当当前 Pipeline 或阶段的状态与上一次构建的状态不同时执行。这对于识别状态变化（例如从失败到成功）非常有用。 |
| `fixed`      | 只有当当前 Pipeline 或阶段的状态由“失败”变为“成功”时执行。                         |
| `regression` | 只有当当前 Pipeline 或阶段的状态由“成功”变为“失败”时执行。                         |
| `aborted`    | 只有当 Pipeline 或阶段被中断时执行。                                      |
| `cleanup`    | 在 Pipeline 的最后执行，用于清理工作，不管结果如何都会执行。                          |

每个状态块内部可以包含一系列的步骤，这些步骤可以是发送通知、执行脚本或其他任何可用的 Jenkins 步骤。示例如下:

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello, World!'
            }
        }
    }
    post {
        always {
            echo 'I will always say Hello, World!'
        }
        success {
            echo 'Hurray, I succeeded!'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Something changed!'
        }
    }
}
```

## 指令 {id="directive"}

Jenkins Pipeline 提供了多种指令（Directives），用于定义和控制 Pipeline 的行为和结构。下面是一些常见指令的列表和它们的描述：

| 指令            | 描述                                                       |
|---------------|----------------------------------------------------------|
| `agent`       | 定义 Pipeline 或阶段（Stage）的执行环境。                             |
| `post`        | 定义一组在 Pipeline 或阶段完成后执行的步骤，根据不同的完成状态执行特定的操作。             |
| `stages`      | 定义 Pipeline 中所有阶段（Stages）的容器。这是执行实际工作任务的地方。              |
| `stage`       | 定义 Pipeline 中的一个阶段，表示构建过程的一个逻辑部分，例如构建、测试和部署。             |
| `steps`       | 定义在一个阶段（Stage）或在 `script` 指令中要执行的一系列步骤（Commands）。        |
| `environment` | 用于设置环境变量。可以在 Pipeline 的顶层或特定的 `stage` 中定义，为后续步骤提供环境变量。   |
| `parameters`  | 定义 Pipeline 运行时需要的参数。这些参数可以在构建时由用户提供。                    |
| `triggers`    | 定义自动触发 Pipeline 执行的条件，例如定时执行或代码变更。                       |
| `options`     | 提供 Pipeline 或特定阶段的额外设置，例如超时时间、重试次数等。                     |
| `tools`       | 指定构建过程中需要使用的工具及其版本，例如 `jdk` 或 `maven`。                   |
| `input`       | 定义一个等待用户输入的步骤，可以用于手动确认继续执行或提供参数值。                        |
| `when`        | 控制阶段（Stage）是否执行的条件。基于各种条件判断，例如分支名称或是否变更了特定的文件。           |
| `script`      | 允许在声明式 Pipeline 中嵌入 Groovy 脚本代码，提供了更大的灵活性和控制能力。          |
| `parallel`    | 允许在同一阶段（Stage）中并行执行多个分支，每个分支都有自己的步骤（Steps）和环境。           |
| `timeout`     | 在 `options` 或特定阶段中设置超时限制，如果指定时间内未完成，则终止 Pipeline 或阶段的执行。 |

这些指令提供了构建复杂的 CI/CD Pipeline 的构建块，通过它们可以精细地控制构建过程、环境设置、执行条件等各个方面。

## 内置步骤 {id="built-in-steps"}

Jenkins Pipeline 提供了许多内置步骤（Built-in Steps），这些步骤可以在 `steps` 部分直接调用，用于执行常见的构建、测试和部署任务。以下是一些常用的内置步骤及其描述：

| 步骤名称              | 描述                                                           |
|-------------------|--------------------------------------------------------------|
| `echo`            | 打印消息到构建日志。                                                   |
| `deleteDir()`     | 删除当前工作目录下的所有文件和子目录，通常用于清理工作空间。                               |
| `sh`              | 在 Linux 或 Unix 环境下执行 Shell 脚本命令。                             |
| `bat`             | 在 Windows 环境下执行批处理命令。                                        |
| `checkout`        | 检出源码。一般用于从版本控制系统中获取项目代码。                                     |
| `git`             | 从 Git 仓库中检出代码，是 `checkout` 的一个特化形式，专用于 Git。                  |
| `svn`             | 从 Subversion 仓库中检出代码，是 `checkout` 的一个特化形式，专用于 Subversion。    |
| `stage`           | 定义一个阶段的开始，用于组织和可视化 Pipeline 的不同部分。                           |
| `parallel`        | 并行执行多个分支，每个分支都是并行的。                                          |
| `node`            | 分配一个 Jenkins 节点来执行包含在内的步骤，可以指定标签来选择特定的节点。                    |
| `env`             | 访问或设置环境变量。                                                   |
| `withEnv`         | 临时设置环境变量，只在其块中有效。                                            |
| `pipeline`        | 定义 Pipeline 的开始，是所有 Jenkinsfile 的根级元素。                       |
| `input`           | 要求用户输入，以决定是否继续执行 Pipeline。                                   |
| `timeout`         | 设置一个时间限制，如果指定时间内步骤未完成，则失败。                                   |
| `try/catch`       | 提供异常处理机制，`try/catch` 是 Groovy 语言的一部分，但在 Pipeline 脚本中常用于错误处理。 |
| `sleep`           | 暂停当前 Pipeline 的执行指定的时间长度。                                    |
| `withCredentials` | 安全地使用凭证（如密码、密钥等）而不将其暴露于脚本日志中。                                |
| `build`           | 触发另一个 Jenkins 项目的构建，并可选地等待其完成。                               |
| `mail`            | 发送电子邮件通知。                                                    |

这些步骤提供了在 Jenkins Pipeline 中实现各种自动化任务的基本构建块，从简单的打印日志到执行脚本，再到复杂的并行执行和错误处理。

## 参考 {id="references"}

1. [Jenkinsfile 官方文档](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/)
2. [电子工业出版社 - 《Jenkins 2.x 实践指南》](https://book.douban.com/subject/33407581/)
