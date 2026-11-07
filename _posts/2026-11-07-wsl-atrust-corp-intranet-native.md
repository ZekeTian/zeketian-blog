---
layout: post
title: WSL 通过 aTrust 访问公司内网：用原生能力，不加转发层
categories: Linux
tags: WSL aTrust DNS
author: ZekeTian
---

* content
{:toc}

Windows 上开着 aTrust，浏览器能打开内网系统，同一时刻 WSL 里却访问不了。本文主要讲解让 WSL 里的发行版自行解析并访问内网域名与业务系统的做法，从而不另起代理、不装转发工具，也不动 Windows 侧的 aTrust 设置。

常见的两条建议是“换 mirrored”和“装 wsl-vpnkit”：前者会遇到零信任客户端的架构冲突，后者在已经有一层原生转发的前提下再叠一层；两者都没回答排查里最花时间的那一问：网络模式选对了之后，DNS 为什么还单独断一层。

注意，本文使用的 WSL 内核版本为 **6.18.54.1-microsoft-standard-WSL2**（构建戳 2026-10-06，落在 WSL 发布 3.0.2 与 3.0.3 之间），发行版为 Ubuntu 26.04.1 LTS。这套做法最早在 WSL 2.x 的 `virtioproxy` 上调通，环境快照与验收取自升级后的 3.0.2/3.0.3；两边是同一个后端，3.x 只是把模式改名叫 `consomme`（见 2.2），**不必为了它升级 WSL**。

本文说的“原生”，指 WSL 自带的能力加发行版自带的组件，真正要动手改的只有 DNS 一层；这套配置已在装了 aTrust 的机器上验证可用，WSL 里的内网域名解析和内网业务系统访问都正常。只想动手的读者可以直接跳到 3.1，想弄清为什么的从 2.1 读起。



## 1. 问题描述

现象按发现顺序列在下面，随后是这台机器的环境快照。看清断点落在哪一层，第 2 章才谈得上归因。证据分三级标注：[实测] 是实际跑出来的配置与命令输出；[上游] 指 GitHub issue、PR 或官方文档可查；[推断] 是由机制或时间线推出、没有直接文档的结论。

1. 同一个内网地址，Windows 侧浏览器能打开，WSL 里不通。
2. 最先暴露的是 DNS：查内网域名返回 `socket.gaierror: [Errno -2] Name or service not known`，断点在解析层，由 2.3 解释。
3. 默认的 NAT 模式不通，换 `networkingMode=mirrored` 也不通，期间开过 `dnsTunneling=true`，一样没有改善。三条后来都被否证（2.1、2.4、4.1）。
4. 最后换到 virtioproxy 模式（3.x 起改名为 Consomme）：**即使完全不连 aTrust，内网域名也解析不了**。可见 DNS 故障与 VPN 正交，aTrust 只是把它放大成“内网访问不了”。

现象 4 对应的配置形态如下 [实测]：

| 项 | 值 |
|---|---|
| `/etc/resolv.conf` | 符号链接 → `/run/systemd/resolve/stub-resolv.conf`，内容 `nameserver 127.0.0.53`、`search .` |
| `/etc/systemd/resolved.conf` | `DNS=223.5.5.5 8.8.8.8`、`Domains=~.` |
| `resolvectl status` | `DNSSEC=no (unsupported)`、`resolv.conf mode: stub`、链路 `Default Route: no` |
| 解析结果 | `example.com` 成功；内网域名 `Errno -2`（NXDOMAIN 语义） |

`/etc/wsl.conf` 里与本文相关的只有两项 [实测]：`[boot] systemd=true` 与 `[network] generateResolvConf=false`。

接口形态 [实测]：`enP37022p0s0 192.168.71.62/24`（镜像的宿主地址）、`loopback0 169.254.73.250/28`、默认路由 `via 192.168.71.1 onlink`。这是 Consomme 模式的指纹：宿主地址网卡 + link-local 辅助网卡 + 没有真实 L2 邻居的合成网关 [推断]。`169.254.73.250` 后面还会用到，它就是伪装成 DNS 服务器的合成地址。

版本以宿主 `wsl --version` 为准，内核与 WSL 包版本可解耦 [推断]：构建戳 2026-10-06 落在 3.0.2（10-05）与 3.0.3（10-09）之间，本文按约 3.0.2/3.0.3 讨论。

验证一步：发行版里执行 `uname -r`，Windows 侧执行 `wsl --version`，两组版本号记下来。后文所有“这个 bug 修没修”的判断都以这两个数为准。

