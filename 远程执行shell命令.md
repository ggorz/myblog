---
title: 远程执行shell命令
comments: true
thumbnailImage: https://pic.me33.cn/2017-11-28-092424.png
thumbnailImagePosition: right
date: 2017-09-25 22:38:01
tags:
 - shell
 - hexo
categories:
 - shell
keywords:
 - 远程执行
 - shell
---

shell执行远程命令；hexo远程发布；心酸的历程
<!-- excerpt -->
<!-- toc -->
### shell脚本

本地服务器
```bash
#!/bin/bash
cd ~/myblog
backtime=`date +%Y%m%d%H%M%S`
git add .
git commit -m ${backtime}
git pull
git push
remote_cmd="/Data/webapps/blog/pub.sh"
ssh -t -p 22 root@47.93.26.238 "source /etc/profile;${remote_cmd}"
echo 'done'
```

远程服务器
```
#!/bin/bash
cd /Data/webapps/blog/source/_posts
git pull
cd /Data/webapps/blog
hexo algolia
hexo g
```

### 解决 ssh user@ip 'command' 出现 command not found 的问题
转载[http://blog.csdn.net/whitehack/article/details/51705974](http://blog.csdn.net/whitehack/article/details/51705974)


原因：在远程服务器上执行echo $PATH与 ssh -t -p 20 root@ip "echo $PATH"的值是不同的
      所以会出现远程执行命令 command not found 的错误
解决办法 加上"soucre /etc/profile"