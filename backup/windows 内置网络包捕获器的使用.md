正常情况下都会使用 Wireshark 进行网络数据包的捕获及分析，但是现在受于环境限制，电脑被禁止安装 Wireshark  软件，于是我找到了微软官方的网络包捕获器，比如 Microsoft Network Monitor 这款工具，但是客户还是不让装，办法还是比问题多，可以通过 cmd 命令提示符使用 Windows 内置的网络包捕获功能，操作如下：

## 网络包数据捕获

以**管理员身份**运行 cmd ，输入以下命令后，开始捕获网络数据包

```bash
C:\Windows\system32>netsh trace start capture=yes report=disabled

跟踪配置:
-------------------------------------------------------------------
状态:             正在运行
跟踪文件:         C:\Users\user\AppData\Local\Temp\NetTraces\NetTrace.etl
附加:             关闭
循环:           启用
最大大小:           512 MB
报告:             已禁用
```
此时行任意的网络活动都将被捕获器捕获

输入以下命令停用捕获器，然后会在`C:\Users\[管理员用户名]\AppData\Local\Temp\NetTraces\`目录下生成名为`NetTrace.etl`的文件，这个文件内容就是捕获到的所有网络数据包数据信息。

```bash
C:\Windows\system32>netsh trace stop
正在合并跟踪 ... 完成
跟踪会话已成功停止。
```

## 格式转换

此时`NetTrace.etl`无法被直接读取，即使可以读取，内容都是代码，可读性极差，至少数据包已经捕获到了，我将这份文件发送到自己电脑里，现在需要将`.etl`转换成可被 Wireshark 读取的 `.pcapng` 文件，这就需要借助微软官方的转换工具 [etl2pcapng](https://github.com/microsoft/etl2pcapng) 将下载好的 etl2pcapng 移动至`NetTrace.etl`同一目录中，然后以**管理员身份**运行 cmd ，输入以下命令后，开始转换格式

```bash
C:\Windows\system32>cd C:\Users\user\AppData\Local\Temp\NetTraces

C:\Users\user\AppData\Local\Temp\NetTraces>etl2pcapng.exe NetTrace.etl NetTrace.pcapng
IF: medium=eth                  ID=0    IfIndex=10      VlanID=0
Wrote 39365 frames to NetTrace.pcapng
```
此时将转换后的`NetTrace.pcapng`用 Wireshark 打开即可
