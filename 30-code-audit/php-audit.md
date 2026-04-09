# PHP 代码审计

PHP 是 Web 开发的主流语言，其代码审计涉及框架漏洞、文件操作、反序列化等多个方面。

## 常见漏洞类型

### 代码执行

**危险函数**：
```php
// eval
eval($_GET['code']);

// assert
assert($_GET['code']);

// preg_replace /e修饰符 (PHP < 7.0)
preg_replace('/test/e', $_GET['code'], 'test');

// create_function (PHP < 7.2)
$func = create_function('', $_GET['code']);

// 动态函数调用
$func = $_GET['func'];
$func();
```

**审计要点**：
- 搜索 `eval`、`assert`
- 检查动态函数调用
- 验证用户输入过滤

### 命令执行

**危险函数**：
```php
// system
system($_GET['cmd']);

// exec
exec($_GET['cmd']);

// shell_exec
shell_exec($_GET['cmd']);

// passthru
passthru($_GET['cmd']);

// popen
popen($_GET['cmd'], 'r');

// proc_open
proc_open($_GET['cmd'], $descriptors, $pipes);

// 反引号
`{$_GET['cmd']}`;
```

**审计要点**：
- 搜索命令执行函数
- 检查参数过滤
- 验证 escapeshellarg 使用

### 文件包含

**本地文件包含**：
```php
// 不安全的include
include($_GET['file']);
require($_GET['file']);
include_once($_GET['file']);
require_once($_GET['file']);
```

**远程文件包含**：
```php
// allow_url_include = On
include($_GET['url']);
```

**伪协议利用**：
```php
// php://filter
include('php://filter/read=convert.base64-encode/resource=config.php');

// php://input
include('php://input');

// data://
include('data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8+');

// zip://
include('zip://shell.zip#shell.php');

// phar://
include('phar://shell.phar/shell.php');
```

**审计要点**：
- 搜索 include/require 函数
- 检查文件路径过滤
- 验证伪协议限制

### 文件操作

**文件上传**：
```php
// 不安全的文件上传
$filename = $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $filename);
```

**文件读取**：
```php
// 路径穿越
$file = $_GET['file'];
readfile($file);
file_get_contents($file);
```

**文件写入**：
```php
// 不安全的文件写入
$content = $_GET['content'];
file_put_contents('log.txt', $content);
```

**审计要点**：
- 检查文件类型验证
- 验证文件名过滤
- 查看路径规范化

### SQL 注入

**字符串拼接**：
```php
// 不安全的SQL
$sql = "SELECT * FROM users WHERE id = " . $_GET['id'];
$sql = "SELECT * FROM users WHERE name = '" . $_GET['name'] . "'";
```

**预处理语句**：
```php
// 安全的方式
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

**审计要点**：
- 搜索 SQL 字符串拼接
- 检查 mysqli_query、mysql_query
- 验证预处理语句使用

### 反序列化

**危险场景**：
```php
// 不安全的反序列化
$data = $_GET['data'];
unserialize($data);
```

**魔术方法**：
```php
__wakeup()    // 反序列化时调用
__destruct()  // 对象销毁时调用
__toString()  // 对象被当作字符串时调用
__call()      // 调用不存在的方法时调用
```

**POP 链构造**：
```php
// 寻找可利用的类
class FileHandler {
    private $filename;
    
    function __destruct() {
        unlink($this->filename);  // 文件删除
    }
}
```

**审计要点**：
- 搜索 `unserialize`
- 检查魔术方法
- 追踪 POP 链

### XSS

**反射型**：
```php
// 不安全的输出
echo $_GET['name'];
```

**存储型**：
```php
// 不安全的存储和输出
$comment = $_POST['comment'];
// 存储到数据库
echo $comment;  // 输出时未过滤
```

**审计要点**：
- 搜索 echo、print
- 检查 htmlspecialchars 使用
- 验证输出编码

## PHP 特性漏洞

### 弱类型比较

**危险比较**：
```php
// == 弱类型比较
if ($_GET['pass'] == $password) {
    // 0e123 == 0e456 返回 true
}

// in_array 第三个参数
if (in_array($_GET['id'], $ids)) {
    // in_array('1abc', [1]) 返回 true
}
```

**审计要点**：
- 搜索 `==` 比较
- 检查 `in_array` 使用
- 验证 `===` 使用

### 变量覆盖

**extract**：
```php
// 不安全的extract
extract($_GET);
// 可以覆盖任意变量
```

**parse_str**：
```php
// 不安全的parse_str
parse_str($_SERVER['QUERY_STRING']);
```

**$$变量**：
```php
// 可变变量
$var = $_GET['var'];
$$var = $_GET['value'];
```

**审计要点**：
- 搜索 `extract`、`parse_str`
- 检查可变变量使用
- 验证变量初始化

### 类型转换

**intval 绕过**：
```php
// intval 只取整数部分
$id = intval($_GET['id']);  // id=1.5 返回 1
```

**is_numeric 绕过**：
```php
// is_numeric 接受科学计数法
if (is_numeric($_GET['id'])) {
    // id=1e1 返回 true
}
```

## 框架审计

### ThinkPHP

**历史漏洞**：
- ThinkPHP 5.x RCE
- ThinkPHP 3.x SQL注入
- 路由RCE

**危险代码**：
```php
// 不安全的控制器方法
public function index() {
    $data = input('data');
    eval($data);
}
```

**审计要点**：
- 检查 ThinkPHP 版本
- 搜索 input() 函数
- 验证路由配置

### Laravel

**Mass Assignment**：
```php
// 不安全的批量赋值
User::create($request->all());
```

**SQL注入**：
```php
// 不安全的原始查询
DB::select("SELECT * FROM users WHERE id = " . $id);
```

**审计要点**：
- 检查 $fillable 和 $guarded
- 搜索 DB::raw
- 验证输入验证

### Symfony

**SSTI**：
```php
// 不安全的模板渲染
$twig->render($template, ['name' => $_GET['name']]);
```

**审计要点**：
- 检查模板引擎使用
- 验证用户输入过滤

## 审计工具

### 静态分析

**RIPS**：
```bash
# PHP静态分析工具
rips-cli scan:start -p /path/to/project
```

**PHPStan**：
```bash
phpstan analyse src
```

**Psalm**：
```bash
psalm --init
psalm
```

### 代码搜索

**正则搜索**：
```bash
# 搜索危险函数
grep -r "eval\|assert\|system\|exec" .

# 搜索SQL拼接
grep -r "\$sql.*\$_" .
```

## 审计清单

### 危险函数
- [ ] eval、assert
- [ ] system、exec、shell_exec
- [ ] include、require
- [ ] unserialize
- [ ] extract、parse_str

### 输入验证
- [ ] $_GET、$_POST、$_COOKIE
- [ ] $_SERVER、$_FILES
- [ ] 过滤函数使用
- [ ] 白名单验证

### 输出编码
- [ ] htmlspecialchars
- [ ] htmlentities
- [ ] json_encode
- [ ] 上下文编码

### 配置安全
- [ ] display_errors = Off
- [ ] allow_url_include = Off
- [ ] open_basedir 限制
- [ ] disable_functions 配置

---

**AI 发挥空间**：
- 根据PHP版本识别特定漏洞
- 分析框架特性发现安全问题
- 追踪变量流向发现注入点
- 开发自动化审计脚本

**反幻觉提示**：
- 不要假设所有PHP版本行为一致
- 验证具体的PHP版本和配置
- 测试漏洞的实际可利用性
- 注意不同框架的安全机制
- 某些漏洞可能需要特定的php.ini配置
