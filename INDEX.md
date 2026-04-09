# 黑客 Skill 快速索引

## 按技术分类导航

### 🔍 信息收集
- [02-recon](02-recon/recon.md) - 信息收集与OSINT
  - [子域名枚举](02-recon/subdomain-enum.md)
  - [端口扫描](02-recon/port-scan.md)
  - [开源情报](02-recon/osint.md)

### 🌐 Web安全
- [03-web-exploit](03-web-exploit/web-exploit.md) - Web漏洞利用
  - [SQL注入](03-web-exploit/sql-injection.md)
  - [XSS跨站脚本](03-web-exploit/xss.md)
  - [SSRF服务端请求伪造](03-web-exploit/ssrf.md)
  - [文件上传](03-web-exploit/file-upload.md)
  - [命令注入](03-web-exploit/command-injection.md)
  - [反序列化](03-web-exploit/deserialization.md)
  - [XXE注入](03-web-exploit/xxe.md)
- [04-database](04-database/database.md) - 数据库安全
- [05-middleware](05-middleware/middleware.md) - 中间件与框架安全
- [06-webshell](06-webshell/webshell.md) - Webshell免杀
  - [PHP Webshell](06-webshell/php-webshell.md)
  - [JSP Webshell](06-webshell/jsp-webshell.md)
  - [ASPX Webshell](06-webshell/aspx-webshell.md)
  - [内存马](06-webshell/memory-shell.md)

### 🛡️ 免杀与绕过
- [07-c2-evasion](07-c2-evasion/c2-evasion.md) - C2免杀
- [08-bypass](08-bypass/bypass.md) - 防护绕过技术
- [50-defense-evasion](50-defense-evasion/defense-evasion.md) - 防御对抗

### 💻 系统渗透
- [09-post-exploit](09-post-exploit/post-exploit.md) - 后渗透
  - [权限提升](09-post-exploit/privilege-escalation.md)
  - [凭证获取](09-post-exploit/credential-dumping.md)
  - [持久化](09-post-exploit/persistence.md)
  - [横向移动](09-post-exploit/lateral-movement.md)
  - [数据窃取](09-post-exploit/data-exfiltration.md)
- [10-active-directory](10-active-directory/active-directory.md) - Active Directory专项
  - [Kerberos攻击](10-active-directory/kerberos-attacks.md)
  - [NTLM攻击](10-active-directory/ntlm-attacks.md)
  - [ACL滥用](10-active-directory/acl-abuse.md)
  - [委派攻击](10-active-directory/delegation-attacks.md)
- [11-internal](11-internal/internal.md) - 内网渗透
- [24-windows](24-windows/windows.md) - Windows安全
  - [Windows提权](24-windows/windows-privesc.md)
  - [PowerShell攻击](24-windows/powershell-attacks.md)
  - [Windows持久化](24-windows/windows-persistence.md)
- [25-linux](25-linux/linux.md) - Linux安全
  - [Linux提权](25-linux/linux-privesc.md)
  - [Linux持久化](25-linux/linux-persistence.md)
  - [容器逃逸](25-linux/container-escape.md)
- [26-macos](26-macos/macos.md) - macOS安全

### 🔬 二进制与逆向
- [12-binary](12-binary/binary.md) - 二进制安全
  - [逆向工程](12-binary/reverse-engineering.md)
  - [漏洞利用开发](12-binary/exploit-development.md)
  - [PWN技术](12-binary/pwn.md)
  - [模糊测试](12-binary/fuzzing.md)
- [13-zero-day](13-zero-day/zero-day.md) - 0day挖掘与研究
- [30-code-audit](30-code-audit/code-audit.md) - 代码审计
  - [Java代码审计](30-code-audit/java-audit.md)
  - [PHP代码审计](30-code-audit/php-audit.md)
  - [Python代码审计](30-code-audit/python-audit.md)
  - [Node.js代码审计](30-code-audit/nodejs-audit.md)
  - [业务逻辑审计](30-code-audit/business-logic-audit.md)
  - [API安全审计](30-code-audit/api-security-audit.md)
  - [静态分析工具](30-code-audit/static-analysis-tools.md)
  - [代码审查流程](30-code-audit/code-review-process.md)

