---
layout: post
title: mac上配置iterm2支持rzsz命令
date: 2017-04-02 10:30:00
categories: iterm2
---



首先mac自带的终端是不支持lrzsz的，需要下载安装iterm2,[下载地址](http://www.iterm2.cn/download)。[lrzse官方下载地址](https://ohse.de/uwe/software/lrzsz.html)
<!-- excerpt -->

首先mac自带的终端是不支持lrzsz的，需要下载安装iterm2，下载地址：
http://www.iterm2.cn/download

1.安装lrzsz
brew info lrzsz
brew install lrzsz
 
2.安装脚本到mac指定目录，地址在： https://github.com/mmastrac/iterm2-zmodem
保存 iterm2-send-zmodem.sh 和 iterm2-recv-zmodem.sh 到mac的 /usr/local/bin/ 路径下 
注意添加脚本可执行权限：
chmod +x iterm2-send-zmodem.sh 
chmod +x iterm2-recv-zmodem.sh  
--------------------------------------------------------------------------------
3.iterm2 添加 triggers
Regular expression: \*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh 
 
添加步骤：command+“,” 组合键打开“Preferences”面板->Profiles选项卡->Advanced->Triggers（点击Edit即可）

4.重启iterm2，链接远程Linux，输入rz命令试一下吧（注意上传文件路径不能包含中文）。

注意在每个session中添加这个triggers

