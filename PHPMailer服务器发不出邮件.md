title: PHPMailer服务器发送邮件
comments: true
thumbnailImage: https://pic.me33.cn/2018-04-02-055534.png
thumbnailImagePosition: right
date: 2017-05-09
tags:
 - php
 - PHPMailer
categories:
 - php
 keywords:
  - PHPMailer
----

phpMailer的使用还是挺简单的，基本上把类引进来去配置一些参数就可以使用了

<!-- excerpt -->
<!-- toc -->

# PHPMailer使用
> phpMailer的使用还是挺简单的，基本上把类引进来去配置一些参数就可以使用了



[github链接：phpmailer](https://github.com/PHPMailer/PHPMailer)

tp5可以直接在根目录执行 composer require phpmailer/phpmailer 就可以安装到项目里了

### 发送邮件方法
```
// 发送邮件方法
    private function send_mail($to,$fromname,$title,$content){
        try {
            $mail = new \PHPMailer(true);
            $mail->IsSMTP();
            $mail->CharSet='UTF-8'; //设置邮件的字符编码，这很重要，不然中文乱码
            $mail->SMTPAuth = true; //开启认证
//            $mail->Port = 25; //端口请保持默认
            $mail->Port = 465; //通过465端口进行更安全的SMTPS协议发送邮件
            $mail->SMTPSecure = 'ssl';
            $mail->Host = "smtp.exmail.qq.com"; //使用QQ邮箱发送
            $mail->Username = "your_email@you.com"; //这个可以替换成自己的邮箱
            $mail->Password = "your_password"; //注意 这里是写smtp的授权码 写的不是QQ密码
            //$mail->IsSendmail(); //如果没有sendmail组件就注释掉，否则出现“Could not execute: /var/qmail/bin/sendmail ”的错误提示
            $mail->AddReplyTo("service@51pain.com","mckee");//回复地址
            $mail->From = "service@51pain.com";
            $mail->FromName = $fromname;
            $mail->AddAddress($to);
            $mail->Subject = $title;
            $mail->Body = $content;
            $mail->AltBody = "To view the message, please use an HTML compatible email viewer!"; //当邮件不支持html时备用显示，可以省略
            $mail->WordWrap = 80; // 设置每行字符串的长度
            //$mail->AddAttachment("f:/test.png"); //可以添加附件
            $mail->IsHTML(true);
            $mail->Send();

        }
        catch (phpmailerException $e) {
            return array('type'=>0,'msg'=>"邮件发送失败：".$e->errorMessage());
        }
        return array('type'=>1,'msg'=>"邮件发送成功");
    }
```

### 调用方法
这里 EOT 块中的是邮件的样式
```
   public function sendEmail()
    {
        // 生成回调验证地址
        $uid = Cache::get(session_id().'uid');
        $email = Request::instance()->post('email');
        $data_str = "uid%".$uid."%email%$email";
        $crypt_str = urlencode(Des::encrypt($data_str,config('cryptKey')));  // 加密
        $url = DOMIN_NAME.url('index/Person/verfiyEmail').'?c='.$crypt_str;
        $to = $email;
        $fromname = '新云医疗';
        $title = "感谢您使用我们的账号，请您点击链接验证邮箱";
        $content = <<<EOT
<div id="contentDiv" onmouseover="getTop().stopPropagation(event);" onclick="getTop().preSwapLink(event, 'html', 'ZC1920-1EF1YLPRLnSMdEgQk5mXf74');" style="position:relative;font-size:14px;height:auto;padding:15px 15px 10px 15px;z-index:1;zoom:1;line-height:1.7;" class="body">    <div id="qm_con_body"><div id="mailContentContainer" class="qmbox qm_con_body_content qqmail_webmail_only"><table border="0" cellpadding="0" cellspacing="0" width="100%" style="border-collapse: collapse;background-color: #ebedf0;font-family:'Alright Sans LP', 'Avenir Next', 'Helvetica Neue', Helvetica, Arial, 'PingFang SC', 'Source Han Sans SC', 'Hiragino Sans GB', 'Microsoft YaHei', 'WenQuanYi MicroHei', sans-serif;">
  <tbody><tr>
    <td>
      <table cellpadding="0" cellspacing="0" align="center" width="640">
        <tbody><tr>
          <td style="height:20px;"></td>
        </tr>

        <tr>
          <td height="10"></td>
        </tr>
        <tr>
          <td>
            <table cellpadding="0" cellspacing="0" width="640">
              <tbody><tr style="line-height: 40px;">
                <td width="80" style="padding-left: 290px;">
                  <!--<a href="http://www.51pain.com" target="_blank">-->
                    <!--<img src="http://qn.image.51pain.com/170420224517148.png" width="54">-->
                  <!--</a>-->
                </td>
              </tr>
            </tbody></table>
          </td>
        </tr>
        <tr>
          <td height="40"></td>
        </tr>
        <tr>
          <td style="background-color: #fff;border-radius:6px;padding:40px 40px 0;">
            <table>
              <tbody><tr height="40">
                <td style="padding-left:25px;padding-right:25px;font-size:18px;font-family:'微软雅黑','黑体',arial;">
                  尊敬的<a href="mailto:wangzq1993@foxmail.com" target="_blank">$to</a>，您好,
                </td>
              </tr>
              <tr height="15">
                <td></td>
              </tr>
              <tr height="30">
                <td style="padding-left:55px;padding-right:55px;font-family:'微软雅黑','黑体',arial;font-size:14px;">
                  感谢您使用云贴账户。
                </td>
              </tr>
              <tr height="30">
                <td style="padding-left:55px;padding-right:55px;font-family:'微软雅黑','黑体',arial;font-size:14px;">
                  请点击以下链接进行邮箱验证，以便开始使用您的云贴账户：
                </td>
              </tr>
              <tr height="60">
                <td style="padding-left:55px;padding-right:55px;font-family:'微软雅黑','黑体',arial;font-size:14px;">
                  <a href="$url" target="_blank" style="display: inline-block;color:#fff;line-height: 40px;background-color: #1989fa;border-radius: 5px;text-align: center;text-decoration: none;font-size: 14px;padding: 1px 30px;">
                    马上验证邮箱
                  </a>
                </td>
              </tr>
              <tr height="10">
                <td></td>
              </tr>
              <tr height="20">
                <td style="padding-left:55px;padding-right:55px;font-family:'微软雅黑','黑体',arial;font-size:12px;">
                  如果您无法点击以上链接，请复制以下网址到浏览器里直接打开：
                </td>
              </tr>
              <tr height="30">
                <td style="padding-left:55px;padding-right:65px;font-family:'微软雅黑','黑体',arial;line-height:18px;">
                  <a style="color:#0c94de;font-size:12px;" href="$url" target="_blank">
                    $url
                </td>
              </tr>
              <tr height="20">
                <td style="padding-left:55px;padding-right:55px;font-family:'微软雅黑','黑体',arial;font-size:12px;">
                  如果您并未申请云贴官网账户，可能是其他用户误输入了您的邮箱地址。请忽略此邮件，或者<a href="mailto:payment@51pain.com" target="_blank" style="color:#0c94de">联系我们</a>。
                </td>
              </tr>
              <tr height="20">
                <td></td>
              </tr>
            </tbody></table>
          </td>
        </tr>
        <tr>
          <td style="height:40px;"></td>
        </tr>
        <tr>
          <td style="text-align: center;color:#7a8599;font-size: 12px;">
            <p></p>
            <p style="line-height: 20px;"></p>
            <p style="line-height: 20px;">
              <span onmouseover="QMReadMail.showLocationTip(this)" class="readmail_locationTip" onmouseout="QMReadMail.hideLocationTip(this)">北京市回龙观东大街</span> 388 号 B 座 501 楼 / <span style="border-bottom:1px dashed #ccc;z-index:1" t="7" onclick="return false;" data="400-898-6678">400-808-9176</span>
            </p>
          </td>
        </tr>
        <tr>
          <td style="height:50px;"></td>
        </tr>
      </tbody></table>
    </td>
  </tr>
</tbody></table><img width="1px" height="1px" src="http://email.morse.qiniu.com/o/eJwVzEEOwiAQQNHTyJIMMCOwYENi74EDKImUtKap8fTi7m3-z6E6Yl9FCxqUBdQAVqMBeZ220irvp5dbjHHBC0If-7vIra3tkDy6eIZi70CoFBXCTIm5sDEZlWd0Gh2JPZxpfXy3uTLzUMenp_b61z9B-iON">
<style type="text/css">.qmbox style, .qmbox script, .qmbox head, .qmbox link, .qmbox meta {display: none !important;}</style></div></div><!-- --><style>#mailContentContainer .txt {height:auto;}</style>  </div>
EOT;

        $rst = $this -> send_mail($to,$fromname,$title,$content);
        if($rst['type'] == 1){ // 发送成功，把邮箱记录表中，状态为-1
            $model = new \app\index\model\Member();
            $model -> saveEmail($uid,$email); // 保存更改email
        }
        return json($rst);
    }
```

### 问题
说一说今天遇到的问题：本地测试发邮件可以正常使用，上传到服务器后就不可以使用了。

下面copy点大神的解决思路：

 - PHPMaailer依赖PHP的sockets和openssl扩展，一般情况下都是默认开启的
 
打印phpinfo:
 ![](http://op1yjd9xg.bkt.clouddn.com/2017-05-09-0101.png)
openssl已经开启
![](http://op1yjd9xg.bkt.clouddn.com/2017-05-09-open.png)

 - PHP禁用函数
 
 之前是有东西的，包括socket的函数，删除它之后没效果，然后我就全部删除了。。可是还是不管用！
 ![](http://op1yjd9xg.bkt.clouddn.com/2017-05-09-disable.png)

 - 检查端口是否被占用
 smtp使用端口为25

`netstat -tunpl`
我的25端口是没有被占用的，所以明显不是这个问题

 - class.stmp.php问题
 
 这个是另一个大神的blog上说的
fsockopen() 可能被禁用；这个有可能是php.ini中禁用的
换用另一个函数

将之前的

```
$this->smtp_conn = fsockopen(
                $host,
                $port,
                $errno,
                $errstr,
                $timeout
            );
```
修改为：
```
$this->smtp_conn = @stream_socket_client($host.':'.$port,  // the host of the server
                $errno,  // error number if any
                $errstr, // error message if any
                $timeout);  // give up after ? secs
```
这样问题依然没有解决，本地还是可以正常发送邮件的。

 - 解决
 
 跟着大神的思路往下走，PHPMailer通过465端口进行更安全的SMTPS协议发送邮件

可以修改：

`$mail->Port = 465;`

为：

```
$mail->SMTPSecure = 'ssl';
$mail->Port = 465;
```
问题终于解决了，自己弄了有两个小时。。。

这里附上大神解决思路的blog链接[Linux服务器下PHPMailer发送邮件失败的问题解决](http://www.jb51.net/article/107433.htm)