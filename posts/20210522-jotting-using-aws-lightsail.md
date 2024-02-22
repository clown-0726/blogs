# Lightsail 随笔记录

> 发布日期：2021-05-22 20:10:10

今天最令人痛心的事情是袁隆平院士的离世，国士无双，愿您一路走好～

<!--more-->

![杂交水稻](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210522-jotting-using-aws-lightsail/za-jiao-shui-dao.jpeg)

AWS Lightsail 是 EC2 的更简单替代方案，它提供构建网站和小型 Web 应用程序所需的所有工具。相比较 EC2 需要大量的精力和专业知识进行设置和配置，而 Lightsail 只需简单的点击配置一下就可以得到一个小型站点。

这段时间用 lightsail 建立的一个 wordpress 站点，这篇随笔记录一下使用过程中出现的问题及结局的办法。

### 问题1，如何增加 wordpress 的上传限制
这个问题花费了一些时间，常规的 wordpress 解决方案很简单，只需要在 `.htaccess` 文件中增加下面代码即可。
```
php_value upload_max_filesize 256M
php_value post_max_size 256M
php_value memory_limit 256M
php_value max_execution_time 300
php_value max_input_time 300
```
但是这里不行，需要进行额外的配置。查阅分档后发现要做如下几步：  
找到文件 `/opt/bitnami/apps/phpmyadmin/conf/php-fpm/php-settings.conf`，然后修改里面的以下两个配置。
```
php_value[upload_max_filesize]=80M
php_value[post_max_size]=80M
```

### 问题2，如何开启 https 服务
这个没有在服务器内的 apache 进行配置 https 的服务，这里主要是使用更前一段的负载均衡器，这样在访问人数多的时候可以很容的扩展实例的个数。因此我们需要先在 lightsail 中创建一个负载均衡服务器然后将此挂在到实例上，最后在负载均衡服务器上申请 ssl 证书即可。

### 问题3，如何增加 wordpress 站点的配置
在使用的过程中并没有找到直接点击的方法去升级服务器的配置，这里的做法是先用当前的实例生成一个镜像，然后根据这个镜像还原出一个配置更高的实例，最后删除原来的镜像。

### 文件修改备份
```
-rw-rw-r--  1 bitnami daemon   405 Feb  6  2020 index.php
-rw-rw-r--  1 bitnami daemon 19915 Mar 26 03:25 license.txt
-rw-rw-r--  1 bitnami daemon  7345 Apr 15 13:28 readme.html
-rw-rw-r--  1 bitnami daemon  7165 Mar 26 03:25 wp-activate.php
drwxrwxr-x  9 bitnami daemon  4096 Mar 26 03:25 wp-admin
-rw-rw-r--  1 bitnami daemon   351 Feb  6  2020 wp-blog-header.php
-rw-rw-r--  1 bitnami daemon  2328 Oct  8  2020 wp-comments-post.php
// Below one is modified
-rw-r-----  1 bitnami daemon  4486 Apr 19 09:38 wp-config.php
-rw-rw-r--  1 daemon  daemon  2913 Mar 26 03:25 wp-config-sample.php
drwxrwxr-x  9 bitnami daemon  4096 Apr 19 11:03 wp-content
-rw-rw-r--  1 bitnami daemon  3939 Jul 30  2020 wp-cron.php
drwxrwxr-x 25 bitnami daemon 12288 Mar 26 03:25 wp-includes
-rw-rw-r--  1 bitnami daemon  2496 Feb  6  2020 wp-links-opml.php
-rw-rw-r--  1 bitnami daemon  3313 Mar 26 03:25 wp-load.php
-rw-rw-r--  1 bitnami daemon 44994 Apr 15 13:28 wp-login.php
-rw-rw-r--  1 bitnami daemon  8509 Apr 14  2020 wp-mail.php
-rw-rw-r--  1 bitnami daemon 21125 Mar 26 03:25 wp-settings.php
-rw-rw-r--  1 bitnami daemon 31328 Mar 26 03:25 wp-signup.php
-rw-rw-r--  1 bitnami daemon  4747 Oct  8  2020 wp-trackback.php
-rw-rw-r--  1 bitnami daemon  3236 Jun  8  2020 xmlrpc.php
```

```
-rw-r--r-- 1 bitnami root  256 Feb 15 13:18 banner.conf
drwxr-xr-x 2 bitnami root 4096 Feb 15 13:17 certs
// Below one is modified
-rw-r--r-- 1 bitnami root  719 Feb 15 13:18 htaccess.conf
-rw-r--r-- 1 bitnami root 1045 Feb 15 13:23 httpd-app.conf
-rw-r--r-- 1 bitnami root  455 Feb 15 13:23 httpd-prefix.conf
-rw-r--r-- 1 bitnami root  640 Feb 15 13:18 httpd-vhosts.conf
drwxr-xr-x 2 bitnami root 4096 Feb 15 13:18 php-fpm
```


### References:

[1] Modify The PhpMyAdmin File Upload Limit: https://docs.bitnami.com/bch/apps/wordpress-pro/administration/increase-upload-limit-phpmyadmin/  
[2] How to Increase File Upload Size in AWS Lightsail WordPress Site: https://www.theshravan.net/blog/how-to-increase-file-upload-size-in-aws-lightsail-wordpress-site/  
[3] How To Increase The WordPress Upload Limit On An AWS Lightsail Bitnami Installation: https://isotropic.co/how-to-increase-the-wordpress-upload-limit-on-an-aws-lightsail-bitnami-installation/

