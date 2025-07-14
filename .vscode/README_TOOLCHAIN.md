# CH32L103C8T6 开发环境配置

## 工具链配置

本项目使用VSCode变量来管理工具链路径，方便在不同平台间迁移。

### 1. 修改工具链基础路径

编辑 `.vscode/settings.json` 文件中的 `WCH_TOOLCHAIN_BASE` 变量：

```json
{
    "WCH_TOOLCHAIN_BASE": "/your/toolchain/path",
    "WCH_ELF_FILENAME": "RISCV-LED.elf",
    "BUILD_LEVEL": "RelWithDebInfo"
}
```

### 2. ELF文件名配置

`WCH_ELF_FILENAME` 变量定义了生成的ELF文件名，默认为 `RISCV-LED.elf`。
如果您的CMakeLists.txt中使用了不同的项目名称，请相应修改此变量。

### 3. 不同平台的典型路径

#### Linux
```json
"WCH_TOOLCHAIN_BASE": "/opt/MRS_Toolchain_Linux_x64_V210"
```

#### Windows
```json
"WCH_TOOLCHAIN_BASE": "C:/MRS_Toolchain_Windows_x64_V210"
```

#### macOS
```json
"WCH_TOOLCHAIN_BASE": "/Applications/MRS_Toolchain_macOS_x64_V210"
```

### 4. 工具链结构

工具链基础路径下应包含以下结构：
```
<WCH_TOOLCHAIN_BASE>/
├── RISC-V Embedded GCC12/
│   ├── bin/
│   │   ├── riscv-wch-elf-gcc
│   │   ├── riscv-wch-elf-g++
│   │   └── riscv-wch-elf-gdb
│   └── riscv-wch-elf/
│       └── include/
└── OpenOCD/
    └── OpenOCD/
        ├── bin/
        │   └── openocd
        └── share/
            └── openocd/
                └── scripts/
```

### 5. 验证配置

配置完成后，可以通过以下方式验证：

1. **检查智能感知**：打开任意 .c/.cpp 文件，检查是否有正确的代码补全
2. **检查构建**：运行 `CMake Configure` 任务
3. **检查调试**：尝试启动调试配置

### 6. 环境变量（可选）

也可以设置系统环境变量：
```bash
export WCH_TOOLCHAIN_BASE="/opt/MRS_Toolchain_Linux_x64_V210"
```

然后在settings.json中使用：
```json
"WCH_TOOLCHAIN_BASE": "${env:WCH_TOOLCHAIN_BASE}"
```

## 支持的功能

- ✅ C/C++智能感知和代码补全
- ✅ CMake构建系统
- ✅ GDB调试（基于.mrs配置优化）
- ✅ OpenOCD固件下载（改进的搜索路径）
- ✅ 多种调试和下载模式
- ✅ SVD文件支持（寄存器视图）
- ✅ RISC-V架构专用GDB命令
- ✅ 抽象化ELF文件名配置

## Run Tasks说明
### 1. CMake工具
- 包含Configure, Build, Clean操作
### 2. OpenOCD工具
- 包含Flash操作 

## 调试配置说明

### 1. Debug with WCH-Link
- 完整调试模式，支持断点、单步调试
- 包含RISC-V特定的GDB初始化命令
- 自动设置内存访问和架构
- 支持SVD文件寄存器视图

### 2. Attach to Chip
- 连接到已运行的目标
- 用于调试已启动的程序
- 支持SVD文件寄存器视图

### 3. Download and Run
- 仅下载固件，不进入调试模式
- 下载完成后自动复位

## 快速开始

1. 克隆项目
2. 修改 `.vscode/settings.json` 中的工具链路径
3. 重新加载VSCode窗口
4. 运行 `CMake Build` 任务进行构建
5. 使用 `Download Firmware` 进行固件下载

## 故障排除

### 找不到编译器
- 检查 `WCH_TOOLCHAIN_BASE` 路径是否正确
- 确认工具链已正确安装
- 重新加载VSCode窗口

### IntelliSense不工作
- 检查include路径配置
- 运行 `C/C++: Reset IntelliSense Database` 命令

### 调试无法启动
- 检查OpenOCD路径配置
- 确认硬件连接正常
- 检查配置文件 `wch-riscv.cfg` 是否存在
