> 动态网站的重要数据(文章、页面、图片)都存放在后端数据库，对于新手站长来说 Wordpress 都部署在 SiteGround 这类网站托管服务商，无需管理自己网站的数据库，博主可能职业因素，希望能将 WordPress 的管理更加透明化，更加自主可控，所以就涉及到了WordPress 的数据库管理，本文就推荐一款 WordPress 数据库 web 管理工具 phpMyAdmin ！

> [!TIP]
> phpMyAdmin 仅支持 MySQL / MariaDB 数据库的管理


##  1.  先决条件

安装 phpMyAdmin 的必要条件

###  1.1 子域名申请

用于 phpMyAdmin 的站点管理，比如 `sub.example.com`

###  1.2 子域名 SSL 证书申请

用于phpMyAdmin的站点管理，比如 `https://sub.example.com` 

##  2.  下载安装 phpMyAdmin

输入以下安装命令

```bash
apt install phpmyadmin nginx
```

> [!NOTE]
> 我的 Wordpress 环境是 Ubuntu server 操作系统


安装过程中若弹出图形化界面需要选择 `apache2`  `lighttpd` ,直接退出该选项，因为本文的 web 服务器是 nginx 

弹出 `MySQL application password for phpmyadmin:` 的提示时输入 phpMyAdmin 连接数据库密码即可


## 3. 创建 phpMyAdmin 站点发布目录

将 phpMyAdmin 整个 web 文件链接至 web 发布目录

```bash
ln -s /usr/share/phpmyadmin /var/www/html/exampledir

```

> [!NOTE]
> `exampledir` 替换成你自定义的目录名称，切忌替换成与 `phpmyadmin` 相同或类似字符的名称，容易受到网络安全威胁


## 4. phpMyAdmin 的 Nginx 站点配置

编辑 wordpress 的 `nginx` 配置文件

```bash
vim /etc/nginx/sites-available/wordpress
```

将以下 phpMyAdmin 的 `nginx` 配置(含 https 及其他安全设置),复制到此配置文件，切忌 **覆盖**，是 **新增** 配置

```bash
#防止跨域跳转 wordpress 站点
server {
    listen 443;
    server_name sub.example.com;
    return 404;
}
server {
#默认的 443 改成自定义端口
    listen 63890 ssl http2;
    server_name sub.example.com;
    ssl_certificate /etc/nginx/sub.example.com_bundle.crt;
    ssl_certificate_key /etc/nginx/sub.example.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    root /var/www/html;
    index index.php index.html index.htm;
    location /exampledir {
        alias /usr/share/phpmyadmin;
        index index.php;
    }
    #防止跳转根目录
    location / {
        return 404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }
    location ~ /\.ht {
        deny all;
    }
}
```
>[!WARNING]
> `sub.example.com` 替换实际子域名；`63890` 替换实际端口；`exampledir` 替换实际 phpMyAdmin 发布目录；`php8.2` 替换实际 php 版本


## 5. 重启 nginx 服务及重载 nginx 配置

```bash
systemctl restart nginx
systemctl reload nginx
```

## 6. 验证访问

浏览器输入以下地址即可访问 phpMyAdmin

```bash
https://sub.example.com:63890/exampledir
```
