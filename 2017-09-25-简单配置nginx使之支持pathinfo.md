---
layout: post
title: 简单配置nginx使之支持pathinfo
date: 2017-09-25 22:38:01
categories: pathinfo
---


pathinfo不是nginx的功能，pathinfo是php的功能;php中有两个pathinfo，一个是环境变量$_SERVER['PATH_INFO']；另一个是pathinfo函数，pathinfo() 函数以数组的形式返回文件路径的信息。
<!-- excerpt -->
<!-- toc -->

### 什么是pathinfo
一直叫了这么多年没有正经的看过这个定义；  

pathinfo不是nginx的功能，pathinfo是php的功能。转载[http://www.nginx.cn/426.html](http://www.nginx.cn/426.html)

php中有两个pathinfo，一个是环境变量$_SERVER['PATH_INFO']；另一个是pathinfo函数，pathinfo() 函数以数组的形式返回文件路径的信息;。  

nginx能做的只是对$_SERVER['PATH_INFO']值的设置。

### php中的两个pathinfo
#### php中的pathinfo()  
pathinfo()函数可以对输入的路径进行判断，以数组的形式返回文件路径的信息，数组包含以下元素。
 - [dirname]  路径的目录
 - [basename] 带后缀 文件名
 - [extension]  文件后缀
 - [filename]  不带后缀文件名(需php5.2以上版本)  

```php
# print_r(pathinfo("/nginx/test.txt"));

# 如下

Array
(
    [dirname] => /nginx
    [basename] => test.txt
    [extension] => txt
    [filename] => test
)

```
#### php中的$_SERVER['PATH_INFO']
PHP中的全局变量$_SERVER['PATH_INFO']，PATH_INFO是一个CGI 1.1的标准，经常用来做为传参载体。  

被很多系统用来优化url路径格式，最著名的如THINKPHP框架。  
对于下面这个网址：

http://www.test.cn/index.php/test/my.html?c=index&m=search

我们可以得到 $_SERVER['PATH_INFO'] ＝ ‘/test/my.html’,而此时 $_SERVER['QUERY_STRING'] = 'c=index&m=search';  

如果不借助高级方法，php中http://www.test.com/index.php?type=search 这样的URL很常见，大多数人可能会觉得不太美观而且对于搜索引擎也是非常不友好的(实际上有没有影响未知)，因为现在的搜索引擎已经很智能了，可以收入带参数的后缀网页，不过大家出于整洁的考虑还是想希望能够重写URL，  
下面是简单利用PHP_INFO对路由重写的代码  
```php
if(!isset($_SERVER['PATH_INFO'])){
    $pathinfo = 'default';
}else{
    $pathinfo = explode('/', $_SERVER['PATH_INFO']);
}

if(is_array($pathinfo) && !empty($pathinfo)){
    $page = $pathinfo[1];
}else{
    $page = 'default.php';
}

```

### Nginx对$_SERVER['PATH_INFO']的支持问题
在这之前还要介绍一个php.ini中的配置参数cgi.fix_pathinfo，它是用来对设置cgi模式下为php是否提供绝对路径信息或PATH_INFO信息。没有这个参数之前PHP设置绝对路径PATH_TRANSLATED的值为SCRIPT_FILENAME，没有PATH_INFO值。设置这个参数为cgi.fix_pathinfo=1后，cgi设置完整的路径信息PATH_TRANSLATED的值为SCRIPT_FILENAME，并且设置PATH_INFO信息；如果设为cgi.fix_pathinfo=0则只设置绝对路径PATH_TRANSLATED的值为SCRIPT_FILENAME。cgi.fix_pathinfo的默认值是1。

nginx默认是不会设置PATH_INFO环境变量的的值，需要php使用cgi.fix_pathinfo=1来完成路径信息的获取，但同时会带来安全隐患，需要把cgi.fix_pathinfo=0设置为0，这样php就获取不到PATH_INFO信息，那些依赖PATH_INFO进行URL美化的程序就失效了。

##### 1.可以通过rewrite方式代替php中的PATH_INFO
实例：thinkphp的pathinfo解决方案。设置URL_MODEL=2：
```
location / {
    if (!-e $request_filename){
        rewrite ^/(.*)$ /index.php?s=/$1 last;
    }
}
```

##### 2.nginx配置文件中设置PATH_INFO值
请求的网址是/abc/index.php/abc  PATH_INFO的值是/abc
SCRIPT_FILENAME的值是$doucment_root/abc/index.php
SCRIPT_NAME /abc/index.php
旧版本的nginx使用如下方式配置  
```
location ~ .php($|/) {
    set $script $uri;
    set $path_info "";

    if ($uri ~ "^(.+.php)(/.+)") {
        set $script $1;
        set $path_info $2;
    }

    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$script;
    fastcgi_param SCRIPT_NAME $script;
    fastcgi_param PATH_INFO $path_info;
}
```
新版本的nginx也可以使用fastcgi_split_path_info指令来设置PATH_INFO，旧的方式不再推荐使用，在location段添加如下配置。
```
location ~ ^.+.php {
  (...)
  fastcgi_split_path_info ^((?U).+.php)(/?.+)$;
  fastcgi_param SCRIPT_FILENAME /path/to/php$fastcgi_script_name;
  fastcgi_param PATH_INFO $fastcgi_path_info;
  fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
  (...)
}
```
补充：apache一般是以模块的方式运行php，apache可以对$_SERVER['PATH_INFO']的值进行设置，不需要另外配置。

### 简单配置nginx使之支持pathinfo另一种方式：
```
location ~ \.php {    #去掉$
     root          H:/PHPServer/WWW;
     fastcgi_pass   127.0.0.1:9000;
     fastcgi_index  index.php;
     fastcgi_split_path_info ^(.+\.php)(.*)$;     #增加这一句
     fastcgi_param PATH_INFO $fastcgi_path_info;    #增加这一句
     fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
     include        fastcgi_params;
}
```
