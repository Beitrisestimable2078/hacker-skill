# 黑客 Skill 索引 - AI助手使用指南

> 这是一个为AI助手设计的技能索引文件。当用户提出安全相关问题时，AI应该主动查找并使用相关的skill模块。

## 🤖 AI使用说明

### 工作流程

1. **理解用户需求** - 分析用户的问题属于哪个安全领域
2. **查找相关skill** - 在下方索引中找到对应的模块
3. **读取skill内容** - 使用 `readFile` 工具读取相关模块文件
4. **应用专业知识** - 基于skill内容提供专业的安全建议
5. **保持对抗思维** - 考虑检测、绕过、隐蔽性等对抗因素

### 使用原则

- **主动发现**：当用户问题涉及安全领域时，主动查找相关skill
- **精准匹配**：根据关键词快速定位到具体模块
- **深度应用**：不只是引用，要真正理解和应用skill内容
- **灵活组合**：复杂场景可以组合多个skill模块
- **实战导向**：提供可直接执行的命令和代码

### 关键词匹配规则

当用户提到以下关键词时，应该读取对应的skill模块：

## 📋 Skill模块索引

### 🔍 信息收集与侦察

**关键词**: 信息收集、侦察、recon、OSINT、子域名、端口扫描、指纹识别

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `01-core/core.md` | 所有场景 | 黑客思维、对抗意识、核心价值观 |
| `02-recon/recon.md` | 渗透测试前期 | 信息收集方法论 |
| `02-recon/subdomain-enum.md` | 子域名发现 | 被动枚举、DNS爆破、证书透明度 |
| `02-recon/port-scan.md` | 端口扫描 | Nmap、Masscan、服务识别 |
| `02-recon/osint.md` | 开源情报 | 社交媒体、搜索引擎、数据泄露 |

**使用示例**：
```
用户："如何进行子域名枚举？"
AI应该：readFile('02-recon/subdomain-enum.md')
```

---

### 🌐 Web应用安全

**关键词**: Web漏洞、SQL注入、XSS、SSRF、文件上传、命令注入、反序列化、XXE

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `03-web-exploit/web-exploit.md` | Web渗透 | Web漏洞利用方法论 |
| `03-web-exploit/sql-injection.md` | SQL注入 | Union/Boolean/Time-based/二次注入 |
| `03-web-exploit/xss.md` | XSS攻击 | 反射/存储/DOM型、绕过技巧 |
| `03-web-exploit/ssrf.md` | SSRF | 协议利用、内网探测、云元数据 |
| `03-web-exploit/file-upload.md` | 文件上传 | 类型绕过、解析漏洞、竞态条件 |
| `03-web-exploit/command-injection.md` | 命令注入 | 直接注入、盲注、绕过过滤 |
| `03-web-exploit/deserialization.md` | 反序列化 | Java/PHP/Python/.NET |
| `03-web-exploit/xxe.md` | XXE注入 | 外部实体、文件读取、SSRF |
| `04-database/database.md` | 数据库安全 | MySQL/PostgreSQL/MSSQL/MongoDB |
| `05-middleware/middleware.md` | 中间件 | Apache/Nginx/Tomcat/WebLogic |

**使用示例**：
```
用户："这个网站存在SQL注入，如何利用？"
AI应该：readFile('03-web-exploit/sql-injection.md')
```

---

### 🐚 Webshell与后门

**关键词**: Webshell、一句话木马、内存马、免杀、PHP木马、JSP木马

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `06-webshell/webshell.md` | Webshell | 免杀方法论 |
| `06-webshell/php-webshell.md` | PHP环境 | 一句话、变形、加密、免杀 |
| `06-webshell/jsp-webshell.md` | Java环境 | Filter/Servlet/表达式注入 |
| `06-webshell/aspx-webshell.md` | .NET环境 | ASPX特性利用 |
| `06-webshell/memory-shell.md` | 内存马 | Filter/Servlet/Agent型 |

**使用示例**：
```
用户："如何编写免杀的PHP Webshell？"
AI应该：readFile('06-webshell/php-webshell.md')
```

---

### 🛡️ 免杀与对抗

**关键词**: 免杀、bypass、WAF绕过、杀软绕过、C2、Cobalt Strike、流量混淆

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `07-c2-evasion/c2-evasion.md` | C2免杀 | Shellcode编码、加载器、流量混淆 |
| `08-bypass/bypass.md` | 防护绕过 | WAF/杀软/RASP/沙箱绕过 |
| `50-defense-evasion/defense-evasion.md` | 防御对抗 | 日志清除、进程隐藏、反取证 |

