# TCP/UDP包示例

> TCP 包 交换数据前会有2个客户端加1个服务器端的握手包 简称TCP3次握手

cmd:sudo tcpdump tcp port 80 -x -s0\
IP 192.168.1.251.http > 192.168.1.85.51335: Flags \[P.], seq 1:349, ack 83, win 229, length 348: HTTP: HTTP/1.1 502 Bad Gateway\
0x0000: 4500 0184 10b0 4000 4006 a423 c0a8 01fb\
0x0010: c0a8 0155 0050 c887 ae13 3c97 3137 7fc9\
0x0020: 5018 00e5 8617 0000 4854 5450 2f31 2e31\
0x0030: 2035 3032 2042 6164 2047 6174 6577 6179\
0x0040: 0d0a 5365 7276 6572 3a20 6e67 696e 782f\
0x0050: 312e 3130 2e33 2028 5562 756e 7475 290d\
0x0060: 0a44 6174 653a 2054 6875 2c20 3130 204a\
0x0070: 616e 2032 3031 3920 3036 3a35 303a 3339\
0x0080: 2047 4d54 0d0a 436f 6e74 656e 742d 5479\
0x0090: 7065 3a20 7465 7874 2f68 746d 6c0d 0a43\
...

> IPv4

4500 0184 版本4 头长5 服务类型00 包裹长度0184\
10b0 4000 重组标记10b0 标志 4 段偏移000\
4006 a423 生存时间40 协议代码06 头校验和a423\
c0a8 01fb 来源IP \[大端] 192.168.1.251\
c0a8 0155 目的IP \[大端] 192.168.1.85\
...可选选项 为空

> TCP

0050 c887 来源端口0050=80 目标端口c887=51335\
ae13 3c97 数据序号\
3137 7fc9 确认序号\
5018 00e5 \[1010000000110000000000011100101]偏移1010 保留000000 UAPRSF 110000 端口字段 000000011100101\
8617 0000 包和校验47e5 紧急指针0000\
... 可选选项 为空\
4854 5450 2f31 2e31 ... 用户数据

cmd:sudo tcpdump udp port 8899 -x -s0\
IP 192.168.1.251.8899 > 192.168.1.85.64450: UDP, length 7\
0x0000: 4500 0023 8893 4000 4011 2d96 c0a8 01fb\
0x0010: c0a8 0155 22c3 fbc2 000f 84c1 3131 3131\
0x0020: 3131 00

> IPv4 同上

4500 0023 版本4 头长5 服务类型00 包裹长度0023\
8893 4000 重组标记8893 标志 4 段偏移000\
4011 2d96 生存时间40 协议代码11 头校验和2d96\
c0a8 01fb 来源IP \[大端] 192.168.1.251\
c0a8 0155 目的IP \[大端] 192.168.1.85

> UDP

22c3 fbc2 来源端口22c3=8899 目标端口fbc2=64450\
000f 84c1 包长度 000f 校验和 84c1\
3131 3131 3131 00 用户数据

> IPv6地址格式

2001:0db8:3c4d:0015:0000:0000:1a2f:1a2b 3:站点前缀 1:子网 4:接口 站点前缀：通常由 ISP 或区域 Internet 注册机构分配给你 子网：您（或其他管理员）为您的站点分配的 16 位子网 接口：可以从接口的MAC地址自动配置，或使用 EUI-64 格式手动配置 备注：连续0可用::代替，示例 2001:db8:3c4d:15::1a2f:1a2b

> IPv6备注,包格式:

4bit 版本\
8bit 通讯量等级\
20bit 流标签\
16bit 有效载荷长度紧随 IPv6 数据包头之后的其余数据包部分\
8bit 下一层协议头部 与 IPv4 协议代码字段相同的值\
8bit 跳数限制，每过一个路由减１,到零包被丢弃\
128bit 发送者的地址\
128bit 接收者的地址 存在可选的路由头，预定接收者不一定就是接收者\
...扩展头 可能为空\
... 用户数据

> 程序抓到IP数据包后可根据以下类型对应解析:\
> IP包代码协议:

