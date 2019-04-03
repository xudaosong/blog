outlook客户端使用VML替换图片
=======================
## 背景
使用QQ邮箱，发送一封带图片的邮件，该邮件通过outlook客户端接收，会无法自动下载图片
```html
<a href="https://baidu.com/login?locale=en-US" alt="Client Login" target="_blank">
    <img src='https://baidu.com/email_templates/btn-en.png' />
</a>
```
上面的代码，通过QQ邮箱发送后，在outlook客户端不能自动下载btn-en.png图片

## 解决
通过`<!--[if mso]><![endif]-->`进行判断是否为outlook客户端，如果是outlook则使用vml绘制，否则继续使用图片。因为其它客户端并不支持VML。
```html
<!-- 如果是outlook，则使用vml标记绘制按钮 -->
<!--[if mso]>
    <v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" xmlns:w="urn:schemas-microsoft-com:office:word" href="$DOMAIN_NAME$/login?locale=en-US"
        style="height:40px;v-text-anchor:middle;width:140px;" arcsize="20%" strokecolor="#70BDE4" fillcolor="#70BDE4">
        <w:anchorlock/>
        <center style="color:#ffffff;font-family:sans-serif;font-size:12px;font-weight:bold;">Client Login</center>
    </v:roundrect>
    <div style="width:0px; height:0px; overflow:hidden; display:none; visibility:hidden; mso-hide:all;">
<![endif]-->
<!-- 如果不是outlook则使用图片 -->
<a href="$DOMAIN_NAME$/login?locale=en-US" alt="Client Login" target="_blank">
    <img src='$DOMAIN_NAME$/email_templates/btn-en.png' />
</a>
<!--[if mso]>
    </div>
<![endif]-->
```