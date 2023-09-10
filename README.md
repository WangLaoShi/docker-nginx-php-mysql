# Nginx PHP MySQL Docker

[![Build Status](https://travis-ci.org/nanoninja/docker-nginx-php-mysql.svg?branch=master)](https://travis-ci.org/nanoninja/docker-nginx-php-mysql) 
[![GitHub version](https://badge.fury.io/gh/nanoninja%2Fdocker-nginx-php-mysql.svg)](https://badge.fury.io/gh/nanoninja%2Fdocker-nginx-php-mysql)

运行 Nginx、PHP-FPM、Composer、MySQL 和 PHPMyAdmin 的 Docker。

## 概述

1. [安装先决条件](#install-prerequisites)

    在安装项目之前，请确保满足以下先决条件。

2. [克隆项目](#clone-the-project)

    我们将从其在 GitHub 上的存储库下载代码。

3. [使用 SSL 证书配置 Nginx](#configure-nginx-with-ssl-certificates) [`可选`]

    在运行服务器之前，我们将为 nginx 生成和配置 SSL 证书。

4. [Configure Xdebug](#configure-xdebug) [`可选`]

    我们将为 IDE（ PHPStorm 或 Netbeans ）配置 XDebug。

5. [运行应用程序](#run-the-application)

    至此，我们将准备好所有项目部分。

6. [使用 Makefile](#use-makefile) [`可选`]

    开发时，您可以使用 `Makefile` 进行循环操作。

7. [使用 Docker 命令](#use-docker-commands)

    运行时，您可以使用 docker 命令进行循环操作。

___

## 安装先决条件

要在不使用 **sudo** 的情况下运行 **docker** 命令，您必须将 **docker** 组添加到 **your-user**：

```
sudo usermod -aG docker your-user
```

目前，这个项目主要是为 Unix `(Linux/MacOS)` 创建的。 也许它可以在 Windows 上运行。

所有必需品都应该可用于您的分发。 最重要的是：


* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

通过输入以下命令检查是否已安装 `docker-compose`：

```sh
which docker-compose
```

检查 Docker Compose 兼容性：

* [Compose 文件版本 3 参考](https://docs.docker.com/compose/compose-file/)

以下是可选的，但会让生活更愉快：

```sh
which make
```

在 Ubuntu 和 Debian 上，这些在 meta-package build-essential 中可用。 在其他发行版上，您可能需要单独安装 GNU C++ 编译器。

```sh
sudo apt install build-essential
```

### 要使用的镜像

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL](https://hub.docker.com/_/mysql/)
* [PHP-FPM](https://hub.docker.com/r/nanoninja/php-fpm/)
* [Composer](https://hub.docker.com/_/composer/)
* [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
* [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

安装第三方 Web 服务器（如 MySQL 或 Nginx）时应小心。

该项目使用以下端口：

| Server     | Port |
|------------|------|
| MySQL      | 8989 |
| PHPMyAdmin | 8080 |
| Nginx      | 8000 |
| Nginx SSL  | 3000 |

___

## 克隆项目

要安装 [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)，请按照说明下载并安装：

```sh
git clone https://github.com/WangLaoShi/docker-nginx-php-mysql.git
```

转到项目目录：

```sh
cd docker-nginx-php-mysql
```

### 项目代码树形结构

```sh
.
├── Makefile
├── README.md
├── data
│   └── db
│       ├── dumps
│       └── mysql
├── doc
├── docker-compose.yml
├── etc
│   ├── nginx
│   │   ├── default.conf
│   │   └── default.template.conf
│   ├── php
│   │   └── php.ini
│   └── ssl
└── web
    ├── app
    │   ├── composer.json.dist
    │   ├── phpunit.xml.dist
    │   ├── src
    │   │   └── Foo.php
    │   └── test
    │       ├── FooTest.php
    │       └── bootstrap.php
    └── public
        └── index.php
```

___

## 使用 SSL 证书配置 Nginx

您可以通过编辑 `.env` 文件来更改主机名。

如果您修改了主机名，请不要忘记将其添加到 `/etc/hosts` 文件中。

1. 生成 SSL 证书

    ```sh
    source .env && docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
    ```

2. 配置 Nginx

    - 不要修改`etc/nginx/default.conf`文件，它会被`etc/nginx/default.template.conf`覆盖

    - 编辑 nginx 文件 `etc/nginx/default.template.conf` 并取消注释 SSL 服务器部分：

    ```sh
    # server {
    #     server_name ${NGINX_HOST};
    #
    #     listen 443 ssl;
    #     fastcgi_param HTTPS on;
    #     ...
    # }
    ```

___

## 配置 Xdebug

如果您使用 [PHPStorm](https://www.jetbrains.com/phpstorm/) 或 [Netbeans](https://netbeans.org/) 以外的其他 IDE，请转到 [远程调试](https:// xdebug.org/docs/remote) Xdebug 文档部分。

为了更好地将 Docker 集成到 PHPStorm，请使用 [文档](https://github.com/WangLaoShi/docker-nginx-php-mysql/blob/master/doc/phpstorm-macosx.md)。

1. 获取您自己的本地 IP 地址：

    ```sh
    sudo ifconfig
    ```

2. 编辑 php 文件 `etc/php/php.ini` 并根据需要注释或取消注释配置。

3. 使用您的 IP 设置 `remote_host` 参数：

    ```sh
    xdebug.remote_host=192.168.0.1 # your IP
    ```
___

## 运行应用程序

1. 复制 composer 配置文件： 

    ```sh
    cp web/app/composer.json.dist web/app/composer.json
    ```

2. 启动应用程序：

    ```sh
    docker-compose up -d
    ```

    **请稍等，这可能需要几分钟...**

    ```sh
    docker-compose logs -f # Follow log output
    ```

3. 打开您喜欢的浏览器：

    * [http://localhost:8000](http://localhost:8000/)
    * [https://localhost:3000](https://localhost:3000/) ([HTTPS](#configure-nginx-with-ssl-certificates) 默认情况下未配置)
    * [http://localhost:8080](http://localhost:8080/) PHPMyAdmin (username: dev, password: dev)

4. 停止并清除服务

    ```sh
    docker-compose down -v
    ```

___

## 使用 Makefile

开发时，您可以使用 [Makefile](https://en.wikipedia.org/wiki/Make_(software)) 进行以下操作：

| 命令行           | Description                                                                       |
|---------------|-----------------------------------------------------------------------------------|
| apidoc        | 生成 API 文档 /Generate documentation of API                                          |
| clean         | 清除目录以进行重置/Clean directories for reset                                             |
| code-sniff    | 使用 PHP Code Sniffer (`PSR2`) 检查 API /Check the API with PHP Code Sniffer (`PSR2`) |
| composer-up   | 使用 composer 更新 PHP 依赖 / Update PHP dependencies with composer                     |
| docker-start  | 创建并启动容器 / Create and start containers                                             |
| docker-stop   | 停止并清除所有服务 / Stop and clear all services                                           |
| gen-certs     | 为 `nginx` 生成 SSL 证书 / Generate SSL certificates for `nginx`                       |
| logs          | Follow 日志输出 / Follow log output                                                   |
| mysql-dump    | 创建所有数据库的备份 / Create backup of all databases                                       |
| mysql-restore | 恢复所有数据库的备份 / Restore backup of all databases                                      |
| phpmd         | 使用 PHP Mess Detector 分析 API / Analyse the API with PHP Mess Detector              |
| test          | 使用 phpunit 测试应用程序 / Test application with phpunit                                 |

### 例子

启动应用程序 :

```sh
make docker-start
```

显示帮助 :

```sh
make help
```

___

## 使用 Docker 命令

### 使用 composer 安装包

```sh
docker run --rm -v $(pwd)/web/app:/app composer require symfony/yaml
```

### 使用 composer 更新 PHP 依赖

```sh
docker run --rm -v $(pwd)/web/app:/app composer update
```

### 生成 PHP API 文档

```sh
docker run --rm -v $(pwd):/data phpdoc/phpdoc -i=vendor/ -d /data/web/app/src -t /data/web/app/doc
```

### 使用 PHPUnit 测试 PHP 应用程序

```sh
docker-compose exec -T php ./app/vendor/bin/phpunit --colors=always --configuration ./app
```

### 使用 [PSR2](http://www.php-fig.org/psr/psr-2/) 修复标准代码

```sh
docker-compose exec -T php ./app/vendor/bin/phpcbf -v --standard=PSR2 ./app/src
```

### 使用 [PSR2](http://www.php-fig.org/psr/psr-2/) 检查标准代码

```sh
docker-compose exec -T php ./app/vendor/bin/phpcs -v --standard=PSR2 ./app/src
```

### 使用 [PHP Mess Detector](https://phpmd.org/) 分析源代码

```sh
docker-compose exec -T php ./app/vendor/bin/phpmd ./app/src text cleancode,codesize,controversial,design,naming,unusedcode
```

### 检查已安装的 PHP 扩展

```sh
docker-compose exec php php -m
```

### 处理数据库

#### MySQL shell 访问

```sh
docker exec -it mysql bash
```

和

```sh
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### 创建所有数据库的备份

```sh
mkdir -p data/db/dumps
```

```sh
source .env && docker exec $(docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

#### 恢复所有数据库的备份

```sh
source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

#### 创建单个数据库的备份

**`注意：`** 用您的自定义名称替换“YOUR_DB_NAME”。

```sh
source .env && docker exec $(docker-compose ps -q mysqldb) mysqldump -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" --databases YOUR_DB_NAME > "data/db/dumps/YOUR_DB_NAME_dump.sql"
```

#### 恢复单个数据库的备份

```sh
source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/YOUR_DB_NAME_dump.sql"
```


#### 从 [PDO](http://php.net/manual/en/book.pdo.php) 连接 MySQL

```php
<?php
    try {
        $dsn = 'mysql:host=mysql;dbname=test;charset=utf8;port=3306';
        $pdo = new PDO($dsn, 'dev', 'dev');
    } catch (PDOException $e) {
        echo $e->getMessage();
    }
?>
```

___

## 帮助我们

任何想法、反馈或（希望没有！）
