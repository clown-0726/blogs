# 使用 office 365 SMTP 发送企业邮件

> 发布日期：2022-03-10 16:23:10

![](https://img-blog.csdnimg.cn/img_convert/2dc0a300e43a44c132b326ffeed4b11d.png)



## 背景

最近公司的企业邮箱从 gmail 迁移到了 office 365。不得不说，微软 office 套件的功能在市场上还是无人可以取代的。公司自然需要用 office 365 向客户发送邮件，因此需要重新配置项目的 SMTP 服务器。

一开始直接将 gamil 的 SMTP 服务器地址换成 office 365 的地址即“smtp.office365.com”，但是一直收到下面的错误提示，很容易想到是用户名和密码错误，在确认用户名和密码没有问题之后错误仍然一直存在，在查阅资料（baidu，google，bing）之后，也一直没找到合适的解决方案（基本都是在贴各种常规代码），最终在和客服进行几次交流之后找到了问题的所在。

```
Traceback (most recent call last):
	File "/Users/crown/Projects/hub/docs/email-try.py", line 49, in send
		smtp_obj.login('lilu.cao@orbitfin.ai',"xxx")
	File "/opt/miniconda3/envs/webtest/Lib/python3.7/smtplib.py", line 730, in Login
		raise last_exception
	File "/opt/miniconda3/envs/webtest/lib/python3.7/smtplib.py", line 721, in login
		initial_response_ok=initial_response_ok)
	File "/opt/miniconda3/envs/webtest/lib/python3.7/smtplib.py", line 642, in auth
		raise SMTPAuthenticationError(code, resp)
smtplib.SMTPAuthenticationError: (535, b'5.7.139 Authentication unsuccessful, the request did not meet the criteria to be authenticated successfully.
```

接下来通过配置下面三个步骤完成邮箱的 SMTP 邮件发送。

## 三步配置

#### 正确的发送代码（python）

这一步是最简单的，网上有很多优秀的封装的 email 发送类，我这里贴出一个之前用过的：

```python
import emails
from emails.template import JinjaTemplate as T


USERNAME = 'xxx'
PASSWORD = 'xxx'
smtp_conf = {'host': 'smtp.office365.com',
             'user': USERNAME,
             'password': PASSWORD,
             'port': 587,
             'tls': True}


def send_email():
    message = emails.html(subject=T('测试邮件'),
                          html=T('<p>详情见附件<br><br>'),
                          mail_from=('auto-reporter', USERNAME))
    message.attach(data=open('readme.md', 'r'), filename="readme.txt")
    r = message.send(to=('Orangleliu', USERNAME), smtp=smtp_conf)
    print(r)


def office365():
    import smtplib
    mailserver = smtplib.SMTP('smtp.office365.com', 587)
    mailserver.ehlo()
    mailserver.starttls()
    mailserver.login(USERNAME, PASSWORD)
    mailserver.sendmail(USERNAME, USERNAME, 'python email')
    mailserver.quit()


if __name__ == "__main__":
    send_email()

————————————————
版权声明：本文为CSDN博主「orangleliu」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/orangleliu/article/details/84065548
```

需要注意的是，在使用 office 365 的 smtp 服务的时候，加密要用 tls 而不是 ssl，也就是：

```python
mailserver.starttls()
```

#### 在组织中启用 SMTP AUTH

office 365 默认是对每一个用户都不启用 SMTP AUTH 的，因此我们需要对要使用 SMTP 进行发送邮件的用户开启 SMTP AUTH。一共有两种开启方式，一种是管理员登录 admin 控制中心（https://admin.microsoft.com/）针对相应的用户进行开启。具体步骤如下：

**1）登入 office 365 管理中心**

通过 https://admin.microsoft.com/ 登入管理中心，然后根据下图依次点击找到对应用户的 “Manage email apps”。

![](https://img-blog.csdnimg.cn/img_convert/141486056420e3399262fdaa248baca0.png)

**2）勾选 Authenticated SMTP**

然后点击 Manage email apps，在打开的侧边栏中将最后一项 “Authenticated SMTP” 勾选上。

![](https://img-blog.csdnimg.cn/img_convert/9469c31490c448a59fbe12407905e62e.png)

第二种方式是通过 powershell 进行操作，这种方式在这里不做具体阐述，具体可以参考下面的官网链接。

[中文版本] https://docs.microsoft.com/zh-cn/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

[英文版本] https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission 

#### 关闭安全性默认值

这一步至关重要，即便是我们开启了 SMTP AUTH，如果没有关闭安全默认值，那么邮件也无法使用。这一点其实已经在上面的“启用或禁用通过身份验证的客户端 SMTP (SMTP AUTH) Exchange Online”文章中指出，只是我们很难注意到这个点。

![](https://img-blog.csdnimg.cn/img_convert/789dc515dd22203b83e8ed176d933abb.png)

关于什么是“安全默认值”，可以参考下面点文章。

https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/concept-fundamentals-security-defaults

下面是关闭“安全默认值”的步骤

**1) 进入 Azure active directory**

我们进入 admin 管理中心后，根据下图，进入 Azure active directory。

![](https://img-blog.csdnimg.cn/img_convert/e383a4e6d3e5563fb3654885cbed24a1.png)

**2) 关闭安全默认值**

在 Azure active directory 界面，通过下图的指示，关闭安全默认值。

![](https://img-blog.csdnimg.cn/img_convert/fbb28c52fed57a8f68566e8a310e1189.png)

## 总结

完成上述三个步骤，我们就可以成功的通过 python 使用 office 365 的 SMTP 进行邮件发送。尤其是最有一步，对于默认安全值的问题，如果没有咨询微软的客服，很难排查出发送失败的原因。

## 参考文档

[1] Python 使用office365邮箱自动发送邮件 https://blog.csdn.net/orangleliu/article/details/84065548?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-84065548.pc_agg_new_rank&utm_term=Python+%E4%BD%BF%E7%94%A8office365%E9%82%AE%E7%AE%B1%E8%87%AA%E5%8A%A8%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6&spm=1000.2123.3001.4430  
[2] python3使用smtplib发送邮件，带xlsx附件 https://www.cnblogs.com/xiao987334176/p/11975248.html  
[3] 启用或禁用通过身份验证的客户端 SMTP (SMTP AUTH) Exchange Online https://docs.microsoft.com/zh-cn/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission  
[4] Enable or disable authenticated client SMTP submission (SMTP AUTH) in Exchange Online https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission  
[5] Security defaults in Azure AD https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/concept-fundamentals-security-defaults

