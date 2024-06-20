使用 Filezilla 客户端进行 SFTP 远程显示如下报错日志，无法正常登录

```
Couldn't agree a key exchange algorithm (available: curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512) 
```
这个日志的含义：**无法交换密钥信息** 于是我使用 Wireshark 抓了个包看了下，发现 3 次握手正常，至少说明网络层和传输层是没什么问题的，因为我之前就设置了正确的防火墙策略，而且防火墙日志是 allow 的信息，但是通过 Wireshark 追踪流发现没有 SFTP 的密钥交换信息。这个一般是应用层协议处理的问题，于是换了客户端软件，使用 WinSCP 连接试了下，正常了。

随后发现 SFTP 服务端软件是 Wing FTP Server，查了下它的官网，从官网发布的软件更新说明中发现 2024 年 2 月 27 日 **修复某些 SFTP 客户端无法连接到 WingFTP，并显示错误“无法交换密钥”，这是由主机密钥算法的顺序引起的。**  这条信息断定了这个  Wing FTP Server 版本没有更新。 如果环境中上部署的 Wing FTP Server 未更新修复上述问题，只能不断的尝试其他 SFTP 客户端，经过测试以下版本的 SFTP 客户端是不受影响的

- FileZilla_3.67.0_win64_sponsored
- FileZilla_3.67.0_win64_sponsored
- WinSCP 6.3.2