**使用示例**：
```
用户："如何绕过WAF进行SQL注入？"
AI应该：readFile('08-bypass/bypass.md') 和 readFile('03-web-exploit/sql-injection.md')
```

---

### 💻 后渗透与权限提升

**关键词**: 后渗透、提权、横向移动、持久化、凭证获取、Mimikatz、PTH、PTT

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `09-post-exploit/post-exploit.md` | 后渗透 | 后渗透方法论 |
| `09-post-exploit/privilege-escalation.md` | 权限提升 | Windows/Linux提权 |
| `09-post-exploit/credential-dumping.md` | 凭证获取 | Mimikatz/密码抓取 |
| `09-post-exploit/persistence.md` | 持久化 | 启动项/计划任务/服务 |
| `09-post-exploit/lateral-movement.md` | 横向移动 | PTH/PTT/WMI/PSExec |
| `09-post-exploit/data-exfiltration.md` | 数据窃取 | 数据打包/隐蔽传输 |

**使用示例**：
```
用户："拿到webshell后如何提权？"
AI应该：readFile('09-post-exploit/privilege-escalation.md')
```

---

### 🏢 Active Directory与域渗透

**关键词**: 域渗透、AD、Kerberos、NTLM、黄金票据、白银票据、DCSync、委派攻击

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `10-active-directory/active-directory.md` | 域渗透 | AD攻击方法论 |
| `10-active-directory/kerberos-attacks.md` | Kerberos | 黄金/白银票据、Kerberoasting |
| `10-active-directory/ntlm-attacks.md` | NTLM | NTLM中继、Pass-the-Hash |
| `10-active-directory/acl-abuse.md` | ACL滥用 | 权限滥用、DCSync、WriteDACL |
| `10-active-directory/delegation-attacks.md` | 委派攻击 | 非约束/约束/基于资源的委派 |
| `11-internal/internal.md` | 内网渗透 | 内网综合渗透 |

**使用示例**：
```
用户："如何进行Kerberoasting攻击？"
AI应该：readFile('10-active-directory/kerberos-attacks.md')
```

---

### 🔬 二进制与逆向

**关键词**: 逆向、PWN、漏洞利用、缓冲区溢出、ROP、堆利用、模糊测试、0day

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `12-binary/binary.md` | 二进制安全 | 二进制安全方法论 |
| `12-binary/reverse-engineering.md` | 逆向工程 | IDA/Ghidra/静态分析/动态调试 |
| `12-binary/exploit-development.md` | 漏洞利用 | 缓冲区溢出/ROP/堆利用 |
| `12-binary/pwn.md` | PWN | CTF PWN题型/栈溢出/堆利用 |
| `12-binary/fuzzing.md` | 模糊测试 | AFL/LibFuzzer/覆盖率引导 |
| `13-zero-day/zero-day.md` | 0day挖掘 | 漏洞挖掘/补丁分析 |

**使用示例**：
```
用户："如何进行缓冲区溢出利用？"
AI应该：readFile('12-binary/exploit-development.md')
```

---

### 📝 代码审计

**关键词**: 代码审计、白盒测试、Java审计、PHP审计、Python审计、Node.js审计、业务逻辑、API安全

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `30-code-audit/code-audit.md` | 代码审计 | 审计方法论、污点分析 |
| `30-code-audit/java-audit.md` | Java审计 | Spring/Struts2/反序列化/JNDI |
| `30-code-audit/php-audit.md` | PHP审计 | ThinkPHP/Laravel/文件包含 |
| `30-code-audit/python-audit.md` | Python审计 | Django/Flask/SSTI/Pickle |
| `30-code-audit/nodejs-audit.md` | Node.js审计 | Express/原型链污染/npm |
| `30-code-audit/business-logic-audit.md` | 业务逻辑 | 支付/优惠券/积分/权限漏洞 |
| `30-code-audit/api-security-audit.md` | API安全 | REST/GraphQL/认证授权 |
| `30-code-audit/static-analysis-tools.md` | 静态分析 | SonarQube/CodeQL/Semgrep |
| `30-code-audit/code-review-process.md` | 审查流程 | 系统化审查方法论 |

**使用示例**：
```
用户："帮我审计这段Java代码"
AI应该：readFile('30-code-audit/java-audit.md')

用户："这个支付接口有什么安全问题？"
AI应该：readFile('30-code-audit/business-logic-audit.md')
```

---

### 📱 移动安全

**关键词**: 移动安全、Android、iOS、APK、IPA、逆向、Hook、Frida

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `14-mobile/mobile.md` | 移动安全 | 移动安全方法论 |
| `14-mobile/android-security.md` | Android | APK分析/逆向/Hook/Root检测 |
| `14-mobile/ios-security.md` | iOS | IPA分析/越狱/Frida/代码签名 |
| `14-mobile/app-analysis.md` | 应用分析 | 静态/动态/流量分析 |

