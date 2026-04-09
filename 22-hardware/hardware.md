# 硬件与固件安全

硬件设备的安全测试，包括固件分析、硬件调试、侧信道攻击等。

## 硬件接口

### UART
- 串口识别
- 波特率检测
- Shell 访问
- 日志获取

### JTAG
- 接口识别
- 调试访问
- 固件提取
- 内存读写

### SPI/I2C
- Flash 芯片读取
- EEPROM 访问
- 总线嗅探
- 数据提取

## 固件分析

### 固件提取
- 硬件提取
- 网络下载
- OTA 更新劫持
- 调试接口

### 固件解包
- binwalk
- firmware-mod-kit
- 文件系统提取
- 压缩格式识别

### 固件分析
- 文件系统分析
- 二进制分析
- 配置文件
- 密钥和凭证
- 后门检测

### 固件修改
- 文件系统修改
- 二进制 patch
- 后门植入
- 固件重打包

## 侧信道攻击

### 功耗分析
- SPA（Simple Power Analysis）
- DPA（Differential Power Analysis）
- CPA（Correlation Power Analysis）

### 电磁分析
- EM 泄漏
- 信号捕获
- 密钥恢复

### 时序攻击
- 时间测量
- 缓存时序
- 分支预测

### 故障注入
- 电压故障
- 时钟故障
- 激光故障
- 电磁故障

## 硬件工具

### 调试工具
- Bus Pirate
- JTAGulator
- OpenOCD
- 逻辑分析仪

### 提取工具
- Flash 编程器
- EEPROM 读写器
- 芯片脱焊工具

### 分析工具
- 示波器
- 频谱分析仪
- ChipWhisperer

---

硬件安全需要专业设备和技能，建议从基础开始逐步学习。