## 2. 为什么通不了

三个问题按顺序回答：aTrust 要什么样的流量才放行（2.1），网络模式选哪个（2.2），DNS 为什么在模式选对之后还断一层（2.3、2.4）。

### 2.1 aTrust 要先认出“是谁”，mirrored 给不出这个身份

结论先给：mirrored 模式失败是**架构错配**，不是配置错误。

aTrust 这类零信任客户端用 WFP 过滤驱动加虚拟网卡做“流量身份化”，按进程、用户、设备环境、目的地址分类和重定向。它得先看见一个**带身份的 Windows 传输层端点**，才谈得上放不放行 [上游+推断]。

mirrored 模式把 Windows 网卡镜像进 Guest。Guest 有自己的协议栈，包以“转发身份”注入主机栈，**没有进程身份**，等于在客户端钩子之下转发。

Microsoft 的排错文档对同架构的 Global Secure Access 写得很直接：“WSL 的流量在 GSA 客户端钩子之下被转发”，并建议关闭 mirrored 或关闭 DNS tunneling [上游]。aTrust 与 GSA 属于同一类客户端，这是 mirrored 下失败的同类反证。

同一套判断也解释了 NAT：Guest 的源地址是 `vEthernet (WSL)` 网段，aTrust 既认不出进程身份，也不认为这个源地址可达（详见 4.1）。

验证一步：打开排错文档搜 “GSA”，能读到上面引的那句原文；本文对 aTrust 的判断就架在它上面。

### 2.2 数据面：让主机进程代为发起

排掉 NAT 与 mirrored 之后，剩下能选的模式必须满足一个条件：**让 WSL 的连接以 Windows 主机进程的身份发出**。Consomme 正好是这个形态 —— 这是它成为方案载体的原因，不是因为它新。

Consomme 不是包级转发，而是**用户态 NAT**：OpenVMM 里的 `net_consomme`，WSL 里原本叫 `VirtioProxy`，2026-06-23 的 #40873 直接改名，而 `.wslconfig` 仍把 `virtioproxy` 当合法别名 [上游]。实现上是 socket 级映射：主机侧有 port tracker，Guest 的 `connect()` 要在主机侧换成主机 socket（`DuplicateSocketFd`、seccomp、pidfd 那一套）[上游]。

改名不换机制，这一点决定本文的适用范围：`.wslconfig` 里写 `networkingMode=consomme` 还是 `networkingMode=virtioproxy`，进到 OpenVMM 里是同一个后端，**WSL 2.x 与 3.x 的差别只在名字** [上游]。下面三条推论对两代版本同样成立，WSL 2.x 不必为此升级。

由此可以推出三件事 [推断]：

1. Guest 的 `connect()` 在主机侧被终结，由 Windows 进程用主机 TCP 栈重新发起。aTrust 看到的是本机进程，隧道路由自动生效，Guest 里不用加路由。
2. 没有二次 NAT 与非对称回程：回程由主机 socket 交回设备侧，再灌给 Guest。
3. 两侧各自协商 MSS，**不存在“隧道套隧道”的 IP 层 PMTU 黑洞**。NAT 模式下 aTrust 再封装后有效 MTU 掉到 1400 以下，PMTUD 一旦被掐断就是“握手成功、传数据卡死”。

验证一步：查 #40873 与 OpenVMM 文档，两处都把 `net_consomme` 定位成用户态 NAT，不是包过滤驱动。定位错了，2.1 的身份问题会被误读成网卡没镜像全。

### 2.3 DNS：网络模式选对了，DNS 却走了一条旁路

网络模式定了，内网访问还是不通，卡在 DNS。结论：**Consomme 的 DNS 根本不在 2.2 那条数据面上**。

做法 [上游]：DHCP 把 Consomme 自己的网关地址下发给 Guest（即 `loopback0 169.254.73.250`），发往该地址 53 端口的请求在设备侧被拦截、由主机代答（Windows 走 `DnsQueryRaw` 一类 API，Linux 与 macOS 走 `res_send`；openvmm #2633、#2807，WSL #14461）。早期版本是“透传主机 DNS 列表，Guest 直接向真实服务器查询”，那条路**走数据面**，反而更正确。

这条旁路有三个缺陷，正好对上第 1 节的现象：

