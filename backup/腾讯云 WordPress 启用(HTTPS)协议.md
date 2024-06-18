> [!WARNING]
> HTTPS 是网站必须使用的加密传输协议，需要安装配置 SSL 证书，这是网站必要的安全设置，接下来就演示在腾讯云环境中给WordPress 安装 Nginx SSL 证书！

> [!TIP]
> 谷歌浏览器默认支持 HTTPS，不支持 HTTP，比如输入 `example.com`  ,它会自动跳转 `https://example.com`

## 1. 设置 WordPress 站点 URL

在 WordPress 后台控制面板的设置中将 WordPress地址 (URL) 及站点地址 (URL) 都设置成 `https://example.com`

> [!WARNING]
> 改成 https 的 URL 后，wordpress 整个网站及后台控制台将无法访问,这是正常现象，继续参考本文以下内容配置即可，如果想回退此操作，请参考文末内容

## 2. 下载Nginx SSL证书

在我的 <strong>[腾讯云证书控制台](https://cloud.tencent.com/login?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2Fssl)</strong> 中选择您需要安装的证书并单击<u><strong>下载</strong></u>

在弹出的 <strong>证书下载</strong> 窗口中，服务器类型选择 <strong>Nginx</strong> ，单击下载并解压缩 <strong>example.com_nginx.zip</strong> 至本地电脑。然后发现解压后的 <strong>example.com_nginx</strong> 文件夹内有以下格式的证书文件：

- `example.com.csr`
- `example.com.key`
- `example.com_bundle.crt`
- `example.com_bundle.pem`

## 3. 上传Nginx SSL证书

使用 WinSCP（即本地计算机与远程服务器间的复制文件工具）登录 Nginx 服务器

将已获取到的  `example.com_bundle.crt`  证书文件和  `example.com_key` 私钥文件从本地目录拷贝到 Nginx 服务器的目录（Nginx 默认安装目录为 `/etc/nginx` ，请根据实际情况操作）

## 4. Nginx SSL证书生效设置

编辑 `/etc/nginx/sites-available` 根目录下的 wordpress 文件。修改内容如下：

> [!NOTE]
> 默认nginx配置文件在  `/etc/nginx/sites-available/default`  我这里的配置文件是  `wordpress` 

执行以下命令行编辑该文件

```bash
vim /etc/nginx/sites-available/wordpress
```
> [!NOTE]
>由于版本问题，配置文件可能存在不同的写法。例如：Nginx  版本为  `nginx/1.15.0`  以上请使用  `listen 443 ssl`  代替  `listen 443 ssl on` 。


复制如下配置到此文件并覆盖掉原内容

```bash
server {
    #SSL 默认访问端口号为 443
    listen 443 ssl http2;
    server_name example.com wwww.example.com;
    #请填写证书文件的相对路径或绝对路径
    ssl_certificate /etc/nginx/example.com_bundle.crt;
    #请填写私钥文件的相对路径或绝对路径
    ssl_certificate_key /etc/nginx/example.com.key;
    ssl_session_timeout 5m;
    #请按照以下协议配置
    ssl_protocols TLSv1.2 TLSv1.3;
    #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    #wordpress站点根目录
    root /var/www/wordpress;
    #wordpress站点索引页
    index index.php;
    #wordpress站点根路径
    location / {
        try_files $uri $uri/ /index.php?$args;
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

命令 `nginx -t` 验证配置

按如下命令重启 nginx 服务及重载 nginx 配置

```bash
systemctl restart nginx
systemctl reload nginx
```

此时可以通过 `https://example.com` 访问网站,但是每次访问都必须手动输入前缀 https ，很麻烦，继续看下文 HTTP 自动跳转 HTTPS 的安全配置

## 5. HTTP 自动跳转 HTTPS 安全设置

在上述第3大步骤 <strong>Nginx SSL证书生效设置</strong> 的配置中 <strong>新增</strong> 以下配置，复制粘贴即可

```bash
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

命令 `nginx -t` 验证配置

按如下命令重启 nginx 服务及重载 nginx 配置

```bash
systemctl restart nginx
systemctl reload nginx
```

现在就可以直接通过 `example.com` 访问网站，域名会自动跳转 https ，无需手动在 URL 前缀输入 `https`

