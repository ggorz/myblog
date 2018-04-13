---
title: phpstorm配置xdebug
comments: true
thumbnailImage: https://pic.me33.cn/2018-04-13-085558.png
thumbnailImagePosition: right
date: 2018/4/13 下午4:54
tags:
 - phpstorm
categories:
 - ide
keywords:
 - phpstrom
 - xdebug
---

配置了好久终于配置成功了；简单记录一下步骤要点。

<!-- excerpt -->
<!-- toc -->

## 安装xdebug扩展
安装好xdebug扩展之后修改php.ini配置文件：
```shell
[Xdebug]
zend_extension="/usr/local/Cellar/php@7.1/7.1.6_18/lib/php/extensions/no-de     bug-non-zts-20160303/xdebug.so"
;extension=xdebug.so
xdebug.remote_enable=1
xdebug.remote_host=test.com
xdebug.remote_port=9001
;xdebug.profiler_enable = off
;xdebug.profiler_enable_trigger = on
;xdebug.profiler_output_name = cachegrind.out.%t.%p
;xdebug.profiler_output_dir = "/Users/zhaoqiwang/tmp"
;xdebug.show_local_vars = 0
xdebug.idekey=PHPSTROM

```
说明：1.一定是 zend_extension ；而非extension。后面是完整的路径  
     2. xdebug.remote_host 是调试的站点
     3.xdebug.remote_port 这个端口号不能是9000。nginx服务器php-fpm会占用9000端口
 
 ## 配置phpstrom
 1.配置debug
 ![](https://pic.me33.cn/2018-04-13-091133.png)
 2.添加server
 ![](https://pic.me33.cn/2018-04-13-090849.png)
 3.添加php web application
 ![](https://pic.me33.cn/2018-04-13-092228.png)
 ![](https://pic.me33.cn/2018-04-13-091009.png)
 
 
 ## 安装Chrome的XDebug插件
 参考链接：[Install Xdebug Helper](https://confluence.jetbrains.com/display/PhpStorm/Configure+Xdebug+Helper+for+Chrome+to+be+used+with+PhpStorm)
 ![](https://pic.me33.cn/2018-04-13-091257.png)
 安装完之后右击图标选择选项  
 修改ide为PHPstorm
 ![](https://pic.me33.cn/2018-04-13-091342.png)
 
 ## 使用
 ![](https://pic.me33.cn/2018-04-13-091613.png)
 debugger栏中就会看到变量信息
 ![](https://pic.me33.cn/2018-04-13-091648.png)
 
 