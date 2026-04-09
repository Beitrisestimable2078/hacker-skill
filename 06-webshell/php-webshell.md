# PHP Webshell 免杀

PHP Webshell 是最常见的 Webshell 类型，需要针对性的免杀技术。

## 函数替代

### 命令执行替代
避免直接使用 system、exec、shell_exec，使用以下方式：
- 反引号执行
- popen/proc_open
- pcntl_exec
- mail 函数（特定环境）
- imap_open（特定环境）
- error_log（特定环境）

### 代码执行替代
避免直接使用 eval、assert：
- 可变函数
- call_user_func/call_user_func_array
- array_map/array_filter
- usort/uasort/uksort
- preg_replace_callback
- ReflectionFunction

## 字符串混淆

### 编码方式
- base64_encode/base64_decode
- str_rot13
- gzcompress/gzuncompress
- convert_uuencode/convert_uudecode
- 异或加密
- 自定义编码

### 拆分技术
- 字符串拼接
- 字符数组组合
- 变量拼接
- 常量拼接
- 函数返回值拼接

## 回调函数利用

### 数组函数
```php
// 示例思路（不是完整代码）
// 使用 array_map 等函数进行间接调用
// 结合字符串拆分和变量函数
```

### 排序函数
```php
// 示例思路
// usort、uasort 等函数的回调特性
// 动态构造函数名
```

## 反射机制

### ReflectionFunction
```php
// 示例思路
// 使用反射动态调用函数
// 避免直接出现函数名
```

### ReflectionClass
```php
// 示例思路
// 通过反射调用类方法
// 动态实例化和调用
```

## 类封装

### 业务类伪装
- 日志处理类
- 缓存管理类
- 配置读取类
- 数据处理类
- 工具函数类

### 魔术方法
- __construct
- __destruct
- __call
- __get/__set
- __toString

## 文件操作

### 文件函数组合
- fopen + fread/fwrite + fclose
- file_get_contents + file_put_contents
- SplFileObject
- glob/scandir

### 包含技巧
- include/require 动态路径
- 伪协议（php://input、data://、zip://）
- 远程包含（allow_url_include）

## 高级技巧

### 多层加密
- 第一层：base64
- 第二层：gzcompress
- 第三层：异或
- 第四层：自定义算法

### 动态生成
- 运行时生成代码
- 临时文件写入
- 内存执行
- eval 替代方案

### 流量加密
- 请求参数加密
- 响应内容加密
- Cookie 加密传输
- 自定义协议

## 检测对抗

### 静态检测
- 避免关键字出现
- 代码结构变换
- 大量干扰代码
- 正常业务混合

### 动态检测
- 延迟执行
- 条件判断
- 环境检测
- 时间窗口

### Webshell 扫描器
- 河马查杀
- D盾
- 百度 WEBDIR+
- 深信服 Webshell 检测

## 生成策略

根据目标环境选择合适的混淆级别：
- 低防护：基础混淆即可
- 中防护：多层编码 + 回调函数
- 高防护：完全加密 + 业务伪装 + 流量加密

---

PHP Webshell 免杀需要不断测试和调整，针对不同的检测工具使用不同的技术。
