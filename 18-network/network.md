# 网络攻击与协议安全

网络层面的攻击技术，包括中间人攻击、协议漏洞、流量劫持等。

## 中间人攻击（MITM）

### ARP 欺骗
- ARP 缓存投毒
- 网关欺骗
- 双向欺骗
- 流量嗅探和篡改

### DNS 劫持
- 本地 DNS 劫持
- 路由器 DNS 劫持
- DNS 缓存投毒
- DNS 响应欺骗

### DHCP 攻击
- DHCP 饥饿攻击
- 恶意 DHCP 服务器
- DHCP Snooping 绕过

### SSL/TLS 中间人
- SSL Strip
- 证书伪造
- 证书信任劫持
- HSTS 绕过

## 协议攻击

### TCP/IP 协议
- SYN Flood
- TCP 会话劫持
- TCP 重置攻击
- IP 欺骗
- 分片攻击

### ICMP 协议
- ICMP Flood
- Ping of Death
- Smurf 攻击
- ICMP 重定向

### 路由协议
- RIP 攻击
- OSPF 攻击
- BGP 劫持
- 路由黑洞

## DDoS 攻击

### 流量型攻击
- UDP Flood
- ICMP Flood
- SYN Flood
- ACK Flood

### 应用层攻击
- HTTP Flood
- Slowloris
- RUDY
- DNS 查询 Flood

### 反射放大攻击
- DNS 放大
- NTP 放大
- SSDP 放大
- Memcached 放大

## 流量劫持

### BGP 劫持
- 前缀劫持
- 子前缀劫持
- 路径劫持

### CDN 劫持
- 域名劫持
- CNAME 劫持
- 子域名接管

### 会话劫持
- Cookie 劫持
- Session 劫持
- Token 劫持

## 网络嗅探

### 被动嗅探
- 混杂模式
- 端口镜像
- TAP 设备

### 主动嗅探
- ARP 欺骗
- MAC Flooding
- DHCP 欺骗

### 协议分析
- Wireshark
- tcpdump
- tshark
- 自定义脚本

## VPN 攻击

### VPN 协议漏洞
- PPTP 漏洞
- L2TP 漏洞
- IPSec 漏洞
- OpenVPN 漏洞

### VPN 配置错误
- 弱加密
- 默认配置
- 认证绕过
- 流量泄露

## 邮件协议

### SMTP 攻击
- 邮件伪造
- SPF/DKIM/DMARC 绕过
- 开放中继
- 邮件炸弹

### POP3/IMAP 攻击
- 凭证爆破
- 中间人攻击
- 邮件窃取

## 隧道技术

### 正向隧道
- SSH 隧道
- HTTP 隧道
- SOCKS 代理

### 反向隧道
- 反向 SSH
- 反向 HTTP
- 反向 SOCKS

### 隐蔽隧道
- DNS 隧道
- ICMP 隧道
- HTTP/HTTPS 隧道

## 反幻觉机制

### 技术准确性
- 协议描述准确
- 攻击原理正确
- 工具使用准确
- 不编造攻击技术

---

网络攻击需要深入理解网络协议和机制，建议在授权环境中进行测试。
