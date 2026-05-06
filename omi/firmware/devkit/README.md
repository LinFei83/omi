# Omi DevKit 固件

Omi DevKit 是基于 Seeed Xiao nRF52840 Sense 开发板的音频采集与传输设备固件。该项目使用 Zephyr RTOS 开发，支持蓝牙音频流传输、SD 卡离线存储等功能。

## 硬件平台

- **主控芯片**: Nordic nRF52840 (ARM Cortex-M4F)
- **开发板**: Seeed Xiao BLE Sense
- **传感器**: 
  - PDM 麦克风
  - LSM6DSL 六轴传感器（加速度计 + 陀螺仪）
- **存储**: SD 卡支持（SPI/SDMMC）
- **音频输出**: I2S 扬声器
- **电源管理**: 锂电池充电管理（快充100mA/慢充50mA）
- **外部闪存**: P25Q16H QSPI Flash

## 主要功能

### 核心功能
- **音频采集**: 16kHz 采样率，PDM 麦克风输入
  - PDM_DIN: P0.16
  - PDM_CLK: P1.00
  - PDM_PWR: P1.10
- **音频编码**: Opus 编码（32kbps，低延迟模式）
- **蓝牙传输**: BLE 5.0，最大 MTU 498 字节，支持 2M PHY
- **离线存储**: SD 卡录音存储，支持 FAT32/exFAT
  - SPI_SCK: P1.13
  - SPI_MOSI: P1.15
  - SPI_MISO: P1.14
  - SPI_CS: P0.02
- **扬声器输出**: I2S 音频播放
  - I2S_SCK: P0.29 (A3)
  - I2S_LRCK: P0.28 (A2)
  - I2S_SDOUT: P0.03 (A1)
  - 扬声器电源控制: P0.04
- **电池监控**: 电池电压和电量百分比检测
  - 充电速度控制: P0.13
  - 充电使能: P0.17
  - 电池读取使能: P0.14
  - ADC 通道: AIN7
- **按键控制**: 支持设备开关和状态切换
  - 按键输出: P0.04
  - 按键输入: P0.05
- **振动反馈**: 触觉反馈功能
  - 振动马达控制: P1.11
- **加速度计**: LSM6DSL 六轴传感器（可选）
  - I2C 地址: 0x6A
  - 中断引脚: P0.11

### LED 状态指示
- **启动序列**: 红 → 绿 → 蓝 → 全亮 → 全灭
- **运行状态**:
  - 蓝色: 已连接并录音
  - 红色: 录音中但未连接
  - 绿色闪烁: 充电中
  - 关机: 所有 LED 熄灭

## 硬件配置版本

项目支持多个硬件配置版本：

1. **devkitv1**: 基础版本，标准配置
2. **devkitv1-spisd**: 使用 SPI 方式连接 SD 卡
3. **devkitv2-adafruit**: 使用 Adafruit SD 卡模块

每个版本有对应的配置文件：
- `prj_xiao_ble_sense_devkitv1.conf`
- `prj_xiao_ble_sense_devkitv1-spisd.conf`
- `prj_xiao_ble_sense_devkitv2-adafruit.conf`

## 项目结构

```
devkit/
├── src/
│   ├── main.c              # 主程序入口
│   ├── transport.c/.h      # 蓝牙传输层
│   ├── codec.c/.h          # Opus 音频编解码
│   ├── mic.c/.h            # 麦克风驱动
│   ├── speaker.c/.h        # 扬声器驱动
│   ├── led.c/.h            # LED 控制
│   ├── button.c/.h         # 按键处理
│   ├── sdcard.c/.h         # SD 卡文件系统
│   ├── storage.c/.h        # 存储管理
│   ├── usb.c/.h            # USB 充电检测
│   ├── wdog_facade.c/.h    # 看门狗管理
│   ├── config.h            # 全局配置
│   └── lib/
│       ├── opus-1.2.1/     # Opus 编解码库
│       └── battery/        # 电池管理库
├── overlay/                # 设备树覆盖文件
├── CMakeLists.txt          # CMake 构建配置
├── prj_*.conf              # Zephyr 项目配置
└── flash.sh                # 烧录脚本
```

