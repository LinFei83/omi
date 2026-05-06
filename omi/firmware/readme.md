# Omi 固件

本仓库包含 Omi AI 可穿戴设备的固件。

## 概述

Omi 固件基于 Zephyr RTOS 构建，提供音频采集、处理和电池管理功能。它包含用于流式传输音频数据的蓝牙连接。由于该固件的复杂性，无法在 Arduino IDE 中构建，需要更高级的工具链。

## 目录结构

- `omi/`: 主应用程序项目文件
    - `src/`: 应用程序代码的源文件
    - `lib/`: 应用程序使用的库
    - `CMakeLists.txt`: CMake 构建配置
    - `CMakePresets.json`: CMake 预设配置
- `devkit/`: 开发套件应用程序项目文件（用于 Omi DevKit1、Omi DevKit2）
    - `src/`: 开发套件版本的专用源文件
    - `lib/`: 开发套件使用的库
    - `CMakeLists.txt`: CMake 构建配置
    - `CMakePresets.json`: CMake 预设配置
- `test/`: 测试项目文件
- `boards/`: 自定义板卡定义和配置
- `scripts/`: 构建和实用脚本

## 构建和烧录固件

请遵循我们[官方文档](https://docs.omi.me/doc/developer/firmware/Compile_firmware)中的说明。

## 设备特定构建

对于不同的设备硬件版本（例如 V1 和 V2），使用单独的覆盖文件和项目配置文件。有关可用配置，请参阅 `CMakePresets.json`。

这些覆盖文件在构建时为固件提供有关引脚和设备功能的上下文信息。每个设备都需要自己独特的构建。

要选择适当的覆盖文件，请在 nRF Connect 扩展侧边栏中选择 `CMakePresets.json` 中设置的配置。

## 调试

启用 USB 串口调试：

### DevKit2

1. 克隆此仓库并编辑 `firmware/devkit/prj_xiao_ble_sense_devkitv2-adafruit.conf`。
2. 确保以下设置已启用（设置为 `y` 且未被注释）：
   - `CONFIG_CONSOLE=y`
   - `CONFIG_PRINTK=y`
   - `CONFIG_LOG=y`
   - `CONFIG_LOG_PRINTK=y`
   - `CONFIG_UART_CONSOLE=y`
3. 离线存储目前处于实验阶段，必须禁用才能进行日志记录。确保以下设置已禁用（设置为 `n` 且未被注释）：
   - `CONFIG_OMI_ENABLE_OFFLINE_STORAGE=n`
4. **注意：** 可以通过以下设置在启用离线存储的情况下进行日志记录。**这些设置可能会影响通过 BLE 传输数据或写入 SD 卡时的性能**：
   - `CONFIG_LOG_PROCESS_THREAD_PRIORITY=5`
   - `CONFIG_LOG_PROCESS_THREAD_CUSTOM_PRIORITY=y`
5. 根据[官方文档](https://docs.omi.me/doc/developer/firmware/Compile_firmware)中的说明构建和烧录调试固件。
6. 在 VS Code 中使用 nRF Serial Terminal 查看调试输出。

使用 nRF Connect 扩展也支持完整的实时代码调试；但是，这需要 J-Link 调试器设备和额外的设置。

## 关键组件

- **主应用程序**: 协调设备从初始化到关闭的整体功能。
- **音频采集**: 处理麦克风输入和音频缓冲。
- **编解码器**: 处理原始音频数据。
- **传输**: 管理蓝牙连接和音频流。
- **存储**: 处理 SD 卡操作和音频文件管理。
- **LED 控制**: 提供有关设备状态的视觉反馈。

## 关于存储读取

每当没有与应用程序的蓝牙连接时，存储将自动激活。每当您打开设备时，都会创建一个新文件，该文件将开始填充 opus 编码数据。每当您连接到应用程序时，存储的内容将开始流式传输到应用程序。完成后，它将尝试删除设备上的文件。

每个数据包的格式与流式音频数据包不同。
