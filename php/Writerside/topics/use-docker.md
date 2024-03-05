# Docker环境搭建

这篇文档描述了如何使用 Docker 初始化 Laravel 的开发环境。这是一种简单、现代的方式，希望大家都能够采用这种方式。这种方式统一了线上线下环境，有效避免了因为环境的不一致导致的问题。

## 前提条件 {id="pre-condition"}

需要说明的说，这不是一篇讲解 Docker 的文章，我们假设读者已经掌握了 Docker、Docker Composer 相关的知识，并且已经完成了本地 Docker 的安装。

另外，这篇文档使用了当前比较新的技术版本，Laravel 采用 10 的版本、PHP 使用的是 8.2 版本。

## 初始化项目 {id="init"}

首先，我们需要初始化一个项目，将 Laravel 的框架代码从远程仓库下载到本地，并框架所需的依赖。这需要使用 Composer，这里使用 Docker 来完成:
```Shell
docker run -v .:/app/ composer:latest create-project laravel/laravel app
```

## 编写 Dockerfile {id="dockerfile"}

接着，我们需要编写一个 Dockerfile 文件，在项目的根目录下，内容如下:
```Docker
FROM composer:latest AS composer
WORKDIR /app
COPY ./composer.json ./composer.lock ./
RUN composer install --no-dev --no-scripts

FROM php:8.2-fpm
RUN apt-get update && apt-get install -y libpng-dev libonig-dev libxml2-dev && \
    docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
WORKDIR /var/www
COPY . /var/www
COPY --from=composer /app/vendor/ /var/www/vendor/
EXPOSE 9000
CMD ["php-fpm"]
```
这个 Dockerfile 分为两个阶段。首先是 composer 阶段，从 Composer 镜像开始，拷贝 composer.json 和 composer.lock 文件，然后安装依赖。然后是 php-fpm 阶段，从 PHP 镜像开始，安装必要的 PHP 扩展，并从 composer 阶段拷贝已经安装好的依赖进来。

这样做的好处是，最后的镜像将不会包含用于安装依赖的额外文件和工具，只包含运行应用所需的最小集，因此可以减小最后镜像的体积。

## 编写 docker-compose.yml {id="docker-compose"}

接着，我们在根目录下编写名为 `docker-compose.yml` 的容器编排文件:
```yaml
version: '3.1'

services:

    php:
        build:
            context: .
            dockerfile: Dockerfile
        volumes:
            - ./:/var/www
        ports:
            - "9000:9000"

    mysql:
        image: mysql:8.0
        command: --default-authentication-plugin=mysql_native_password --secure-file-priv=""
        volumes:
            - db_data:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: root
            MYSQL_DATABASE: app
        ports:
            - "33060:3306"

    nginx:
        image: nginx:latest
        ports:
            - "8080:80"
        volumes:
            - ./:/var/www
            - ./nginx.conf:/etc/nginx/conf.d/default.conf

volumes:
    db_data:
```
当中，我们启动了三个服务，分别是 `php`、`nginx` 以及 `mysql`，这样我们一个基本的应用就完成了。

## 启动服务 {id="start-service"}

最后，我们可以启动服务了，命令如下:
```Shell
$ docker-compose up
```
如果没有异常的话，当我们访问`http://localhost:8080` 会出现如下界面:

![image](http://file-linker.oss-cn-hangzhou.aliyuncs.com/IlpiwHSmijELjaVNAwEI.png)

<warning>
如果出现，`storage` 等目录下没有权限，无法写入日志。可以进入到容器中，执行 `chown -R www-data:www-data storage/` 命令手动赋予权限。
</warning>

## 总结 {id="summary"}

这篇文档我们介绍了如何使用 docker 以及 docker-compose 来快速启动 Laravel 项目的开发环境。Laravel 官方开发了 Laravel Herd ，用来管理本地开发环境。目前只支持 MacOS, Windows 的支持需要等到 2024 年 3 月底。到时候可以再写一篇来讲讲。