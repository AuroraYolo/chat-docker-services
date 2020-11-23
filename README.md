
集成了多个服务，目前有nginx、php、mysql、mongodb、redis、rabbitmq、phpredisadmin、supervisord(安装在php容器中)。如果你想支持更多的服务，可以参考原有的服务目录结构、env.sample配置、docker-compose-sample.yml配置

# 目录
- [1.目录结构](#1目录结构)
- [2.使用](#2使用)
- [3.添加快捷命令](#4添加快捷命令)
- [4.配置文件说明](#5配置文件说明)
- [5.存在问题](#9存在问题)
- [6.swoole框架建议](#10swoole框架建议)

## 1.目录结构

```
/
├── Mysql                               Mysql目录
│   ├── conf                            Mysql配置文件目录
│   │    └── mysql.cnf                  mysql配置文件
│   └── Dockerfile                      Mysql构建文件
├── nginx                               Nginx
│   ├── conf                            Mysql配置文件目录
│   │    ├── conf.d                     Nginx站点ssh秘钥目录
│   │    ├── vhost                      Nginx虚拟站点配置目录
│   │    └── nginx.conf                 Nginx默认配置文件
│   └── Dockerfile                      Nginx构建文件
├── rebbitmq                            rebbitmq
│   └── Dockerfile                      rebbitmq构建文件
├── redis                               redis
│   └── Dockerfile                      redis构建文件
├── docker-compose-sample.yml           Docker-compose配置示例文件
├── env.smaple                          环境配置示例文件
└── README.MD                           文档说明
```

## 2.使用
1. 本地安装`git`、`docker`和`docker-compose`。
2. `clone`项目
3. 拷贝并命名配置文件，启动：
    ```
    $ cd dcnmp
    $ cp env.example .env
    $ cp docker-compose-php.yml docker-compose.yml
    $ docker-compose up
    ```
4. 访问在浏览器中访问：
 - [http://localhost](http://localhost)： 默认*http*站点
 - [https://localhost](https://localhost)：自定义证书*https*站点，访问时浏览器会有安全提示，忽略提示访问即可
两个站点使用同一PHP代码：`./www/localhost/index.php`。

要修改端口、日志文件位置、php代码目录位置、php扩展、php镜像版本等，请修改**.env**文件，然后重新构建：
```bash
$ docker-compose build php7    # 重建单个服务
$ docker-compose build          # 重建全部服务

```

## 4.添加快捷命令
在开发的时候，我们可能经常使用`docker exec -it`切换到容器中，把常用的做成命令别名是个省事的方法。

打开~/.bashrc，加上：
```bash
alias chat_nginx='docker exec -it chat_nginx /bin/sh'
alias chat_mysql='docker exec -it chat_mysql /bin/bash'
alias chat_redis='docker exec -it chat_redis /bin/sh'
alias chat_rabbitmq='docker exec -it chat_rabbitmq /bin/sh'
alias chat_redis='docker exec -it chat_redis /bin/sh'
alias chat_nsqd='docker exec -it chat_nsqd /bin/sh'
alias chat_nsq_admin='docker exec -it chat_nsq_admin /bin/sh'
```
其它的服务一样，自行设置

## 5.配置文件说明
```bash
主要说明几个全局变量：
// 各个服务软件的数据备份，比如mysql、redis等等，防止容器挂后数据不见的问题
DATA_PATH=/app/data
// 各个服务的配置,放在当前目录下每个服务目录的conf目录中
CONF_PATH=./
// 各个服务的log
LOG_PATH=/app/log
```

例如：mysql的配置复制到/app目录下，则为/app/conf/mysql/服务目录下的conf的所有目录文件。然后修改配置文件CONF_PATH=/app/conf

## 6.存在问题
1：目前在/php/Dockerfile文件中，无法使用$(nproc)，使用会报错，无法拿到cpu的数目

2：如果mysql启动不成功，查看容器报错为：[ERROR] Could not open file '/var/log/mysql/mysql.error.log' for error logging: Permission denied
则需要你删除映射的/app/log/mysql目录，然后重新开启容器,如果还不行，则在宿主机直接运行chmod -R 0777 /app/log/mysql


## 7.swoole框架建议
1、建议直接在宿主机使用supervisord来管理项目