### 📱 移动与物联网
- [14-mobile](14-mobile/mobile.md) - 移动安全
  - [Android安全](14-mobile/android-security.md)
  - [iOS安全](14-mobile/ios-security.md)
  - [应用分析](14-mobile/app-analysis.md)
- [21-iot](21-iot/iot.md) - 物联网安全
- [22-hardware](22-hardware/hardware.md) - 硬件与固件安全
- [23-scada](23-scada/scada.md) - 工控与车联网安全

### 🎭 社工与钓鱼
- [15-social-engineering](15-social-engineering/social-engineering.md) - 社会工程学与钓鱼

### 🦠 恶意软件
- [16-malware](16-malware/malware.md) - 恶意软件
- [17-apt](17-apt/apt.md) - APT攻击

### 🌐 网络与无线
- [18-network](18-network/network.md) - 网络攻击与协议安全
- [19-wireless](19-wireless/wireless.md) - 无线安全与射频
- [20-voip](20-voip/voip.md) - VoIP与通信安全

### ☁️ 云与容器
- [27-cloud](27-cloud/cloud.md) - 云安全
  - [AWS安全](27-cloud/aws-security.md)
  - [Azure安全](27-cloud/azure-security.md)
  - [Kubernetes安全](27-cloud/k8s-security.md)
- [28-container](28-container/container.md) - 容器安全
- [29-blockchain](29-blockchain/blockchain.md) - 区块链与Web3安全

### 🔐 密码与隐私
- [31-crypto](31-crypto/crypto.md) - 密码学与加密
- [34-privacy](34-privacy/privacy.md) - 隐私保护与反追踪
- [35-steganography](35-steganography/steganography.md) - 隐写术与隐蔽信道

### 🤖 新兴技术
- [32-ai-security](32-ai-security/ai-security.md) - AI安全
- [33-supply-chain](33-supply-chain/supply-chain.md) - 供应链安全

### 🛠️ 高级技术
- [36-living-off-land](36-living-off-land/living-off-land.md) - 白利用技术
- [37-forensics](37-forensics/forensics.md) - 数字取证
- [38-threat-intel](38-threat-intel/threat-intel.md) - 威胁情报
- [39-attribution](39-attribution/attribution.md) - 溯源分析

### 📊 运营与管理
- [40-vulnerability-management](40-vulnerability-management/vulnerability-management.md) - 漏洞管理
- [41-security-operations](41-security-operations/security-operations.md) - 安全运营
- [42-threat-modeling](42-threat-modeling/threat-modeling.md) - 威胁建模

### 🎯 实战与竞赛
- [43-penetration-testing](43-penetration-testing/penetration-testing.md) - 渗透测试方法论
- [44-redteam](44-redteam/redteam.md) - 红队战术
- [45-red-team-ops](45-red-team-ops/red-team-ops.md) - 红队行动
- [46-ctf](46-ctf/ctf.md) - CTF竞赛
- [47-business-logic](47-business-logic/business-logic.md) - 业务安全

### 🔧 工具与合规
- [48-tool-dev](48-tool-dev/tool-dev.md) - 安全工具开发
- [49-compliance](49-compliance/compliance.md) - 合规与报告

---

## 常见场景Skill组合推荐

### 🎯 Web渗透测试
1. [02-recon](02-recon/recon.md) - 信息收集
2. [03-web-exploit](03-web-exploit/web-exploit.md) - Web漏洞利用
3. [04-database](04-database/database.md) - 数据库安全
4. [06-webshell](06-webshell/webshell.md) - Webshell免杀
5. [08-bypass](08-bypass/bypass.md) - WAF绕过

### 🏢 内网渗透
1. [09-post-exploit](09-post-exploit/post-exploit.md) - 后渗透
2. [10-active-directory](10-active-directory/active-directory.md) - AD攻击
3. [11-internal](11-internal/internal.md) - 内网渗透
4. [24-windows](24-windows/windows.md) - Windows安全
5. [25-linux](25-linux/linux.md) - Linux安全

### 🔴 红队行动
1. [15-social-engineering](15-social-engineering/social-engineering.md) - 社工钓鱼
2. [07-c2-evasion](07-c2-evasion/c2-evasion.md) - C2免杀
3. [44-redteam](44-redteam/redteam.md) - 红队战术
4. [45-red-team-ops](45-red-team-ops/red-team-ops.md) - 红队行动
5. [50-defense-evasion](50-defense-evasion/defense-evasion.md) - 防御对抗