## 技术参数

### 音频规格
- **采样率**: 16kHz
- **缓冲区大小**: 100ms (1600 samples)
- **编码格式**: Opus (CELT 模式)
- **比特率**: 32kbps
- **每包样本数**: 160
- **编码复杂度**: 3

### 蓝牙规格
- **设备名称**: "Omi DevKit 2"
- **最大连接数**: 1
- **TX 功率**: +8 dBm
- **MTU**: 498 字节
- **L2CAP 缓冲**: 2048 字节
- **支持特性**: 2M PHY, 数据长度扩展

### 功耗管理
- **DCDC 转换器**: 已启用
- **QSPI Flash**: 自动深度睡眠
- **看门狗**: 启用，防止系统冻结
- **USB 充电检测**: 支持

## 构建和烧录

### 环境要求
- Zephyr SDK
- CMake 3.20+
- Nordic nRF Command Line Tools

### 构建命令
```bash
# 使用 west 构建（以 devkitv2-adafruit 为例）
west build -b seeed_xiao_nrf52840_sense -- \
  -DCONF_FILE=prj_xiao_ble_sense_devkitv2-adafruit.conf \
  -DDTC_OVERLAY_FILE=overlay/xiao_ble_sense_devkitv2-adafruit.overlay
```

### 烧录
```bash
# 使用提供的脚本
./flash.sh

# 或使用 west
west flash
```

### UF2 方式烧录
如果需要通过拖拽方式烧录：
1. 将配置文件中 `CONFIG_BUILD_OUTPUT_UF2=n` 改为 `y`
2. 重新构建
3. 双击复位按钮进入 UF2 模式
4. 拖拽生成的 `.uf2` 文件到 USB 驱动器

## 蓝牙配对

### 配对模式
设备上电后会**自动进入广播模式**，无需手动操作进入配对状态。具体流程：

1. **自动广播**: 设备启动完成后，蓝牙会自动开始广播（Advertising）
2. **设备名称**: "Omi DevKit 2"
3. **广播类型**: BT_LE_ADV_CONN（可连接广播）
4. **广播内容**:
   - 设备名称
   - 音频服务 UUID: 19B10000-E8F2-537E-4F6C-D104768A1214
   - DFU 服务 UUID: 00001530-1212-EFDE-1523-785FEABCD123
   - 设备信息服务

### 连接状态
- **最大连接数**: 1（同时只能连接一个设备）
- **最大配对数**: 1
- **断开后**: 自动重新开始广播，等待新的连接

### 手动控制蓝牙
通过按键可以控制蓝牙开关：
- 短按按键: 正常使用
- 长按按键: 可以触发设备关机，停止蓝牙广播

### 配对步骤（客户端）
1. 打开手机或电脑的蓝牙设置
2. 搜索附近的蓝牙设备
3. 找到名为 "Omi DevKit 2" 的设备
4. 点击连接即可（无需 PIN 码）

## 硬件组件依赖说明

### 必需组件
根据当前默认配置（devkitv2-adafruit），以下组件初始化失败会导致系统**无法启动**：

1. **SD 卡** (CONFIG_OMI_ENABLE_OFFLINE_STORAGE=y)
   - 如果 SD 卡初始化失败，系统会返回错误并停止
   - 错误提示: "Failed to mount SD card (err %d)"
   
2. **麦克风** (PDM 麦克风，无条件编译)
   - 麦克风初始化失败会导致系统停止
   - 错误提示: "Failed to start microphone: %d"
   - LED 错误指示: 红色和绿色 LED 交替闪烁 5 次

3. **Opus 编解码器** (CONFIG_OMI_CODEC_OPUS=y)
   - 编解码器初始化失败会停止系统
   - 错误提示: "Failed to start codec: %d"
   - LED 错误指示: 蓝色 LED 闪烁 5 次

### 可选组件
以下组件可以通过修改配置文件禁用：

