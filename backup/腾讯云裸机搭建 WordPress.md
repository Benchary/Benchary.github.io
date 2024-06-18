> 这是篇使用腾讯云的轻量服务器快速搭建自己的 WordPress 独立网站的教程  如果是国内用户，还是非常推荐使用腾讯云的轻量服务器作为独立网站的主机，因为性价比高。WordPress 是建站首选解决方案之一，是全球主流的 CMS 系统，小至个人博客，大至政府网站都会使用 WordPress，而且 WordPress 是开源免费的。  

> [!NOTE]
> 尽量选择裸机安装，不要选择默认的 wordpress 预装环境，因为内置的宝塔面板资源占用较大，严重影响 wordpress 资源开销，影响网站访问体验。

## 1. 操作系统安装

选择 Debian/Ubuntu 最新版本作为 wordpress 的运行操作系统,我选择 Ubuntu

登录系统后输入如下命令进行系统更新:

```bash
apt-get updata
apt-get upgrade
```
如果显示系统软件更新的选项框，直接跳过即可

## 2. WordPress 站点环境安装

LNMP 是 WordPress 网站部署的基础环境组件，前面一节是 Linux 系统安装过程，接下来安装 Nginx、MariaDB 和 PHP

### 2.1 安装 Nginx 网站服务

```bash
apt install nginx
systemctl start nginx
systemctl enable nginx
```

### 2.2 安装 MariaDB 数据库

```bash
apt install mariadb-server
```

#### 2.2.1 配置 MariaDB 安装安全性

运行命令`mysql_secure_installation`,按如下提示配置即可

```bash
Enter current password for root (enter for none):
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] y
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] n
Remove test database and access to it? [Y/n]y
Reload privilege tables now? [Y/n]y
```

> [!NOTE]
>数据库配置完成后即运行状态，无需再次重启数据库服务

### 2.3 安装 PHP 环境

```bash
apt install php php-fpm php-mysql php-common php-gd php-cli php-curl
```

## 3. WordPress 站点环境配置

> [!WARNING]
> 这里开始很重要，关系到后面 Wordpress 能否正常运行。


### 3.1 WordPress 的 Nginx 配置

创建一个新的 nginx 站点配置文件并命名 WordPress

```bash
vim /etc/nginx/sites-available/wordpress
```

> [!WARNING]
> 先将`sites-available`目录中默认的`default`文件重命名其他名称


将以下内容复制到刚新建的站点配置文件中

```
server {
 listen 80;
 server_name example.com www.example.com;

 root /var/www/wordpress;
 index index.php;

 location / {
 try_files $uri $uri/ /index.php?$args;
 }

 location ~ \.php$ {
 include snippets/fastcgi-php.conf;
 fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
 }

 location ~ /\.ht {
 deny all;
 }
}
```

> [!WARNING]
> 替换`example.com`为你的实际域名 替换`php7.4`为你的实际 php 版本


创建一个符合链接将站点配置文件链接到 sites-enable 目录

```bash
ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
```

运行`nginx -t`验证 nginx 配置是否正常，如果正常会有如下两行提示

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

重启 nginx 服务

```
systemctl restart nginx
```

### 3.2 WordPress 的数据库配置

本地登录 MariaDB 数据库

```bash
mysql -u root -p
```

创建 WordPress 数据库

```bash
create database wordpress;
```

创建 WordPress 数据库用户及密码 [^1]

```bash
create user 'wpuser'@'localhost' IDENTIFIED BY 'wpassword';
```

> [!WARNING]
> 替换`wpuse`为实际用户名 替换`wpassword`实际密码

赋予 WordPress 数据库用户权限

```bash
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. WordPress 主程序安装发布

接下来才是安装 WordPress 的步骤，前面都是环境准备

### 4.1 下载 WordPress

进入 Web 根目录，并下载最新版 WordPress

```bash
cd /var/www/
wget https://cn.wordpress.org/latest-zh_CN.zip
```

解压文件

```bash
unzip latest-zh_CN.zip
```

### 4.2 发布 WordPress

设置网站 WordPress根目录权限

```bash
chown -R www-data:www-data /var/www/wordpress
chmod -R 755 /var/www/wordpress
```

浏览器直接访问你的网址域名或IP地址`http://example.com `进行 web 界面安装

> [!NOTE]
> 替换`example.com`为你的实际域名


> [!WARNING]
> 为了网站安全，建议安装 ssl 证书使用`https://example.com`方式访问


## 5. web 界面安装注意点

> 安装过程就省略了，注意以下填写就行

- wordpress 数据名: 默认的`wordpress`
- wordpress 数据库用户名: 第 3.2 节的`wpuser`
- wordpress 数据库密码: 第 3.2 节`wpassword`
- wordpress 数据库前缀: 默认的 `wp_`
- wordpress 用户名: 自定义 `wordpress后台管理员账户名`
- wordpress 密码: 自定义 `wordpress后台管理员密码`


