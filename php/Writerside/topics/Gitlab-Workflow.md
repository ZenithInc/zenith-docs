# Gitlab 工作流

这篇文档描述了使用 Gitlab 的工作流，从创建项目到 CI/CD，完整的演示一个项目从开始编码到部署。

## 前置准备

本文假设读者已经熟悉 Linux、Shell、PHP、Git 以及 Laravel 框架。对于 Gitlab 可以没有基础，本文将详细介绍。

文本使用的是 Linux 的发行版之一: Rocky Linux 9.2 版本、PHP 8.3 、Laravel 10.x 版本。

本文实验，采用了三台服务器:

* 配置 8C16M 1 台： 作为 Gitlab 服务器，4C4M 可以支持 500 用户，8C8M 可以支持 1000 用户。
* 配置 4C4M 2 台: 作为应用服务器，部署 Laravel 应用，对外提供服务。

## Gitlab 简介 {id="what-is-gitlab"}

Gitlab 是一个 DevSecOps 平台，也就是集合 Dev（软件开发） + Sec（软件安全） + Ops（软件运维）团队之间相互协同的文化、实践和工具的一个平台。它具备了如下特征:

* 在软件的设计、编码、测试和部署过程中都优先考虑安全。
* 自动化安全测试。
* 跨团队协作。
* 持续学习和适应。

## Gitlab 安装 {id="install"}

接下来，我们就可以开始动手在服务器上安装 Gitlab 了。具体的指令如下, 安装依赖:
```Shell
sudo dnf install -y curl policycoreutils openssh-server perl
```

安装发送邮件需要的依赖:
```Shell
sudo dnf install -y postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

接着安装 Gitlab package repository:
```Shell
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```
如果你看到终端输出`The repository is setup!` 类似输出，则表明已经 Gitlab package repository 安装成功了。

最后可以安装 Gitlab-EE 最新版了:

```Shell
sudo EXTERNAL_URL="https://gitlab.hacking.icu" dnf install -y gitlab-ee
```
看到如下输出，则表明已经安装成功:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/g8ScrKle0daxzWWsTRRL.png)

接下来，我们就可以通过浏览器访问 Gitlab 了:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/uB225cYraSEokZsRacm5.png)

> 初始化的 root 用户的密码可以通过 `cat /etc/gitlab/initial_root_password` 命令查看，这个文件会在 24 小时之后被自动删除。

## 初始化配置 {id="configure"}

登录之后，我们会看到如下界面。最上方有一个警告，如下图所示:

![](http://file-linker.oss-cn-hangzhou.aliyuncs.com/R53b2ruTynIDvN0X9V9b.png)

<warning>
默认情况下，允许任何访问这个网站的人注册，这可能由安全风险。可以在单击 `Deactivate` 按钮禁用用户登录。
</warning>