- **加速度计** (CONFIG_OMI_ENABLE_ACCELEROMETER=n) - 默认已禁用
- **扬声器** (CONFIG_OMI_ENABLE_SPEAKER=y)
- **按键** (CONFIG_OMI_ENABLE_BUTTON=y)
- **电池管理** (CONFIG_OMI_ENABLE_BATTERY=y)
- **振动反馈** (CONFIG_OMI_ENABLE_HAPTIC=y)
- **USB 功能** (CONFIG_OMI_ENABLE_USB=y)

### 禁用硬件组件的方法

如果没有焊接 SD 卡或麦克风，需要修改配置文件：

**方法 1: 禁用 SD 卡存储**
编辑 `prj_xiao_ble_sense_devkitv2-adafruit.conf`，将：
```conf
CONFIG_OMI_ENABLE_OFFLINE_STORAGE=y
```
改为：
```conf
CONFIG_OMI_ENABLE_OFFLINE_STORAGE=n
```

**方法 2: 修改麦克风初始化逻辑**
麦克风目前是强制初始化的，如果要禁用需要：
1. 在配置文件中添加新的开关（如 `CONFIG_OMI_ENABLE_MIC=n`）
2. 在 `src/main.c` 中用 `#ifdef` 包裹麦克风初始化代码（328-340行）

**建议配置（最小系统）**
如果只是测试蓝牙功能，可以修改为：
```conf
CONFIG_OMI_ENABLE_OFFLINE_STORAGE=n
CONFIG_OMI_ENABLE_ACCELEROMETER=n
CONFIG_OMI_ENABLE_BUTTON=n
CONFIG_OMI_ENABLE_SPEAKER=n
CONFIG_OMI_ENABLE_BATTERY=n
CONFIG_OMI_ENABLE_USB=n
CONFIG_OMI_ENABLE_HAPTIC=n
```

## 调试

### 查看启动日志

**步骤 1: 启用日志输出**

在配置文件 `prj_xiao_ble_sense_devkitv2-adafruit.conf` 中，将以下被注释的行取消注释：

```conf
# 启用日志系统
CONFIG_LOG=y
CONFIG_LOG_PRINTK=y
CONFIG_UART_CONSOLE=y

# 可选：设置日志级别
CONFIG_LOG_DEFAULT_LEVEL=3  # 0=OFF, 1=ERR, 2=WRN, 3=INF, 4=DBG
```

同时需要启用控制台输出：
```conf
CONFIG_CONSOLE=y
CONFIG_PRINTK=y
```

**步骤 2: 连接串口**

Xiao BLE Sense 开发板通过 USB 提供虚拟串口功能：

1. **Windows 系统**:
   - 连接 USB 线
   - 打开设备管理器，查找 "端口(COM 和 LPT)"
   - 找到 "USB Serial Device (COMx)" 或类似名称
   - 使用串口工具（推荐 PuTTY、Tera Term 或 Arduino Serial Monitor）
   - 配置：波特率 115200，8 位数据位，无奇偶校验，1 位停止位

2. **Linux 系统**:
   ```bash
   # 查找串口设备
   ls /dev/ttyACM*
   
   # 使用 screen 连接
   screen /dev/ttyACM0 115200
   
   # 或使用 minicom
   minicom -D /dev/ttyACM0 -b 115200
   ```

3. **macOS 系统**:
   ```bash
   # 查找串口设备
   ls /dev/tty.usbmodem*
   
   # 使用 screen 连接
   screen /dev/tty.usbmodem14101 115200
   ```

**步骤 3: 重启设备查看日志**

连接串口后，按下开发板的复位按钮，你将看到类似以下的启动日志：

```
Power-on-reset
Booting...
Model: Omi DevKit 2
Firmware revision: 2.0.10
Hardware revision: Seeed Xiao BLE Sense

Initializing LEDs...
Watchdog init...
Battery initialized
Button initialized
Speaker initialized

Mount SD card...
Failed to mount SD card (err -2)  # 如果没有 SD 卡

Initializing transport...
Transport bluetooth initialized
Advertising successfully started

Initializing codec...
Initializing microphone...
Device initialized successfully

Entering main loop...
```

**步骤 4: 常见启动日志分析**

