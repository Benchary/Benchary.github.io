> 关于 WordPress 网站备份迁移计划还是继续执行了，困扰很久，这么折腾，为什么一定要做迁移计划，基于以下3点：  
   - 验证 WordPress 网站可移植性  
   - 验证 WordPress 网站数据备份方案  
   - 验证 WordPress 网站数据云端下沉本地方案  
 
 所以 WordPress 网站备份迁移，尽管过程繁琐，但备份迁移计划至关重要。数据云端备份下沉本地是保障动态网站稳定运行的关键措施，不容忽视。


 ```mermaid
flowchart LR
A[数据备份] -->|数据导出| B[数据迁移]
B --> |数据导入|C[结束迁移]
C -->D[测试验证]
```

# 1. 迁移前准备

- 全环境资产梳理
- 线上云端数据备份
- 线上云端数据导出
- 线下本地环境准备

## 1.1 全环境资产梳理

- 云端环境:腾讯云轻量应用服务器
- 本地环境:VMware Workstation Pro 17


| 硬件/软件  | 名称 | 云端环境   | 本地环境  |
| -------- | -------- | -------- | -------- |
|   硬件   | CPU | 2核|2核 |
| 硬件     | 内存 | 2GB |1.5GB |
| 硬件     | 硬盘 | 40GB |50GB |
| 软件     | 操作系统 | Debian Linux 12 |Debian Linux 12 |
| 软件     | 网站服务 | Nginx 1.22.1 |Nginx 1.22.1 |
| 软件     | 数据库 | MariaDB 10.11.6 |MariaDB 10.11.6 |
| 软件     | 数据库管理 | phpMyAdmin |phpMyAdmin |

## 1.2 云端数据备份

迁移前给轻量应用服务器最后打一次快照

进入 **【轻量应用服务器】**  的菜单，选择  **【快照】**  ，点击 **【创建快照】**

## 1.3 云端数据导出

做接下来的操作之前，务必先停用 Nginx 服务

`systemctl stop nginx`

### 1.3.1 网站数据打包

进入 nginx 发布目录，压缩打包 WordPress 整个目录为 `wordpress.zip`

```bash
cd /var/www/
zip wordpress.zip ./wordpress -r
```

导出 WordPress SQL 数据

```bash
mysqldump -u root -p wordpress > wordpress.sql
```

> [!WARNING]
`-p` 后面跟着的是 wordpress 的数据库名称，默认名称是 wordpress


### 1.3.2 网站数据导出

将上述 WordPress 网站文件及 SQL 数据通过 WinSCP 传输工具下载到本地

## 1.2 本地环境准备

~~操作系统安装及LNMP部署省略~~，采用  `apt-get install` 方式即可

### 1.2.1 数据库配置

~~数据库安装配置过程省略~~，安装完成后，做以下操作

- 创建 WordPress 数据库
- 创建 WordPress 数据库用户名及密码


### 1.2.2 站点服务配置

> [!note]
集合了phpMyAdmin及wordpress的配置


编辑本地环境的 nginx 配置文件

```bash
vim /etc/nginx/sites-available/wordpress
```

将以下 phpMyAdmin 的 nginx 配置 (含 wordpress 的配置) 复制到此配置文件

```bash
server {
    listen 80;
    root /var/www/html;
    index index.php;
    location /wordpress {
        try_files $uri $uri/ /wordpress/index.php?$args;
    }
    location /phpmyadmin {
        alias /usr/share/phpmyadmin;
        index index.php;
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

Nginx 服务重载重启

```
systemcl reload nginx
systemcl restart nginx
```

# 2.  数据迁移

迁移过程就是将之前从云端导出到本地的网站数据，再导入到本地的过程

## 2.1 数据库 SQL 导入
通过 winscp 将已导出到本地的 **wordpress.sql** 上传到本地环境的 `/root` 目录
然后执行以下数据库的 SQL 导入
```bash
mysql -u root -p wordpress < ./wordpress.sql
```

### 2.1.1 SQL 值调整

此时的数据库中还保留着迁移前的 wordpress 中的 url，通过 phpMyAdmin 对 wordpress 数据库中的 `wp_options` 和 `wp_posts` 做以下调整

- 调整 **wp_options** 表

找到 `wp_options` 表中的 `siteurl` 和 `home` ，将这两个字段的 `option_value` 值都改成 `http://当前IP/wordpress`

- 调整 **wp_posts** 表

找到 `wp_posts` 表,然后编辑默认的 SQL ，重新输入以下 SQL ，将所有内容的 URL 全部替换成新环境 URL

```sql
UPDATE wp_posts SET post_content = REPLACE(post_content, '迁移前URL', '迁移后URL');
```

## 2.2 网站文件导入

利用 phpMyadmin 调整完数据库 SQL 的值后就停用 nginx ,因为接下来要导入 WordPress 网站文件

```bash
systemctl stop nginx
```

再用 WinSCP 将之前从云端导出到本地的 **wordpress.zip** 文件上传以下目录

```bash
/var/www/html/
```

进入此目录并解压 wordpress.zip 文件
```bash
cd /var/www/html/
unzip ./wordress.zip
```

设置 wordpress 目录网站权限

```bash
chown www-data:www-data /var/www/html/wordpress -R
chown 755 /var/www/html/wordpress -R 
```

# 3. 结束迁移

迁移后先重启数据库，再启用网站服务

重启数据库

```bash
systemctl restart mysqld
```

重启网站服务

```
systemctl start nginx
```

# 4. 测试验证

访问以下 URL 测试是否正常打开 WordPress 每个页面


```bash
http://当前IP/wordpress
```

访问以下 URL 测试是否正常打开 WordPress 的后台管理

```bash
http://当前IP/wordpress/wp-admin
```

> [!warning]
别忘了检查网站的 **首页** 地址是否是当前地址

# 5. 附:迁移的重要事项


> [!warning]
**一致性**
注意保证两个环境的操作系统及软件应用程序的版本，包括账号密码都尽量保证一致。提高迁移的成功率


> [!warning]
**错误回滚**
提前做好备份，有快照功能的打快照，提高迁移过程中的容错率


> [!warning]
**网络环境**
不要在DDNS环境下进行任何的操作无论是迁移还是备份，都会影响WordPress的迁移，反正我尝试了，没成功，保证网络结构简单，不要在复杂的网络环境下操作，尤其是与WordPress的网络中间隔了好几个安全设备的什么,比如WAF的防火墙之类的，即使有，请将策略全部放行，迁移成功后再启用策略，这是迁移成功的关键。