1. **不在数据面上。** 代答走宿主解析器的 API 与选择逻辑，拿不到 aTrust 的分域（split-horizon）策略。结果要么内网域名 NXDOMAIN（`Errno -2` 就是这条），要么拿到公网同名记录，变成“连上了，但连错了地方”。
2. **绑在 link-local 合成地址上。** 只有拦截在线时它才像个 DNS 服务器；拦截一失效（`dnsTunneling` 没生效、回落到 `none`），查询就发往无人应答的地址，表现为**纯超时**。
3. **没有搜索域能力。** 单上游承载不了“内网域走内网、其余走公网”，也解释不了短名。

还有一个常见误判：`search .` 表示**搜索域为空**，所以单标签短名（如 `intranet`）在任何 DNS 配置下都会失败。这不是 aTrust 造成的，`Domains=~.` 只提供“默认路由”，不提供搜索域 [实测+上游语义]。`/run/systemd/resolve/resolv.conf` 同样是 `search .`，可以自己 `cat` 确认。

验证一步：`resolvectl status` 里看当前链路的 `DNS Servers`。出现 `169.254.73.250`，说明上游就是那台合成网关，解析走的是本节这条旁路；改完 3.1 之后再用 3.4 的两条 `getent` 复验。

### 2.4 为什么 dnsTunneling 救不了这一层

第 3 条里“开了 `dnsTunneling` 也没用”同样归因到版本，三个关键节点 [上游]：

| 时间 | 事件 |
|---|---|
| 2026-03-18 | **#14461 合入**：为 virtioproxy 打通 DNS tunneling；`BestEffortDnsParsing`、`DnsTunnelingIpAddress` 等 socket 选项被清除，改为在 `VirtioNetworkingFlags` 上给设备侧一个 flag |
| 2026-10-26 | **#42026 合入**：修复“IPv6 被禁用时 Consomme 回落到 `networkingMode=none`” |
| 2026-11-06 | **#42180 合入**：补上缺失的 DNS 后缀 watcher（Windows 只改后缀时 Guest 搜索域会陈旧） |

当前 build 内建 2026-10-06：晚于 #14461 约六个半月，早于 #42026 与 #42180，正好落在“tunneling 已实现、回落与搜索域两处未修”的窗口里 [推断]。所以 `dnsTunneling=true` 无效不该归因于开关本身，问题出在它的作用域之外：拦截点可能没命中；命中之后由宿主解析器代答，拿不到分域策略；拦截失效时回落到 `none`；搜索域陈旧则要等 #42180。前三条就是 2.3 的三个缺陷。

同一张表还能推出一件事 [推断]：早于 #14461 的版本走透传，DNS 查询直接落在数据面上，不会撞上这条旁路，旧版因此可能比现在顺。这只是一个机制推论——本文没有在那些版本上实测，不把它当结论；而且它不构成“新版才需要本文”的理由：透传路径同样拿不到搜索域，短名照样失败，断连重连后列表还可能陈旧。3.1 的改法在 WSL 2.x 上同样成立，只是不再依赖那条透传路径。

验证一步：拿 #42026 的合入日期与 `wsl --version` 对一下，早于 10-26 就不要指望 IPv6 相关的回落行为正常。

## 3. 解决办法

要让内网访问通，只需要动 DNS 这一层：保留 `networkingMode=consomme`（WSL 2.x 上即 `virtioproxy`，数据面正确性不可替代），把 **DNS 完整搬离合成网关**，改由 systemd-resolved 以真实可路由的 DNS 服务器为上游。内网查询与内网业务流量因此走同一条被 aTrust 认可的主机 socket 路径，且没有引入新组件。

### 3.1 定网络模式，再改三处解析配置

先确认网络模式，写在 Windows 用户目录的 `.wslconfig` 里：

```ini
[wsl2]
networkingMode=consomme
```

WSL 3.x 写 `consomme`，WSL 2.x 写 `virtioproxy`，两者等价（见 2.2）。改哪个名字，后面的配置都一样。改完 `.wslconfig` 要执行一次 `wsl --shutdown`；发行版里那三处靠重启 WSL 或重启 systemd-resolved 生效。

其余三处在发行版里。先改 `/etc/wsl.conf`：

```ini
[boot]
systemd=true

[network]
generateResolvConf=false
```

`[boot]` 这段是前提：systemd-resolved 由 systemd 拉起，没有 systemd 就没有 `127.0.0.53` 这个 stub，后面的步骤全部落空。`[network]` 那段让 WSL 不再生成 `resolv.conf`，免得它和 systemd-resolved 抢同一个文件。