0 HOPOPT IPv6 逐跳选项\
1 ICMP Internet 控制消息\
2 IGMP Internet 组管理\
3 GGP 网关对网关\
4 IP IP 中的 IP（封装）\
5 ST 流\
6 TCP 传输控制\
7 CBT CBT\
8 EGP 外部网关协议\
9 IGP 任何专用内部网关 （Cisco 将其用于 IGRP）\
10 BBN-RCC-MON BBN RCC 监视\
11 NVP-II 网络语音协议\
12 PUP PUP\
13 ARGUS ARGUS\
14 EMCON EMCON\
15 XNET 跨网调试器\
16 CHAOS Chaos\
17 UDP 用户数据报\
18 MUX 多路复用\
19 DCN-MEAS DCN 测量子系统\
20 HMP 主机监视\
21 PRM 数据包无线测量\
22 XNS-IDP XEROX NS IDP\
23 TRUNK-1 第 1 主干\
24 TRUNK-2 第 2 主干\
25 LEAF-1 第 1 叶\
26 LEAF-2 第 2 叶\
27 RDP 可靠数据协议\
28 IRTP Internet 可靠事务\
29 ISO-TP4 ISO 传输协议第 4 类\
30 NETBLT 批量数据传输协议\
31 MFE-NSP MFE 网络服务协议\
32 MERIT-INP MERIT 节点间协议\
33 SEP 顺序交换协议\
34 3PC 第三方连接协议\
35 IDPR 域间策略路由协议\
36 XTP XTP\
37 DDP 数据报传送协议\
38 IDPR-CMTP IDPR 控制消息传输协议\
39 TP++ TP++ 传输协议\
40 IL IL 传输协议\
41 IPv6 Ipv6\
42 SDRP 源要求路由协议\
43 IPv6-Route IPv6 的路由标头\
44 IPv6-Frag IPv6 的片断标头\
45 IDRP 域间路由协议\
46 RSVP 保留协议\
47 GRE 通用路由封装\
48 MHRP 移动主机路由协议\
49 BNA BNA\
50 ESP IPv6 的封装安全负载\
51 AH IPv6 的身份验证标头\
52 I-NLSP 集成网络层安全性 TUBA\
53 SWIPE 采用加密的 IP\
54 NARP NBMA 地址解析协议\
55 MOBILE IP 移动性\
56 TLSP 传输层安全协议\
使用 Kryptonet 密钥管理\
57 SKIP SKIP\
58 IPv6-ICMP 用于 IPv6 的 ICMP\
59 IPv6-NoNxt 用于 IPv6 的无下一个标头\
60 IPv6-Opts IPv6 的目标选项\
61 任意主机内部协议\
62 CFTP CFTP\
63 任意本地网络\
64 SAT-EXPAK SATNET 与后台 EXPAK\
65 KRYPTOLAN Kryptolan\
66 RVD MIT 远程虚拟磁盘协议\
67 IPPC Internet Pluribus 数据包核心\
68 任意分布式文件系统\
69 SAT-MON SATNET 监视\
70 VISA VISA 协议\
71 IPCV Internet 数据包核心工具\
72 CPNX 计算机协议网络管理\
73 CPHB 计算机协议检测信号\
74 WSN 王安电脑网络\
75 PVP 数据包视频协议\
76 BR-SAT-MON 后台 SATNET 监视\
77 SUN-ND SUN ND PROTOCOL-Temporary\
78 WB-MON WIDEBAND 监视\
79 WB-EXPAK WIDEBAND EXPAK\
80 ISO-IP ISO Internet 协议\
81 VMTP VMTP\
82 SECURE-VMTP SECURE-VMTP\
83 VINES VINES\
84 TTP TTP\
85 NSFNET-IGP NSFNET-IGP\
86 DGP 异类网关协议\
87 TCF TCF\
88 EIGRP EIGRP\
89 OSPFIGP OSPFIGP\
90 Sprite-RPC Sprite RPC 协议\
91 LARP 轨迹地址解析协议\
92 MTP 多播传输协议\
93 AX.25 AX.25 帧\
94 IPIP IP 中的 IP 封装协议\
95 MICP 移动互联控制协议\
96 SCC-SP 信号通讯安全协议\
97 ETHERIP IP 中的以太网封装\
98 ENCAP 封装标头\
99 任意专用加密方案\
100 GMTP GMTP\
101 IFMP Ipsilon 流量管理协议\
102 PNNI IP 上的 PNNI\
103 PIM 独立于协议的多播\
104 ARIS ARIS\
105 SCPS SCPS\
106 QNX QNX\
107 A/N 活动网络\
108 IPComp IP 负载压缩协议\
109 SNP Sitara 网络协议\
110 Compaq-Peer Compaq 对等协议\
111 IPX-in-IP IP 中的 IPX\
112 VRRP 虚拟路由器冗余协议\
113 PGM PGM 可靠传输协议\
114 任意 0 跳协议\
115 L2TP 第二层隧道协议\
116 DDX D-II 数据交换 (DDX)\
117 IATP 交互式代理传输协议\
118 STP 计划传输协议\
119 SRP SpectraLink 无线协议\
120 UTI UTI\
121 SMP 简单邮件协议\
122 SM SM\
123 PTP 性能透明协议\
124 ISIS over IPv4\
125 FIRE\
126 CRTP Combat 无线传输协议\
127 CRUDP Combat 无线用户数据报\
128 SSCOPMCE\
129 IPLT\
130 SPS 安全数据包防护\
131 PIPE IP 中的专用 IP 封装\
132 SCTP 流控制传输协议\
133 FC 光纤通道
