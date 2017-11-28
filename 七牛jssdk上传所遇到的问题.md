---
title: 七牛jssdk上传所遇到的问题
date: 2017-08-21 09:47:31
tags:
 - php
 - 七牛
categories:
 - php
keywords:
 - 七牛
 - jssdk
thumbnailImage: https://pic.me33.cn/2017-08-22-041339.jpg

---
七牛云存储确实不错，不过开发过程中难免会菜很多坑。需求是在手机端的web网页上传头像，由于上传到服务器，再由服务器处理发给七牛再把返回的图片链接给到浏览器，这个过程比较缓慢，修改为用七牛的JSSDK来上传图片
<!-- excerpt -->

### 1.iOS点击不弹出拍照和图片按钮

七牛的qiniu.js根据提供的按钮id和父级id会创建隐藏的type为file的input  
之所以iOS上不呼出上传图片的按钮，是因为没有触发到七牛创建的input，原因可能是被自己的按钮阻挡了事件,七牛会自己根据所提供标签的z-index层级然后降1级创建新标签
![](https://pic.me33.cn/2017-08-16-06CAC636-EBF9-4EBC-B2CD-E276EE9F841A.png)

解决: 自己主动触发七牛的js（虽然有点low）

```
// 在自己的按钮上增加点击触发函数
<input type="button" onclick="clickqiniu(this)" style="z-index: 3" class="upload">

// 点击函数（不要等页面加载完再定义函数）
// 解决iOS点击不能触发上传事件
function clickqiniu(e){
    $(e).siblings("div").children("input").trigger("click");
    return false;
}
```

### 2.iOS上传的图片，在安卓手机上看旋转了90度
原因跟图片的EXIF信息有关系，可以再七牛图片链接后面加上?imageMogr2/auto-orient解决  
这样可以根据原图EXIF信息自动旋正

### 3.图片裁剪
官方文档给的图片处理的代码
![](https://pic.me33.cn/2017-08-16-QQ20170816-192509.png)

看到这里我是想要骂人的，鬼知道这个东西要放在什么位置  
经过我查看qiniu.js文件才明白上面的参数是什么意思  

qiniu.js中imageMogr2的定义：
![](https://pic.me33.cn/2017-08-16-QQ20170816-192902%402x.png)
可见这个函数有两个参数：第一个是对象类型的各种参数配置[七牛详细参数](https://developer.qiniu.com/dora/manual/1270/the-advanced-treatment-of-images-imagemogr2)、第二项是key值，key就是我们上传图片的名称  

这里有一个问题：
auto-orient 这个参数有 ‘-’ 会报错：  

![](https://pic.me33.cn/2017-08-16-QQ20170816-193317%402x.png)  

所以改成下图：  

![](https://pic.me33.cn/2017-08-16-QQ20170816-193340%402x.png)  

当然qiniu.js中也要修改:  

![](https://pic.me33.cn/2017-08-16-861FA2F6-A470-492E-83C0-FA381869F49D.png) 

我把处理图片的函数放在了完成上传图片之后，下面是我的代码：  
```
// 七牛上传图片
            var uploader = Qiniu.uploader({
                runtimes: 'html5,flash,html4',
                browse_button: 'pickfiles',//上传按钮的ID
                container: 'btn-uploader',//上传按钮的上级元素ID
                drop_element: 'btn-uploader',
                max_file_size: '100mb',//最大文件限制
                flash_swf_url: '/mobile_static/js/Moxie.swf',
                dragdrop: false,
                chunk_size: '4mb',//分块大小
                uptoken_url: "/Person/getqiniu_upload_token",//设置请求qiniu-token的url
//            uptoken : '3xKpCHDKrZEygn2ZTv73_dzB_3OQ4DQ3hBI0nc7b:UUdFvgVzQirBVGb_nKfcw4tMX8A=:eyJzY29wZSI6Inl1bnRpZSIsImRlYWRsaW5lIjoxNTAyNzg2NjgyfQ==',
                domain: "{$Think.config.qiniu.domain}",//自己的七牛云存储空间域名
                multi_selection: false,//是否允许同时选择多文件
                //文件类型过滤，这里限制为图片类型
                /*filters: {
                    mime_types : [
                        {title : "Image files", extensions: "jpg,gif,png"}
                    ]
                },*/
                auto_start: true,
                init: {
                    'UploadProgress': function(up, file) {
                        // 每个文件上传时，处理相关的事情
                    },
                    'FileUploaded': function(up, file, info) {
                        //每个文件上传成功后,处理相关的事情
                        //其中 info 是文件上传成功后，服务端返回的json，形式如
                        //{
                        //  "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
                        //  "key": "gogopher.jpg"
                        //}
                        var domain = up.getOption('domain');
                        var res = eval('(' + info + ')');
                        var sourceLink = domain + res.key;//获取上传文件的链接地址

                        // 生成截取图片
                        var imgLink = Qiniu.imageMogr2({
                            auto_orient: true,  // 布尔值，是否根据原图EXIF信息自动旋正，便于后续处理，建议放在首位。
                            strip: true,   // 布尔值，是否去除图片中的元信息
                            thumbnail: '!500x500r',  // 缩放操作参数
                            crop: '!500x500',  // 裁剪操作参数
                            gravity: 'Center',    // 裁剪锚点参数
                            quality: 40,  // 图片质量，取值范围1-100
                            format: 'png',// 新图的输出格式，取值范围：jpg，gif，png，webp等
                        }, res.key);

                        $(".loading_box").css("display","none");
                        //success_box(1,'头像上传成功');
                        $("#pickfiles").next("img").attr("src",imgLink);
                        $("#pickfiles").next("img").css("display","block");
                        $("input[name='avatar']").val(imgLink.substring(domain.length));
                        //do something
                    },
                    'Key': function(up, file) {
                        //当save_key和unique_names设为false时，该方法将被调用
                        loading_box(10)
                        var date = new Date().getTime();
                        var ext = Qiniu.getFileExtension(file.name);
                        var key = "avatar_"+date+"."+ext;

                        return key;
                    },
                    'Error': function(up, err, errTip) {
                        $(".loading_box").css("display","none");
                        error_box(5,'格式不正确');
                    }
                }
            });
```
### 4.上传图片的时候token出错

{error: "token not specified"}

![](https://pic.me33.cn/2017-08-16-8F666B38-393C-4CDA-9AA9-8F7046E9090A.png)

我这里的问题是主动请求token的URL返回token格式不正确，正确格式应该是这样子的：
```
{
"uptoken": "0MLvWPnyya1WtPnXFy9KLyGHyFPNdZceomL..."
}
```