改完执行 `wsl --shutdown`，重进发行版确认 systemd 在跑：`ps -p 1 -o comm=` 输出 `systemd` 才算成了。`[boot] systemd=true` 从 WSL 0.67.6 起支持 [上游]，2.x 与 3.x 都在这个范围内。

然后把 `/etc/resolv.conf` 做成指向 stub 的符号链接：

```shell
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
ls -l /etc/resolv.conf
```

期望输出 `resolv.conf -> /run/systemd/resolve/stub-resolv.conf`。用链接而不是普通文件，是因为普通文件会被各种工具重写，`wsl --shutdown` 之后就没了。

`-f` 会直接覆盖已存在的文件，不用先 `rm`；万一报 `Operation not permitted` 或 `Device or resource busy`，先解掉 `chattr +i` 或挂载再执行。

最后写 `/etc/systemd/resolved.conf`，第一项填公司内网 DNS：

```ini
[Resolve]
# 内网 DNS 放在第一位，它同时负责内网域与（若支持递归）公网域
DNS=<内网DNS地址> 223.5.5.5 8.8.8.8
Domains=~.
```

`Domains=~.` 让这个全局 scope 成为所有域名的默认归属，即“什么域名都问我”。内网 DNS 地址从 Windows 侧 aTrust 适配器上取（`ipconfig /all` 里那一行），取不到就找 IT 要。

**可选：** 想让“断开期间查过、重连后立刻要通”不留空窗，可以再加一行 `Cache=no-negative`。它**不是必需项**，默认 `yes` 会把 NXDOMAIN 一起缓存，TTL 跟内网 DNS 的 SOA 走；不加也能用，只是重连后要等负缓存过期，或者手动 `sudo resolvectl flush-caches` 清掉。3.4 的 A/B 对照会在断连时查内网域名，不加这行时，验收里“重连后自动恢复”这一条可能要多等一会儿。

改完执行一次 `sudo systemctl restart systemd-resolved` 让它生效。

**注意：** 要用单标签短名（如 `intranet`），`Domains=` 里必须再加一个**不带 `~` 前缀**的条目，它同时充当搜索域，例如 `Domains=~. corp.example.com`。只写 `~.` 不解决短名问题。

### 3.2 为什么是混合上游，而不是按域分服务器

`DNS=` 里既有内网 DNS 又有公网 DNS，属于“混合上游”。另一种更精确的写法是按域分服务器，先对比再决定：

| | 混合上游（`DNS=内网 公网`，单 scope） | 分域（`Domains=~corp.example.com` 加独立服务器） |
|---|---|---|
| 实现难度 | `resolved.conf` 三行 | 需要链路级配置（`resolvectl dns <link>` 或 networkd 的 `.network`） |
| 在 WSL 的稳定性 | 高 | 低：链路名由 Consomme 生成，随机器而异 |
| 表达力 | 无法表达“公网域绝不问内网 DNS” | 可以精确表达 |

选混合上游有两个理由。第一，**单 scope 下 resolved 无法把“域 → 服务器”绑起来**：`Domains=` 只决定归属哪个 scope，不决定问哪台服务器；真分域要下沉到链路级配置，在链路名随机器变化的 WSL 里不划算。第二，**它的前置条件可判定**：不连 VPN，直接向那台内网 DNS 查一个公网域名。

```shell
dig @<内网DNS地址> example.com A +short
```

返回正确 A 记录，说明它是递归或转发型，混合方案最优；返回 NXDOMAIN 或 REFUSED，说明它是权威型，混合列表会让公网域间歇性失败，应改用本地转发器（dnsmasq 或 unbound：内网域 forward 到内网 DNS，其余 forward 到公网 DNS）。

### 3.3 请求怎么走：一条不碰拦截状态机的链路

改完之后，内网查询与内网业务流量走同一条路径，全程不依赖 Consomme 的 DNS 拦截 [推断，但与实测一致]：

```text
glibc/NSS
  → 127.0.0.53（systemd-resolved stub，回环，与链路和路由无关）
  → resolved 向上游发普通 UDP/TCP 53
  → Consomme 在主机侧用主机 socket 代发起
  → aTrust 看到"本机进程发起的内网 DNS 查询"，按隧道与身份策略放行
  → 内网 DNS 应答
```

这四段各有分工：`127.0.0.53` 是回环地址，链路抖动、路由变化、`resolv.conf` 被重写都不再影响解析；符号链接管住“重启后又坏”；真实上游让“合成地址不可路由”不再发生；混合上游补上“内网 DNS 不覆盖公网域”的空白。

### 3.4 验证与验收

