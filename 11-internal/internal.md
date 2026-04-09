# 内网渗透

内网渗透是获得初始访问后，在内部网络中进行的深入渗透和横向移动。

## 内网信息收集

### 网络拓扑
- 网络段划分
- 路由信息
- 网关和 DNS
- VLAN 信息
- 网络设备

### 主机发现
- ARP 扫描
- ICMP 扫描
- TCP/UDP 扫描
- NetBIOS 扫描
- 被动发现

### 服务识别
- 端口扫描
- 服务指纹
- 版本探测
- 漏洞识别

### 域环境
- 域控位置
- 域用户和组
- 域信任关系
- 组策略
- 域计算机

### 敏感资产
- 文件服务器
- 数据库服务器
- 邮件服务器
- Web 应用
- 管理系统
- 备份系统

## 隧道与代理

### 端口转发

#### SSH 隧道
- 本地端口转发
- 远程端口转发
- 动态端口转发
- SSH 跳板

#### 其他转发
- Netsh 端口转发
- Iptables 转发
- Socat 转发
- Chisel

### Socks 代理
- Socks4/Socks5
- Proxychains
- Regeorg
- Neo-reGeorg
- Venom

### VPN 隧道
- OpenVPN
- WireGuard
- IPSec
- PPTP/L2TP

### 应用层隧道
- DNS 隧道（dnscat2、iodine）
- ICMP 隧道（icmpsh、ptunnel）
- HTTP 隧道
- WebSocket 隧道

## 横向移动

### Windows 横向

#### 远程执行
- PsExec
- WMI
- DCOM
- WinRM
- PowerShell Remoting
- SC（服务控制）
- AT/SCHTASKS（计划任务）

#### 凭证传递
- Pass-the-Hash
- Pass-the-Ticket
- Overpass-the-Hash
- Pass-the-Key

#### RDP
- RDP 连接
- RDP 劫持
- Sticky Keys 后门

### Linux 横向
- SSH
- 密钥认证
- 跳板机
- 自动化脚本

### 漏洞利用
- MS17-010（永恒之蓝）
- Zerologon
- PrintNightmare
- 其他内网漏洞

## 权限维持

### 多点部署
- 多台主机植入
- 不同权限级别
- 不同持久化方式
- 备用通道

### 隐蔽通道
- 正常业务流量
- 加密通信
- 协议伪装
- 定时回连

## 内网漏洞利用

### 常见漏洞
- 永恒之蓝系列
- Zerologon
- PrintNightmare
- ProxyLogon/ProxyShell
- Log4j

### 漏洞扫描
- Nessus
- OpenVAS
- 自定义扫描脚本
- 漏洞验证

## 数据收集

### 敏感信息
- 文件共享
- 数据库
- 邮件系统
- 源代码
- 配置文件
- 凭证信息

### 业务系统
- ERP 系统
- OA 系统
- 财务系统
- 客户数据
- 核心业务数据

## 内网工具

### 信息收集
- fscan
- LadonGo
- Nmap
- Masscan

### 漏洞利用
- Metasploit
- Cobalt Strike
- Empire
- Impacket

### 代理隧道
- Proxychains
- Regeorg
- Chisel
- Frp

---

内网渗透需要谨慎操作，避免触发告警和影响业务，同时保持多条退路。
