# 快速开始指南

## 概述

黑客Skill是一个全面的网络安全技能库，包含50个核心模块，涵盖渗透测试、安全研究、漏洞挖掘等各个领域。

## 平台使用指南

### 🎯 在 Kiro 中使用

#### 方法1：放置在steering目录（推荐）
```bash
# 将整个hacker-skill文件夹复制到Kiro的steering目录
cp -r hacker-skill ~/.kiro/steering/
```

**自动加载**：
- `01-core/core.md` 会自动加载，建立基础认知

**手动引用**：
在对话中使用文件路径引用特定模块：
```
请参考 ~/.kiro/steering/hacker-skill/03-web-exploit/web-exploit.md 帮我分析这个SQL注入漏洞
```

#### 方法2：项目级配置
将hacker-skill放在项目的`.kiro/steering/`目录下，仅在该项目中生效。

### 🎨 在 Cursor 中使用

#### 方法1：.cursorrules文件
在项目根目录创建`.cursorrules`文件：
```
# 引用黑客Skill
请参考以下安全技能库：
- 核心配置：./hacker-skill/01-core/core.md
- Web安全：./hacker-skill/03-web-exploit/web-exploit.md
- 内网渗透：./hacker-skill/11-internal/internal.md
```

#### 方法2：项目配置
1. 将hacker-skill文件夹放在项目目录
2. 在对话中直接引用：
```
@hacker-skill/03-web-exploit/web-exploit.md 帮我分析这个漏洞
```

### 💬 在 Claude Code 中使用

#### 方法1：项目设置
1. 将hacker-skill文件夹放在项目目录
2. 在项目设置中添加上下文文件
3. 选择需要的skill文件

#### 方法2：对话引用
在对话中直接引用文件：
```
请参考 hacker-skill/03-web-exploit/web-exploit.md 中的SQL注入技术
```

---

## 使用示例

### 示例1：Web渗透测试

**场景**：对一个Web应用进行渗透测试

**引用的Skill**：
```
1. 信息收集：hacker-skill/02-recon/recon.md
2. Web漏洞：hacker-skill/03-web-exploit/web-exploit.md
3. SQL注入：hacker-skill/03-web-exploit/sql-injection.md
4. Webshell：hacker-skill/06-webshell/webshell.md
```

**对话示例**：
```
我需要对 http://target.com 进行渗透测试，请参考以下skill：
- hacker-skill/02-recon/recon.md
- hacker-skill/03-web-exploit/web-exploit.md

帮我制定测试计划并开始信息收集。
```

### 示例2：内网渗透

**场景**：已获得初始权限，需要进行内网渗透

**引用的Skill**：
```
1. 后渗透：hacker-skill/09-post-exploit/post-exploit.md
2. AD攻击：hacker-skill/10-active-directory/active-directory.md
3. 内网渗透：hacker-skill/11-internal/internal.md
4. Windows安全：hacker-skill/24-windows/windows.md
```

**对话示例**：
```
我已经获得了一台Windows主机的权限，请参考：
- hacker-skill/09-post-exploit/post-exploit.md
- hacker-skill/10-active-directory/active-directory.md

帮我进行权限提升和域渗透。
```

### 示例3：Webshell免杀

**场景**：需要生成免杀的PHP Webshell

**引用的Skill**：
```
1. Webshell免杀：hacker-skill/06-webshell/webshell.md
2. PHP Webshell：hacker-skill/06-webshell/php-webshell.md
```

**对话示例**：
```
请参考 hacker-skill/06-webshell/php-webshell.md 生成一个免杀的PHP Webshell，
要求：
- 避免使用eval、assert等危险函数
- 使用回调函数和字符串混淆
- 提供3个不同混淆级别的版本
```

### 示例4：红队行动

**场景**：进行红队演练

**引用的Skill**：
```
1. 社工钓鱼：hacker-skill/15-social-engineering/social-engineering.md
2. C2免杀：hacker-skill/07-c2-evasion/c2-evasion.md
3. 红队战术：hacker-skill/44-redteam/redteam.md
4. 防御对抗：hacker-skill/50-defense-evasion/defense-evasion.md
```

**对话示例**：
```
我需要进行红队演练，目标是测试企业的检测和响应能力。
请参考：
- hacker-skill/44-redteam/redteam.md
- hacker-skill/45-red-team-ops/red-team-ops.md

帮我设计攻击链和C2基础设施。
```

### 示例5：移动应用测试

**场景**：Android应用安全测试

**引用的Skill**：
```
1. 移动安全：hacker-skill/14-mobile/mobile.md
2. 代码审计：hacker-skill/30-code-audit/code-audit.md
3. 业务逻辑：hacker-skill/47-business-logic/business-logic.md
```

**对话示例**：
```
我需要测试一个Android应用，请参考：
- hacker-skill/14-mobile/mobile.md

帮我分析APK文件，查找潜在的安全漏洞。
```

---

## 常见问题

### Q1：如何选择合适的Skill？
**A**：参考 [INDEX.md](INDEX.md) 中的场景推荐和学习路径，根据实际需求选择。

### Q2：可以同时引用多个Skill吗？
**A**：可以，建议根据任务复杂度引用2-5个相关Skill。

### Q3：Skill会自动更新吗？
**A**：需要手动更新。建议定期从仓库拉取最新版本。

### Q4：如何贡献新的Skill？
**A**：可以在对应模块文件夹下添加子skill文件，遵循现有格式。

### Q5：Skill支持中文吗？
**A**：支持。AI会根据用户语言自动调整回复语言。

---

## 最佳实践

### ✅ 推荐做法
1. **明确目标**：清楚说明你要做什么
2. **引用相关Skill**：选择2-5个最相关的模块
3. **提供上下文**：说明目标环境和限制条件
4. **分步执行**：复杂任务分解成多个步骤
5. **验证结果**：要求AI提供可执行的命令和代码

### ❌ 避免做法
1. 一次引用过多Skill（超过5个）
2. 引用不相关的Skill
3. 没有明确的目标和需求
4. 期望AI直接执行攻击（AI只提供技术指导）

---

## 进阶技巧

### 技巧1：组合使用
```
我需要进行完整的Web渗透测试，请按以下顺序参考skill：
1. hacker-skill/02-recon/recon.md - 信息收集
2. hacker-skill/03-web-exploit/web-exploit.md - 漏洞利用
3. hacker-skill/09-post-exploit/post-exploit.md - 后渗透
```

### 技巧2：深度引用
```
请深入参考 hacker-skill/10-active-directory/active-directory.md 中的
Kerberos攻击部分，帮我进行Kerberoasting攻击。
```

### 技巧3：场景化请求
```
场景：我是红队成员，目标是测试企业的AD安全。
当前状态：已获得一台域内主机的普通用户权限。
目标：获取域管理员权限。

请参考以下skill制定攻击计划：
- hacker-skill/10-active-directory/active-directory.md
- hacker-skill/24-windows/windows.md
```

---

## 快速链接

- [完整索引](INDEX.md) - 所有模块的快速导航
- [结构说明](STRUCTURE.md) - 完整的文件结构
- [自检报告](AUDIT-REPORT.md) - 质量检查和优化建议
- [README](README.md) - 项目说明

---

**开始使用**：选择一个场景，引用相关的Skill，开始你的安全研究之旅！