- **成功启动**: 最后会显示 "Device initialized successfully" 和 "Entering main loop..."
- **SD 卡错误**: "Failed to mount SD card (err -2)" - 没有插入 SD 卡或格式不正确
- **麦克风错误**: "Failed to start microphone: -5" - PDM 麦克风硬件问题
- **蓝牙错误**: "Transport bluetooth init failed (err %d)" - 蓝牙初始化失败
- **看门狗复位**: "Reset by WATCHDOG" - 系统在运行中冻结

### LED 启动状态指示

即使没有启用串口日志，也可以通过 LED 判断启动状态：

1. **正常启动序列**:
   - 红色闪烁 → 绿色闪烁 → 蓝色闪烁 → 全亮 → 全灭
   - 然后蓝色亮 1 秒后熄灭
   - 进入主循环（蓝色=已连接，红色=未连接）

2. **蓝牙传输错误**:
   - 绿色 LED 快速闪烁 5 次

3. **编解码器错误**:
   - 蓝色 LED 闪烁 5 次

4. **麦克风错误**:
   - 红色和绿色 LED 交替闪烁 5 次

### 日志级别说明
- `0`: 关闭日志
- `1`: 仅错误
- `2`: 错误 + 警告
- `3`: 错误 + 警告 + 信息（推荐）
- `4`: 全部调试信息（可能导致崩溃）

### 查看日志
使用串口工具连接开发板 UART（通常为 USB CDC ACM）：
```bash
# Linux/macOS
screen /dev/ttyACM0 115200

# Windows
使用 PuTTY 或 Tera Term
```

## 复位原因检测

固件启动时会显示上次复位的原因：
- **Watchdog**: 看门狗超时
- **Pin-reset**: 复位引脚触发
- **Soft-reset**: 软件复位
- **CPU LOCKUP**: CPU 锁死
- **NFC**: NFC 场检测唤醒
- **Power-on-reset**: 上电复位

## 配置选项

### 功能开关
在 `.conf` 文件中可以配置以下功能：
```conf
CONFIG_OMI_CODEC_OPUS=y              # Opus 编解码
CONFIG_OMI_ENABLE_OFFLINE_STORAGE=y  # SD 卡存储
CONFIG_OMI_ENABLE_ACCELEROMETER=n    # 加速度计
CONFIG_OMI_ENABLE_BUTTON=y           # 按键
CONFIG_OMI_ENABLE_SPEAKER=y          # 扬声器
CONFIG_OMI_ENABLE_BATTERY=y          # 电池管理
CONFIG_OMI_ENABLE_USB=y              # USB 功能
CONFIG_OMI_ENABLE_HAPTIC=y           # 振动反馈
```

### 优化建议
- 生产环境：关闭所有日志以节省内存和功耗
- 开发环境：启用日志级别 3，便于调试
- 性能测试：启用 `CONFIG_DEBUG_OPTIMIZATIONS=n`

## 常见问题

### 蓝牙连接失败
- 检查设备名称是否正确
- 确认是否已配对过多设备（最大配对数：1）
- 查看蓝牙协议栈日志

### SD 卡无法挂载
- 确认 SD 卡格式为 FAT32 或 exFAT
- 检查 SPI 引脚配置是否正确
- 查看 SD 卡是否损坏

### 音频编码失败
- 检查 Opus 库是否正确编译
- 确认 ARM 优化标志已启用
- 查看内存是否充足

### 系统频繁复位
- 检查看门狗喂狗是否正常
- 排查是否有死循环或长时间阻塞
- 启用调试日志查看崩溃位置

## 设备信息

- **型号**: Omi DevKit 2
- **制造商**: Based Hardware
- **固件版本**: 2.0.10
- **硬件版本**: Seeed Xiao BLE Sense

## 许可证

本项目包含多个开源组件：
- Zephyr RTOS: Apache License 2.0
- Opus Codec: BSD License
- Battery Library: Apache License 2.0

## 贡献

欢迎提交问题和改进建议。

## 相关资源

- [Zephyr 官方文档](https://docs.zephyrproject.org/)
- [Nordic nRF52840 数据手册](https://www.nordicsemi.com/products/nrf52840)
- [Seeed Xiao BLE Sense 文档](https://wiki.seeedstudio.com/XIAO_BLE/)
- [Opus 编解码器](https://opus-codec.org/)