三组自检加五条验收，全过才算解决。先看解析器状态：

```shell
resolvectl status
```

关注三处：`resolv.conf mode: stub`、`DNSSEC=no (unsupported)`、当前链路 `Default Route: no`。`no` 是对的 —— 默认路由由全局 scope 承担。

再做 A/B 对照，这一组证明 DNS 与 VPN 已经解耦。断开 aTrust：

```shell
getent hosts example.com      # 公网：应成功
getent hosts <内网域名>       # 内网：应失败（无隧道时失败才是正确的）
```

用 `getent` 而不是 `nslookup`：`getent` 走 glibc 的 NSS，和浏览器、`curl`、`pip` 看到的是同一条路；`nslookup` 自带解析器直连 53 端口，绕过 NSS 与 `/etc/hosts`，容易出现“nslookup 能查到、程序照样报错”的假象。

重连 aTrust 后跑同样两条命令，内网域名应当自己恢复，**不需要手改任何文件**。最后在 WSL 里访问一次内网业务系统（`git clone` 内网仓库或 `curl` 内网接口），把域名与 TCP 两条链路一起验。

验收五条：`resolv.conf` 指向 `127.0.0.53` 且 `wsl --shutdown` 后不变；断开 aTrust 时内网域名失败、**公网域名照常解析**；重连后内网域名与业务系统自动恢复；FQDN 与单标签短名都能解析；容器内结果与发行版一致。

这套配置在装了 aTrust 的机器上跑下来，五条全部满足：`wsl --shutdown` 之后配置没有退回，断开与重连 aTrust 也不需要手改任何文件。

平时再遇到“昨天还好、今天访问不了”，先看解析服务在不在：`systemctl is-active systemd-resolved` 应返回 `active`；返回 `inactive` 或 `failed` 就执行 `sudo systemctl restart systemd-resolved`，大部分“突然解析不了”到这一步就恢复了。

## 4. 其它方案为什么不适用

网上能搜到的做法逐条对照如下。这张表也说明第 2 章为什么要分层拆：多数方案只修其中一层。

### 4.1 四种做法为什么都不是首选

| 方案 | 机制 | 为什么不适用 |
|---|---|---|
| **mirrored 网络模式** | 把 Windows 网卡镜像进 Guest，包以转发身份注入主机栈 | 包到了，身份没到。aTrust 对无法归属的流量丢弃或不重定向。**已实测无效** |
| **旧 NAT 模式（HNS/vmswitch NAT）** | IP 级转发，Guest 源地址是 `vEthernet (WSL)` 网段 | 无进程身份；源地址不在 aTrust 可达网段（martian 与非对称回程）；双层封装导致 PMTU 黑洞。**已实测无效** |
| **wsl-vpnkit 或自建转发** | 在 Windows 侧把 WSL 流量重新发起 | 机制同类、可用，但在已有原生转发上再叠一层。代价见 4.2 |
| **在 WSL/Docker 里跑 aTrust Linux 客户端**（docker-easyconnect / aTrustLogin） | 容器内起 tun 隧道，对外给 socks5/http 或做网关 | 依赖 `/dev/net/tun`；有终端准入、单会话互斥、SSO/GUI 等障碍，**合规风险最高**。属备选而非首选 |

另外几种做法只提一句：继续调 `dnsTunneling` 已被 2.4 否证；在 Guest 手工加路由只是把“超时”变成“丢包”（aTrust 不封装主机转发的第三方流量）；WSL 1 机制上更容易通，但开发环境整体降级；Windows 侧代理只覆盖代理感知的客户端，裸 TCP/UDP 不管。

验证一步：表里的“已实测无效”指 mirrored 与旧 NAT 两行，`dnsTunneling` 的否证见 2.4；其余各行靠第 2 章的机制推理判掉，属推断而非实测，客户端版本不同时值得重跑。

### 4.2 为什么不自己搭一层转发

不装 wsl-vpnkit 不等于没得选：可以自建，Windows 侧起 gvproxy 或 socat、WSL 侧配代理，或开一张 NAT 网卡。都能通，问题是**它们解决的需求本来已经被满足了**。

Consomme 本身就是 WSL 的 socket 级转发（2.2）：Guest 的 `connect()` 已在主机侧由 Windows 进程重新发起。再搭一层，就是在同一段路径上叠第二、第三次转发，换来的三笔账：