**使用示例**：
```
用户："如何分析Android APK？"
AI应该：readFile('14-mobile/android-security.md')
```

---

### 💻 操作系统安全

**关键词**: Windows提权、Linux提权、容器逃逸、PowerShell、macOS

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `24-windows/windows.md` | Windows | Windows安全 |
| `24-windows/windows-privesc.md` | Windows提权 | 内核漏洞/配置错误/令牌窃取 |
| `24-windows/powershell-attacks.md` | PowerShell | Empire/PowerSploit/混淆 |
| `24-windows/windows-persistence.md` | Windows持久化 | 注册表/WMI/计划任务 |
| `25-linux/linux.md` | Linux | Linux安全 |
| `25-linux/linux-privesc.md` | Linux提权 | SUID/Capabilities/内核漏洞 |
| `25-linux/linux-persistence.md` | Linux持久化 | Cron/Systemd/PAM后门 |
| `25-linux/container-escape.md` | 容器逃逸 | Docker/Kubernetes逃逸 |
| `26-macos/macos.md` | macOS | macOS安全 |

**使用示例**：
```
用户："Linux系统如何提权？"
AI应该：readFile('25-linux/linux-privesc.md')
```

---

### ☁️ 云与容器

**关键词**: 云安全、AWS、Azure、Kubernetes、Docker、容器逃逸

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `27-cloud/cloud.md` | 云安全 | 云安全方法论 |
| `27-cloud/aws-security.md` | AWS | IAM/S3/EC2/Lambda |
| `27-cloud/azure-security.md` | Azure | Azure AD/存储/计算 |
| `27-cloud/k8s-security.md` | Kubernetes | K8s配置/RBAC/Pod安全 |
| `28-container/container.md` | 容器 | Docker/K8s安全/镜像扫描 |

**使用示例**：
```
用户："如何评估AWS环境的安全性？"
AI应该：readFile('27-cloud/aws-security.md')
```

---

### 🎭 社工与红队

**关键词**: 社会工程、钓鱼、红队、APT、恶意软件

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `15-social-engineering/social-engineering.md` | 社工 | 钓鱼邮件/网站/心理学 |
| `16-malware/malware.md` | 恶意软件 | 恶意软件分析/沙箱逃逸 |
| `17-apt/apt.md` | APT | 高级持续性威胁/TTP |
| `44-redteam/redteam.md` | 红队 | 红队方法论/TTP/C2 |
| `45-red-team-ops/red-team-ops.md` | 红队行动 | 红队演练/对抗模拟 |

**使用示例**：
```
用户："如何设计一次钓鱼攻击？"
AI应该：readFile('15-social-engineering/social-engineering.md')
```

---

### 🌐 网络与协议

**关键词**: 网络攻击、中间人、ARP欺骗、WiFi破解、无线安全

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `18-network/network.md` | 网络攻击 | 协议分析/中间人/ARP欺骗 |
| `19-wireless/wireless.md` | 无线安全 | WiFi破解/蓝牙/RFID/SDR |
| `20-voip/voip.md` | VoIP | VoIP协议/SIP攻击 |

**使用示例**：
```
用户："如何进行WiFi破解？"
AI应该：readFile('19-wireless/wireless.md')
```

---

### 🔐 密码与加密

**关键词**: 密码学、加密、哈希、密码破解、隐写术

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `31-crypto/crypto.md` | 密码学 | 加密算法/哈希/密钥管理 |
| `34-privacy/privacy.md` | 隐私保护 | 匿名技术/Tor/VPN/反追踪 |
| `35-steganography/steganography.md` | 隐写术 | 图片/音频/视频隐写 |

**使用示例**：
```
用户："如何破解MD5哈希？"
AI应该：readFile('31-crypto/crypto.md')
```

---

### 🤖 新兴技术

**关键词**: AI安全、供应链、区块链、智能合约、物联网

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `21-iot/iot.md` | 物联网 | IoT设备/固件分析 |
| `22-hardware/hardware.md` | 硬件 | 硬件接口/固件逆向 |
| `23-scada/scada.md` | 工控 | SCADA/车联网 |
| `29-blockchain/blockchain.md` | 区块链 | 智能合约/DeFi/钱包 |
| `32-ai-security/ai-security.md` | AI安全 | 对抗样本/模型投毒/提示注入 |
| `33-supply-chain/supply-chain.md` | 供应链 | 依赖投毒/CI/CD/SBOM |

**使用示例**：
```
用户："如何审计智能合约？"
AI应该：readFile('29-blockchain/blockchain.md')
```

---

