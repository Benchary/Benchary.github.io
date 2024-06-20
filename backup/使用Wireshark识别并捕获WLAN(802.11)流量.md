## 案例
想用 WireShark 捕获分析 WLAN 流量，但是发现捕获到的封包中并未识别到 802.11 的数据帧，而是被转义成了 Ethernet 数据帧信息，这样就屏蔽了真实的 802.11 信息 （如下信息），对WLAN 数据包分析造成了一定的阻碍。

```
> Frame 31: 714 bytes on wire (5712 bits), 714 bytes captured (5712 bits) on interface \Device\NPF_{6EE416E4-A3E4-4C74-9DDC-> FC5AA6ECC4C8}, id 0
v Ethernet II, Src: Intel_89:af:cb (5c:80:b6:89:af:cb), Dst: Cisco_ff:fc:04 (00:08:e3:ff:fc:04)
     > Destination: Cisco_ff:fc:04 (00:08:e3:ff:fc:04)
     > Source: Intel_89:af:cb (5c:80:b6:89:af:cb)
     > Type: IPv4 (0x0800)
> Internet Protocol Version 4, Src: 172.16.209.84 (172.16.209.84), Dst: 20.231.121.79 (20.231.121.79)
> Transmission Control Protocol, Src Port: instl-bootc (1068), Dst Port: http (80), Seq: 1780, Ack: 1, Len: 660
> [4 Reassembled TCP Segments (2439 bytes): #29(339), #30(1440), #36(1440), #31(660)]
> Hypertext Transfer Protocol
```
## 分析

其实通常是因为无线网络适配器的驱动程序或操作系统将无线帧转换为了以太网帧格式，以下是一些可能导致这种情况的原因：

- **1.操作系统的网络栈：**
   - 操作系统的网络栈可能会将无线帧（如802.11 帧）转换为以太网帧（Ethernet II 帧），然后再传递给上层协议处理。这是因为操作系统通常对以太网帧处理进行了优化，以提高效率。
- **2.无线网卡驱动程序：**
   - 某些无线网卡的驱动程序可能在将数据发送到操作系统之前，将无线帧封装在以太网帧中。这使得操作系统可以无需区分有线或无线来源，统一处理网络数据。
- **3.虚拟接口：**
   - 在某些操作系统中，无线网络接口可能被虚拟化为以太网类型的接口，以简化网络配置和数据处理。
- **4.监听模式：**
   - 如果无线网卡被置于监控模式，Wireshark 应该能够看到原始的 802.11 无线帧。但如果它没有被置于监控模式，或者操作系统默认处理了这些帧，Wireshark 看到的可能是以太网封装。
 
> [!TIP]
**Wireshark 本身并不改变数据包的封装类型，但它在显示数据包时，可能会根据操作系统提供的信息来标识封装类型。**

## 解决办法
获取原始 802.11 数据帧办法如下，下载安装 [Microsoft Network Monitor](https://www.microsoft.com/en-us/download/details.aspx?id=4865),并开始捕获。

> [!note]
**Microsoft Network Monitor** 是微软官方的数据包捕获器，可以捕获到原始 802.11 帧

以下是通过 Microsoft Network Monitor 捕获到 802.11 数据帧，并将其保存至 `.cap` 格式文件

```
  Frame: Number = 4, Captured Frame Length = 146, MediaType = WiFi
+ WiFi: [Unencrypted Data] .T....., (I) 
+ LLC: Unnumbered(U) Frame, Command Frame, SSAP = SNAP(Sub-Network Access Protocol), DSAP = SNAP(Sub-Network Access Protocol)
+ Snap: EtherType = Internet IP (IPv4), OrgCode = XEROX CORPORATION
+ Ipv4: Src = 172.16.209.84, Dest = 172.16.7.99, Next Protocol = TCP, Packet ID = 60106, Total IP Length = 82
+ Tcp: Flags=...AP..., SrcPort=1041, DstPort=HTTP Alternate(8080), PayloadLen=42, Seq=462750881 - 462750923, Ack=2748106622, Win=510
  Http: HTTP Payload, URL: 
  TLSSSLData: Transport Layer Security (TLS) Payload Data
+ TLS: TLS Rec Layer-1 SSL Application Data
```
再用 WireShark 打开上述由 Microsoft Network Monitor 捕获到的` .cap`格式文件，如下就可以显示原始 802.11 的数据帧信息了

```
> Frame 4: 146 bytes on wire (1168 bits), 146 bytes captured (1168 bits)
> NetMon 802.11 capture header
> 802.11 radio information
> IEEE 802.11 Data, Flags: .......T
> Logical-Link Control
> Internet Protocol Version 4, Src: 172.16.209.84 (172.16.209.84), Dst: proxy.sub.tfme.com (172.16.7.99)
> Transmission Control Protocol, Src Port: danf-ak2 (1041), Dst Port: http-alt (8080), Seq: 1, Ack: 96, Len: 42
```

## WireShark 捕获原始 802.11 数据帧

如果需要直接使用 WireShark 捕获原始 802.11 数据帧，需要将 WLAN 网卡启用**监听模式**，

> 如何判断 WLAN 网卡是否支持 **监听模式**可以通过 Microsoft Network Monitor 捕获 WLAN 数据包分析，如下图显示 `Not Monitor Mode` 就是不支持。
>
> ```
>- WiFi: [Unencrypted Data] .T....., (I) 
>  - MetaData: 
>     Version: 2 (0x2)
>     Length: 32 (0x20)
>   - OpMode: Extensible Station Mode
>      StationMode:           (...............................0) Not Station Mode
>      APMode:                (..............................0.) Not AP Mode
>      ExtensibleStationMode: (.............................1..) Extensible Station Mode
>      Unused:                (.0000000000000000000000000000...)
>      MonitorMode:           (0...............................) Not Monitor Mode
>     Flags: 4294967295 (0xFFFFFFFF)
>     RemData: Outbound
>     TimeStamp: 05/08/2024, 01:33:45.909746 UTC
> ```

然后点击`开始`运行 `nmwifi` 工具，它允许用户切换无线网卡到监控模式。

> [!note]
**nmwifi** 是安装 **Microsoft Network Monitor** 时自带的，使用 **nmwifi** 之前，需要将无线网卡设置为未连接任何网络的状态,

> [!note]
 **WireShark** 捕获原始 802.11 数据帧期间，切忌关闭 **nmwifi** 

> [!Warning]
 如果运行 **nmwifi** 提示 **“⚠ 没有可用的网络适配器”**，表示 WLAN 网卡并不支持 **监听模式**