### 📱 移动应用测试
1. [14-mobile](14-mobile/mobile.md) - 移动安全
2. [03-web-exploit](03-web-exploit/web-exploit.md) - API测试
3. [31-crypto](31-crypto/crypto.md) - 加密分析
4. [47-business-logic](47-business-logic/business-logic.md) - 业务逻辑

### ☁️ 云安全评估
1. [27-cloud](27-cloud/cloud.md) - 云安全
2. [28-container](28-container/container.md) - 容器安全
3. [33-supply-chain](33-supply-chain/supply-chain.md) - 供应链
4. [40-vulnerability-management](40-vulnerability-management/vulnerability-management.md) - 漏洞管理

### 🔍 应急响应与取证
1. [37-forensics](37-forensics/forensics.md) - 数字取证
2. [38-threat-intel](38-threat-intel/threat-intel.md) - 威胁情报
3. [39-attribution](39-attribution/attribution.md) - 溯源分析
4. [16-malware](16-malware/malware.md) - 恶意软件分析

### 🏆 CTF竞赛
1. [46-ctf](46-ctf/ctf.md) - CTF竞赛
2. [03-web-exploit](03-web-exploit/web-exploit.md) - Web题型
3. [12-binary](12-binary/binary.md) - Pwn/Reverse
4. [31-crypto](31-crypto/crypto.md) - Crypto题型

---

## 学习路径建议

### 🌱 初级路径（入门）
1. [02-recon](02-recon/recon.md) - 学习信息收集
2. [03-web-exploit](03-web-exploit/web-exploit.md) - 掌握Web漏洞
3. [43-penetration-testing](43-penetration-testing/penetration-testing.md) - 了解测试方法论
4. [48-tool-dev](48-tool-dev/tool-dev.md) - 学习工具使用

### 🌿 中级路径（进阶）
1. [09-post-exploit](09-post-exploit/post-exploit.md) - 后渗透技术
2. [11-internal](11-internal/internal.md) - 内网渗透
3. [24-windows](24-windows/windows.md) / [25-linux](25-linux/linux.md) - 系统安全
4. [07-c2-evasion](07-c2-evasion/c2-evasion.md) - 免杀技术

### 🌳 高级路径（专家）
1. [10-active-directory](10-active-directory/active-directory.md) - AD攻击
2. [13-zero-day](13-zero-day/zero-day.md) - 漏洞挖掘
3. [17-apt](17-apt/apt.md) - APT攻击
4. [44-redteam](44-redteam/redteam.md) - 红队战术

### 🎯 专项路径（专精）
- **二进制方向**：[12-binary](12-binary/binary.md) → [13-zero-day](13-zero-day/zero-day.md) → [16-malware](16-malware/malware.md)
- **Web方向**：[03-web-exploit](03-web-exploit/web-exploit.md) → [04-database](04-database/database.md) → [47-business-logic](47-business-logic/business-logic.md)
- **移动方向**：[14-mobile](14-mobile/mobile.md) → [21-iot](21-iot/iot.md) → [22-hardware](22-hardware/hardware.md)
- **云安全方向**：[27-cloud](27-cloud/cloud.md) → [28-container](28-container/container.md) → [33-supply-chain](33-supply-chain/supply-chain.md)

---

## 快速查找

### 按关键词查找
- **SQL注入** → [03-web-exploit/sql-injection.md](03-web-exploit/sql-injection.md)
- **提权** → [09-post-exploit](09-post-exploit/post-exploit.md), [24-windows](24-windows/windows.md), [25-linux](25-linux/linux.md)
- **免杀** → [06-webshell](06-webshell/webshell.md), [07-c2-evasion](07-c2-evasion/c2-evasion.md)
- **域渗透** → [10-active-directory](10-active-directory/active-directory.md)
- **容器逃逸** → [28-container](28-container/container.md)
- **钓鱼** → [15-social-engineering](15-social-engineering/social-engineering.md)
- **取证** → [37-forensics](37-forensics/forensics.md)
- **溯源** → [39-attribution](39-attribution/attribution.md)

---

**提示**：点击任何链接即可跳转到对应的skill文件。建议根据实际需求组合使用多个skill。