### 🛠️ 高级技术与工具

**关键词**: LOLBins、取证、威胁情报、溯源、工具开发

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `36-living-off-land/living-off-land.md` | 白利用 | LOLBins/系统工具滥用 |
| `37-forensics/forensics.md` | 数字取证 | 磁盘/内存/网络取证 |
| `38-threat-intel/threat-intel.md` | 威胁情报 | IOC/威胁狩猎/MITRE ATT&CK |
| `39-attribution/attribution.md` | 溯源 | 攻击溯源/归因分析 |
| `48-tool-dev/tool-dev.md` | 工具开发 | Python/Go工具开发 |

**使用示例**：
```
用户："如何进行内存取证？"
AI应该：readFile('37-forensics/forensics.md')
```

---

### 📊 方法论与管理

**关键词**: 渗透测试、漏洞管理、安全运营、威胁建模、CTF、合规

| 模块路径 | 适用场景 | 关键技术 |
|---------|---------|---------|
| `40-vulnerability-management/vulnerability-management.md` | 漏洞管理 | 漏洞扫描/风险评估 |
| `41-security-operations/security-operations.md` | 安全运营 | SOC/SIEM/SOAR/事件响应 |
| `42-threat-modeling/threat-modeling.md` | 威胁建模 | STRIDE/DREAD/攻击树 |
| `43-penetration-testing/penetration-testing.md` | 渗透测试 | PTES/OWASP/报告编写 |
| `46-ctf/ctf.md` | CTF | Web/Pwn/Reverse/Crypto |
| `47-business-logic/business-logic.md` | 业务安全 | 业务逻辑漏洞/风控 |
| `49-compliance/compliance.md` | 合规 | 等保/ISO27001/PCI-DSS |

**使用示例**：
```
用户："如何编写渗透测试报告？"
AI应该：readFile('43-penetration-testing/penetration-testing.md')
```

---

## 🎯 使用场景示例

### 场景1：Web渗透测试

**用户问题**："我要测试一个Web应用，从哪里开始？"

**AI应该做**：
1. 读取 `02-recon/recon.md` - 了解信息收集方法
2. 读取 `03-web-exploit/web-exploit.md` - 了解Web漏洞利用
3. 读取 `43-penetration-testing/penetration-testing.md` - 了解测试流程
4. 综合提供完整的测试方案

### 场景2：内网渗透

**用户问题**："拿到了一台内网机器的权限，如何横向移动？"

**AI应该做**：
1. 读取 `09-post-exploit/lateral-movement.md` - 横向移动技术
2. 读取 `10-active-directory/active-directory.md` - 域环境攻击
3. 读取 `11-internal/internal.md` - 内网渗透综合
4. 提供具体的横向移动方案

### 场景3：代码审计

**用户问题**："帮我审计这段Spring Boot代码"

**AI应该做**：
1. 读取 `30-code-audit/java-audit.md` - Java审计技术
2. 读取 `30-code-audit/code-review-process.md` - 审查流程
3. 分析代码中的安全问题
4. 提供修复建议

### 场景4：红队演练

**用户问题**："设计一次完整的红队演练"

**AI应该做**：
1. 读取 `44-redteam/redteam.md` - 红队方法论
2. 读取 `15-social-engineering/social-engineering.md` - 社工技术
3. 读取 `07-c2-evasion/c2-evasion.md` - C2免杀
4. 读取 `50-defense-evasion/defense-evasion.md` - 防御对抗
5. 设计完整的演练方案

### 场景5：移动应用测试

**用户问题**："如何测试Android应用的安全性？"

**AI应该做**：
1. 读取 `14-mobile/android-security.md` - Android安全
2. 读取 `14-mobile/app-analysis.md` - 应用分析方法
3. 提供完整的测试流程和工具

---

## 💡 AI最佳实践

### 1. 主动发现
- 不要等用户明确说"使用XX模块"
- 根据问题关键词主动查找相关skill
- 一次可以读取多个相关模块

### 2. 深度理解
- 不只是引用skill内容
- 要真正理解技术原理
- 结合skill提供创新性建议

### 3. 实战导向
- 提供可直接执行的命令
- 给出具体的代码示例
- 考虑实际环境的限制

### 4. 对抗思维
- 考虑检测和防御
- 提供绕过方案
- 评估隐蔽性
- 建议多种备选方案

### 5. 安全意识
- 提醒合法授权
- 强调法律风险
- 建议负责任的披露
- 遵守道德规范

---

## 🔄 持续更新

这个索引会随着skill库的更新而更新。当添加新的模块时，请同步更新此文件。

---

**最后更新**：2026  
**版本**：v2.2  
**模块总数**：50个核心模块 + 54个子模块