- **数据路径多一个责任方。** 转发进程能看到每条连接的目的地址、端口与时间，也能看到未加密协议的内容；TLS 内容它不解密，但元数据照旧可见。
- **供应链多一个维护方。** 别人的工具或自己写的脚本，版本、补丁、漏洞响应都要自己跟；公司内自维的那套还要额外准入与交接成本。
- **审计多一份说明。** 数据面每多一个组件，“谁能看到什么”就要多答一次。

本文的方案没有新增组件：网络模式是 WSL 自带的，DNS 用发行版自带的 systemd-resolved，配置就是 `.wslconfig` 与三个系统文件 —— **责任方与可见面都停在原有组件之内** [推断]，而不是“原生组件没有漏洞”。

**注意：** 若 IT 已统一审批并维护某套转发工具，后两笔账不成立；若 aTrust 把 `wslservice` 判为不可信进程，原生方案同样失效（见 5）。还有一种情况**必须**加一层：内网 DNS 是权威型、不递归，而内外网域名要并存。这时只在发行版内用 dnsmasq 或 unbound 做一次分域 forward（见 3.2），不要放到 Windows 侧，那会把 DNS 推回“主机解析器替 WSL 作答”的老路。

验证一步：确认数据面上没有多余的转发进程。

```shell
ps -eo comm | grep -iE 'gvproxy|vpnkit|socat'   # WSL 侧：应无输出
powershell.exe -NoProfile -c "Get-Process gvproxy,vpnkit,socat -ErrorAction SilentlyContinue"
```

第二条走 WSL interop 查 Windows 侧（本文的 `wsl.conf` 只关了 `appendWindowsPath`，interop 仍开）。两条都空，说明只有 WSL 与发行版自带的组件在参与。

## 5. 边界与残余风险

下面几条是方案失效的条件，对着看能判断自己会不会撞上。

| 风险 | 说明 |
|---|---|
| 短名解析 | `Domains=~.` 不提供搜索域，短名必须显式配搜索域才可用（见 3.1） |
| 容器 | 容器不继承 resolved，自带 `resolv.conf`；需要 Docker `daemon.json` 的 `dns` 项或直接指向内网解析器，否则“本体通、容器不通” |
| 进程白名单 | 若 aTrust 把授权收紧到服务进程级（`wslservice`、`wsldevicehost` 不在列表），本方案会与所有“走主机”方案一起失效；此时只能走 IT 审批的 Linux 客户端路线 |
| 版本 | 2.x 写 `networkingMode=virtioproxy`、3.x 写 `consomme`，同一后端；3.0.4 以上能拿到 #42026，更新的版本能拿到 #42180，可减少“莫名其妙全灭”的干扰，但**不改变本方案的任何结论**，2.x 也不必为此升级 |
| 内网 DNS 类型 | 混合上游依赖其具备递归能力，判定方法见 3.2 |

## 6. 总结

零信任客户端要的是“有身份的发起者”，不是“能路由的包”。判断一个 WSL 网络方案能不能救 aTrust，先看流量由哪个进程发起，再看它是否经过客户端的钩子。“访问不通”常常有两处独立断点，网络模式与 DNS 要分开验、分开修；先分层（归属、数据面、DNS）再动手，比整体换方案省事。

在企业环境里，转发层能不加就不加：原生已有的那一层够用，再叠一层只换来更多责任方。碰到开关看起来正确却无效时，先写出它的**作用域**，再查它和相关修复的合入时间。排查顺序固定为：先问有没有搜索域，再问上游是谁，最后才怀疑 VPN。

## 参考文章

[1] [WSL 网络配置官方文档](https://learn.microsoft.com/en-us/windows/wsl/networking)

[2] [WSL 排错文档（VPN、GSA 与 mirrored 的不兼容说明）](https://learn.microsoft.com/en-us/windows/wsl/troubleshooting)

[3] [OpenVMM Consomme 网络后端参考](https://openvmm.dev/guide/reference/backends/consomme.html)

[4] [microsoft/WSL #14461：为 VirtioProxy 打通 DNS tunneling](https://github.com/microsoft/WSL/pull/14461)

[5] [microsoft/WSL #40873：VirtioProxy 改名为 Consomme](https://github.com/microsoft/WSL/pull/40873)

[6] [microsoft/WSL #42180：Consomme 补上 DNS 后缀 watcher](https://github.com/microsoft/WSL/pull/42180)

[7] [microsoft/openvmm #2633：Consomme 的 DNS 解析器实现](https://github.com/microsoft/openvmm/pull/2633)

[8] [wsl-vpnkit：备选方案](https://github.com/sakai135/wsl-vpnkit)
