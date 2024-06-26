作为计算机网络技术人员经常会接到类似的故障请求：
- 网页无法访问
- 某台服务器或者设备无法SSH远程

在通过 ping 测试网络正常的情况下，都会根据 TCP 的 3 次握手及 4 次挥手的协议，利用客户端 telnet 目标 IP + 端口的方式来确认目标应用服务是否存活，其实这测试的只是传输层 TCP 是否连接，一些场景下其实并不能完全有效验证目标服务应用层协议是否正常，那么这个判断依据是什么呢？

其实在这个问题上，我之前一直还有个疑惑 telnet 只是个应用层协议，用的是默认端口 23 ，但是为什么 telnet还能远程其他类似 {(http:80) (https:443) (ssh:22)} 这些端口呢？

针对上述两个疑问，继续进行如下剖析：

Telnet 客户端之所以能够远程连接其他服务端口，是因为它本身是基于 TCP 的程序。其实准确的说法是通过 Telnet 客户端程序发起 TCP 连接请求后，与被请求的目标主机的建立 TCP 连接，并在 TCP 传输层连接上发送任何数据。这意味着，只要远程主机上有相应的应用层服务在运行，应用层数据都将被传输层 TCP 承载（网络访问权限没有被限制的情况下), Telnet 客户端就可以连接目标服务并与该服务的 TCP 传输层进行通信。但是 Telnet 发起的 TCP 请求本身不能用于 HTTP 、 FTP 等应用层协议的数据传输。它仅仅将自已的字节输入拷贝至TCP连接中,此时目标 Web 服务器 HTTP 应用层接收到这样连接请求时,它首先需要等待客户端对 web 页面数据的请求。在这种情况下,Telnet 客户端不能提供 HTTP 请求,因为它此时不是工作在应用层，且不是对等 HTTP 请求服务，因此传输过程中不会产生任何应用层数据。

如下图利用 Telnet 客户端与 http 80 端口连接的示意图,

```
telnet_client <---应用层❌---> Http_server
telnet_client <---传输层✔️---> Http_server
telnet_client <---网络层✔️---> Http_server
telnet_client <---接口层✔️---> Http_server
```
所以我们平时用 telnet 测试的只是 TCP 的连接，对于上层应用层服务的状态是无法通过 telnet 进行测试的，所以 telnet 测试目标端口的作用，一般用在以下场景：
- 判断 TCP 连接是否被防火墙或者其他安全设备阻断
- 判断服务器的服务是否活动状态
- 判断服务端口是否启用

那什么情况 Telnet 可以请求应用层数据，这个很简单，只有目标是对等的 Telnet server 时才能传输 Telnet 应用层数据，如下图所示

```
telnet_client <---应用层✔️---> telnet_server
telnet_client <---传输层✔️---> telnet_server
telnet_client <---网络层✔️---> telnet_server
telnet_client <---接口层✔️---> telnet_server
```
