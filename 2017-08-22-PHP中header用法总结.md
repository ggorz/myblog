---
layout: post
title: PHP中header用法总结
date: 2017-08-22 09:48:08
categories: php函数
---


header()函数的作用是：header() 函数向客户端发送原始的HTTP报头。发送一个原始 HTTP 标头[Http Header]到客户端。
标头 (header) 是服务器以 HTTP 协义传 HTML 资料到浏览器前所送出的字串，在标头与 HTML文件之间尚需空一行分隔。有关 HTTP 的详细说明，可以参考 [RFC2068官方文件](//www.w3.org/Protocols/rfc2068/rfc2068)。 在 PHP 中送回 HTML 资料前，会首先 传>完所有的标头。
<!-- excerpt -->

### header函数用法

>header()函数的作用是：header() 函数向客户端发送原始的HTTP报头。发送一个原始 HTTP 标头[Http Header]到客户端。  
标头 (header) 是服务器以 HTTP 协义传 HTML 资料到浏览器前所送出的字串，在标头与 HTML文件之间尚需空一行分隔。有关 HTTP 的详细说明，可以参考 [RFC2068官方文件](//www.w3.org/Protocols/rfc2068/rfc2068)。 在 PHP 中送回 HTML 资料前，会首先 传完所有的标头。  


#### 示例1：重定向
```
<?php

// 直接跳转
header("Location: http://www.me33.cn";);
exit;   //在每个重定向之后都必须加上“exit",避免发生错误后，继续执行。

// 等待3秒后跳转
header("refresh:3;url=http://www.me33.cn");
?>
```

#### 示例2：禁止页面在IE中缓存
要使用者每次都能得到最新的资料，而不是 Proxy 或 cache 中的资料，可以使用下列的标头
```
<?PHP
header( 'Expires: Mon, 26 Jul 1997 05:00:00 GMT' );
header( 'Last-Modified: ' . gmdate( 'D, d M Y H:i:s' ) . ' GMT' );
header( 'Cache-Control: no-store, no-cache, must-revalidate' );
header( 'Cache-Control: post-check=0, pre-check=0', false );
header( 'Pragma: no-cache' ); //兼容http1.0和https
?>
```
>CacheControl = no-cache  // HTTP1.1  
Pragma=no-cache  // HTTP 1.0  
Expires = -1  // 表示网页立即过期  

#### 示例3：修改返回状态码
`header(”Status: 404 Not Found”)。`   找不到网页  
`header(”Status: 403 Forbidden”)。`  禁止访问  

也有下面的写法：  
`header(”http/1.1 404 Not Found”);`  
第一部分为HTTP协议的版本(HTTP-Version)；第二部分为状态代码(Status)；第三部分为原因短语(Reason-Phrase)。

#### 示例4：下载档案( 隐藏文件的位置 )

html标签 就可以实现普通文件下载。如果为了保密文件，就不能把文件链接告诉别人，可以用header函数实现文件下载。
```
<?php
header("Content-type: application/x-gzip");
header("Content-Disposition: attachment; filename=文件名\");
header("Content-Description: PHP3 Generated Data");
```
header实现图片下载
```
header('Content-type: application/force-download');
header('Content-Transfer-Encoding: Binary');
header('Content-length: '.filesize($filePath));
header('Content-disposition: attachment; filename='.basename($filePath));
readfile($filePath);
```
header实现文件下载  
```
header("Content-Type:text/html;charset=utf-8");
header("Content-type:application/force-download");
header("Content-Type:application/octet-stream");
header("Accept-Ranges:bytes");
header("Content-Length:".filesize($filename));//指定下载文件的大小
header('Content-Disposition:attachment;filename="'.$file.'"');
//必要时清除缓存
ob_clean();
flush();
readfile($filename);
```

#### 注意：
一般来说在header函数前不能输出html内容，类似的还有setcookie() 和 session 函数，这些函数需要在输出流中增加消息头部信息。如果在header()执行之前有echo等语句，当后面遇到header()时，就会报出 “Warning: Cannot modify header information - headers already sent by ….”错误。就是说在这些函数的前面不能有任何文字、空行、回车等，而且最好在header()函数后加上exit()函数。
ob_clean();
flush(); 可以解决这个问题。


#### 更多的实例：
```
//文档语言
header('Content-language: en');

//告诉浏览器最后一次修改时间
$time = time() - 60; // or filemtime($fn), etc
header('Last-Modified: '.gmdate('D, d M Y H:i:s', $time).' GMT');

//告诉浏览器文档内容没有发生改变
header('HTTP/1.1 304 Not Modified');

//设置内容长度
header('Content-Length: 1234');

//设置为一个下载类型
header('Content-Type: application/octet-stream');
header('Content-Disposition: attachment; filename="example.zip"');
header('Content-Transfer-Encoding: binary');
// load the file to send:
readfile('example.zip');

// 对当前文档禁用缓存
header('Cache-Control: no-cache, no-store, max-age=0, must-revalidate');
header('Expires: Mon, 26 Jul 1997 05:00:00 GMT'); // Date in the past
header('Pragma: no-cache');

//设置内容类型:
header('Content-Type: text/html; charset=iso-8859-1');
header('Content-Type: text/html; charset=utf-8');
header('Content-Type: text/plain'); //纯文本格式
header('Content-Type: image/jpeg'); //JPG图片
header('Content-Type: application/zip'); // ZIP文件
header('Content-Type: application/pdf'); // PDF文件
header('Content-Type: audio/mpeg'); // 音频文件
header('Content-Type: application/x-shockwave-flash'); //Flash动画

//显示登陆对话框
header('HTTP/1.1 401 Unauthorized');
header('WWW-Authenticate: Basic realm="Top Secret"');
print 'Text that will be displayed if the user hits cancel or ';
print 'enters wrong login data';

```
