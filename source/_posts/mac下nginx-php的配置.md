title: mac 10.10.5 下nginx+php的配置
date: 2015-09-26 23:24:41
tags: [nginx,php]
---
捣腾mac的php运行环境浪费了一天时间 所以打算整理一下 以便以后自己要是再有需求不用像这次这么折腾..

## 安装homebrew
---

参考别人安装nginx文章的时候发现这个东西。是个类似ubuntu的apt-get和nodejs里的npm命令的管理工具,可以让你安装各种软件的时候爽一点。。

下面是安装的命令,需要ruby环境 mac里默认是已安装的:

``` bash
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

顺便记录一下常用的命令:

``` bash
$ brew update #更新可安装包的最新信息,建议每次安装前运行一下
$ brew search package_name #搜索package_name相关的包信息
$ brew install package_name #安装package_name的包
```

然后是这是[homebrew的官网地址](http://brew.sh/)

## 安装nginx
---

之后安装nginx的过程就十分轻松加愉快了

``` bash
$ brew search nginx
$ brew install nginx
```

我安装的是1.8.0版本的

## 配置
---

``` bash
$ cd /usr/local/etc/nginx/ #这是nginx默认的安装路径
$ vim nginx.conf #nginx服务公用的配置文件
```

nginx.conf配置成如下内容:

``` bash
worker_processes  1; #nginx进程数,建议设置为cpu总核心数

error_log       /usr/local/var/log/nginx/error.log warn; #全局错误日志存放路径 warn是类型

pid        /usr/local/var/run/nginx.pid; #进程所用pid配置文件
#事件模块相关配置
events {
    worker_connections  256; #进程最大链接数
}
#http服务器相关配置 http层
http {
    include       mime.types; #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型
    #日志格式设定
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log      /usr/local/var/log/nginx/access.log main; #定义本虚拟主机的访问日志
    port_in_redirect off; #将不会在网站后面加上监听的端口
    sendfile        on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    keepalive_timeout  65; #长链接超时时间,单位是秒

    include /usr/local/etc/nginx/conf.d/*.conf; #引入路径下的所有配置文件,用于将后面一个server层的配置文件引入进来
}
```

在当前目录下新建一个目录及配置文件

``` bash
$ mkdir conf.d && cd conf.d
$ vim default.conf
```

以下是default.conf的内容

``` bash
#虚拟主机的配置
server {
    listen       8080; #监听的端口
    server_name  localhost; #域名,可以有多个,用空格分隔

    root /Users/user_name/nginx_sites/; #该项要修改为你准备存放相关网页的路径

    location / { 
        index index.html index.php; #默认加载文件,可以有多个,用空格分隔
        autoindex on; #开启目录列表访问,适合下载服务器,默认关闭
    }   

    #proxy the php scripts to php-fpm php脚本文件代理给php-fpm执行
    location ~ \.php$ {
        include /usr/local/etc/nginx/fastcgi.conf; #引入nginx默认fastcgi配置文件
        fastcgi_intercept_errors on; #是否显示5xx错误或者4xx错误给客户看
        fastcgi_pass   127.0.0.1:9000; #fastcgi所监听的端口
    }   

}
```

## 安装php-fpm
---

mac中已经默认安装了php-fpm,不过得简单的修改一下配置文件
我就是卡在这一步浪费了好多时间

``` bash
$ sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
$ vim /private/etc/php-fpm.conf
```
找到error_log项 去掉行首的#注释,并且修改为error_log = /usr/local/var/log/php-fpm.log 不修改的话会报路径错误。

## 测试nginx服务
---

在你所配置的nginx配置文件的root项所写路径下随便创建测试文件index.php 内容可自行填写
启动nginx服务，sudo nginx； 
修改配置文件，重启nginx服务，sudo nginx -s reload 
启动php服务，sudo php-fpm； 
在浏览器地址栏中输入localhost:8080，如果配置正确地话，应该能看到PHP输出的页面。

## 参考资料
---

* [Mac OSX 10.9搭建nginx+mysql+php-fpm环境](http://my.oschina.net/chen0dgax/blog/190161#OSC_h2_3)
* [（总结）Nginx配置文件nginx.conf中文详解](http://www.ha97.com/5194.html)

比较懒 mysql的配置等下次用到再补上。。