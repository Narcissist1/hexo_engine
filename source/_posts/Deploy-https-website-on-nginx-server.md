---
title: Deploy https website on nginx server
date: 2016-12-05 10:16:22
tags:	
		- https
		- nginx
categories: 技术相关
---
为了提高网站安全性，最近将 http 的站点转换部署为了 https。 使用了[Let's encrypt](https://letsencrypt.org/) CA认证证书，[certbot](https://certbot.eff.org/#ubuntutrusty-nginx)自动部署证书工具。

# Requirement

Nginx server (编译时需要加 --with-http_ssl_module 模块)。 Ubuntu 14.04 操作系统。

# Deploy HTTPS

## Install Certbot

运行以下命令，安装 certbot-auto 脚本

<!--more-->

```
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```
它会自动安装自身需要的依赖，并且自动更新，只需要运行以下

	$ ./certbot-auto

为了确保 certbot 有权限访问并验证目录，需要在 nginx 配置一下目录。(以默认目录为例)

	sudo vim /etc/nginx/sites-available/default

添加一下 block

```
server {
        . . .

        location ~ /.well-known {
                allow all;
        }

        . . .
}
```

检查 nginx 配置并重启：

```bash
	sudo nginx -t
	sudo nbgix -s reload
```
## 获取证书

首先要知道 web 根目录，也就是网站的根目录。我们使用 webroot 插件请求 ssl 认证，-d 参数指出绑定到哪个域名下面。可以指定多个域名，但是要确保最高域名在最前面。

	certbot-auto certonly -a webroot --webroot-path=/usr/share/nginx/html -d example.com -d www.example.com
	
> certbot-auto 需要管理员权限，非管理员用户需要加 sudo 运行

如果一切顺利，会有如下输出：

```
Output:
IMPORTANT NOTES:
 - If you lose your account credentials, you can recover through
   e-mails sent to sammy@digitalocean.com
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your
   cert will expire on 2016-03-15. To obtain a new version of the
   certificate in the future, simply run Let's Encrypt again.
 - Your account credentials have been saved in your Let's Encrypt
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Let's
   Encrypt so making regular backups of this folder is ideal.
 - If like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

/etc/letsencrypt/live/ 就是保存证书的目录， 2016-03-15 是有效期截止日。

## 证书文件

经过授权之后，你会得到如下文件：

* cert.pem: 你的域名的授权证书
* chain.pem: The Let's Encrypt 链证书
* fullchain.pem: 结合了 cert.pem 和 chain.pem 的文件
* privkey.pem: 你的私钥文件

牢记这些文件的位置非常重要，因为你要定期做备份，以防万一。
`/etc/letsencrypt/live/` 目录实际上是链接到`/etc/letsencrypt/archive` 目录上的。而且它会每次都链接到最新的授权文件。
可以通过如下命令查看：

	sudo ls -l /etc/letsencrypt/live/your_domain_name

## 生成强霍夫曼组

为了进一步加强安全性，我们需要生成一个 2048 位的霍夫曼组

	sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

生成后的文件存储在 /etc/ssl/certs/dhparam.pem

# Config TLS/SSL On Nginx Server

现在你有了一个 ssl 的证书，现在需要将它部署在 Nginx 上。打开 server 配置文件，注释掉，原来的 listen 80, listen servername 等行。

添加一下配置：

```
	listen 443 ssl;
  server_name example.com www.example.com;
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```
仅允许强 ssl 链接，并使用刚刚生成的霍夫曼组：

```
 				ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;
```
最后将 80 端口的请求全部转发到 443 ssl 端口，添加一个新的 server block：

```
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```
检查 nginx 配置并重启：

```bash
	sudo nginx -t
	sudo nbgix -s reload
```

# Renew 证书

[Let's encrypt](https://letsencrypt.org/)的证书有效期为90天，但是推荐每60天更新一次，留出 30 天的缓冲期。运行以下命令更新证书：

	certbot-auto renew
	
以上任务可以设置为一个 crontab 的定期任务。`sudo crontab -e`

```
	30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
	35 2 * * 1 /etc/init.d/nginx reload
```
每周一的，凌晨2:30，renew。 2:35 重启 nginx server。

# Conclusion

这样你就配置好了一个使用[Let's encrypt](https://letsencrypt.org/)证书的，基于 Nginx 的站点。




