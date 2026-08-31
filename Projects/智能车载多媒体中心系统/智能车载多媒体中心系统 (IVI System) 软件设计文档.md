# 智能车载多媒体中心系统 (IVI System) 软件设计文档

## 1. 引言 (Introduction)

### 1.1 项目背景

本项目旨在基于 Orange Pi 3B (RK3566) 开发一款智能车载多媒体信息娱乐系统（In-Vehicle Infotainment, IVI）。针对开发板无物理显示屏的特性，系统采用前后端解耦架构，结合 Qt WebGL Streaming 技术实现跨设备 UI 渲染。

### 1.2 设计目标

- **高内聚低耦合**：业务逻辑与 UI 渲染严格分离（MVC 架构）。
    
- **高可靠性**：引入工业级异步日志系统，支持脱机状态下的故障溯源。
    
- **工程化规范**：严格遵循 Git Flow 工作流，支持 GDB 交叉调试与 ASAN 内存安全检测。
    

## 2. 系统运行环境 (Environment)

- **硬件平台**：Orange Pi 3B (ARM Cortex-A55, 2GB+ RAM)
    
- **外设硬件**：USB 摄像头 (UVC 协议)、PC 端浏览器 (用于 UI 呈现)
    
- **操作系统**：Ubuntu Base / Debian (Linux Kernel 5.10+)
    
- **核心技术栈**：C++14, Qt 5.15 (QML + WebGL), SocketCAN, V4L2, spdlog, CMake
    

## 3. 系统总体架构 (System Architecture)

系统采用**分层架构（Layered Architecture）**，自下而上分为四层：

codeText

```
+-----------------------------------------------------------------+
|                        表现层 (Presentation Layer)              |
|  [ QML 车载仪表盘 UI ]   [ 多媒体播放器 UI ]   [ 倒车影像 UI ]  |
|  (通过 Qt WebGL Streaming 插件，将渲染指令通过 WebSocket 推送至 PC) |
+-----------------------------------------------------------------+
|                        中间件层 (Middleware Layer)              |
|  [ Qt Meta-Object System (信号槽) ]   [ spdlog 异步日志系统 ]   |
+-----------------------------------------------------------------+
|                        服务层 (Service Layer)                   |
|  [ CAN 总线解析线程 ]    [ V4L2 图像采集线程 ]   [ 音频解码线程 ] |
+-----------------------------------------------------------------+
|                        操作系统与驱动层 (OS & Drivers)          |
|  [ SocketCAN (vcan0) ]   [ /dev/video0 ]      [ ALSA / GStreamer]|
+-----------------------------------------------------------------+
```

## 4. 核心模块详细设计 (Module Design)

### 4.1 日志与基础组件模块 (Logger Module)

- **设计要求**：车载设备无外接屏幕，必须依赖日志进行状态监控。日志写入不能阻塞主业务线程。
    
- **技术选型**：spdlog (C++ Header-only 库)
    
- **功能实现**：
    
    - **异步写入**：配置 spdlog::async_logger，将日志放入内存队列，由后台独立线程刷入磁盘。
        
    - **日志轮转 (Log Rotation)**：配置单个日志文件最大 5MB，最多保留 3 个备份（rotating_file_sink），防止嵌入式设备 SD 卡存储溢出。
        
    - **日志分级**：
        
        - TRACE/DEBUG：用于打印 CAN 报文原始 Hex 数据。
            
        - INFO：用于记录模块启动、网络连接状态。
            
        - ERROR/FATAL：用于记录设备节点打开失败、内存分配失败等严重异常。
            

### 4.2 车辆总线通信模块 (CAN Bus Module)

- **设计要求**：实时读取车辆引擎转速、车速等数据，延迟需小于 50ms。
    
- **技术选型**：Linux SocketCAN
    
- **功能实现**：
    
    - **独立线程**：创建一个 std::thread 专门阻塞读取 vcan0 接口的数据，防止阻塞 UI 线程。
        
    - **数据解析**：定义 CAN 报文结构体，根据特定的 CAN ID (如 0x1A6) 解析 Payload 中的车速和转速。
        
    - **线程间通信**：解析完成后，通过 Qt 的 Q_EMIT 发送信号，将数据安全地传递给主线程（UI 线程）更新仪表盘。
        

### 4.3 倒车影像与多媒体模块 (Media & Camera Module)

- **设计要求**：直接与 Linux 内核驱动交互，获取摄像头画面。
    
- **技术选型**：V4L2 (Video for Linux 2) API
    
- **功能实现**：
    
    - **设备初始化**：通过 open("/dev/video0", O_RDWR) 打开摄像头节点。
        
    - **内存映射 (mmap)**：使用 VIDIOC_REQBUFS 向内核申请帧缓冲区，并通过 mmap 映射到用户空间，实现零拷贝读取。
        
    - **格式转换**：摄像头通常输出 YUYV 格式，需在 C++ 层编写转换算法将其转换为 RGB888 格式，封装为 QImage 后推送到 QML 渲染。
        

### 4.4 前端交互与渲染模块 (UI Module)

- **设计要求**：解决无屏显示问题，实现前后端代码解耦。
    
- **技术选型**：Qt QML + C++ Backend + WebGL
    
- **功能实现**：
    
    - **C++ 注册上下文**：使用 qmlRegisterType 或 setContextProperty 将 C++ 后端类暴露给 QML。
        
    - **无头运行**：程序启动时添加参数 -platform webgl:port=8080。香橙派本地不进行物理渲染，PC 端通过浏览器访问 http://<香橙派IP>:8080 即可操作车载 UI。
        

## 5. 接口与数据结构设计 (Interface & Data Structures)

### 5.1 CAN 数据结构定义

codeC++

```
// 车辆状态数据结构
struct VehicleState {
    uint16_t speed;       // 车速 (km/h)
    uint16_t rpm;         // 发动机转速 (r/min)
    float engineTemp;     // 水温 (摄氏度)
    uint8_t gearStatus;   // 档位 (P/R/N/D)
};
```

### 5.2 C++ 与 QML 交互接口 (ViewModel)

codeC++

```
class VehicleViewModel : public QObject {
    Q_OBJECT
    Q_PROPERTY(int speed READ speed NOTIFY speedChanged)
    Q_PROPERTY(int rpm READ rpm NOTIFY rpmChanged)

public:
    explicit VehicleViewModel(QObject *parent = nullptr);
    int speed() const;
    int rpm() const;

signals:
    void speedChanged(int newSpeed);
    void rpmChanged(int newRpm);

public slots:
    // 供 CAN 接收线程调用的槽函数
    void updateVehicleData(const VehicleState& state);
};
```

## 6. 工程化与调试规范 (Engineering & Debugging)

### 6.1 项目目录结构 (CMake 构建)

codeText

```
OrangePi_IVI_System/
├── cmake/                      # [企业级] CMake 模块与工具链配置
│   ├── aarch64-toolchain.cmake # 核心！ARM 交叉编译工具链配置文件
│   ├── FindV4L2.cmake          # 自定义寻找 V4L2 库的 CMake 脚本
│   └── CompilerWarnings.cmake  # 统一配置严格的编译器警告 (-Wall -Wextra -Werror)
├── config/                     # 运行时配置文件 (与代码分离)
│   ├── default_settings.json   # 默认系统配置 (音量、主题、网络参数)
│   └── spdlog_conf.yaml        # 日志系统动态配置文件
├── docs/                       # 项目文档
│   ├── architecture.md         # 架构设计文档
│   ├── api/                    # Doxygen 生成的 API 文档存放处
│   └── Doxyfile                # Doxygen 配置文件
├── frontend/                   # 前端 UI 资源与代码 (前后端分离)
│   ├── qml/                    # QML 源码
│   │   ├── Dashboard.qml       # 仪表盘界面
│   │   ├── MediaView.qml       # 多媒体界面
│   │   └── components/         # 可复用的 QML 自定义组件
│   └── assets/                 # 静态资源
│       ├── fonts/              # 车载定制字体
│       ├── icons/              # SVG 图标
│       └── i18n/               # 国际化翻译文件 (.ts / .qm)
├── include/                    # 暴露给外部或跨模块调用的公共头文件
│   └── ivisystem/              # 加上项目命名空间，防止头文件冲突
│       ├── can/                # CAN 模块公共接口 (如 ICanReceiver.h)
│       ├── media/              # 媒体模块公共接口
│       └── utils/              # 工具类接口
├── src/                        # 核心源代码 (按模块划分)
│   ├── CMakeLists.txt          # src 目录的构建脚本
│   ├── main.cpp                # 应用程序入口 (负责初始化组件、依赖注入)
│   ├── can/                    # CAN 总线业务实现
│   │   ├── SocketCanDriver.cpp # 底层 vcan0 交互
│   │   └── CanDataParser.cpp   # 报文解析逻辑
│   ├── media/                  # 多媒体与外设实现
│   │   ├── V4l2Camera.cpp      # 摄像头 YUV 采集
│   │   └── AudioPlayer.cpp     # 音频播放
│   ├── ui_backend/             # Qt C++ 视图模型 (ViewModel)
│   │   ├── VehicleViewModel.cpp# 桥接 CAN 数据与 QML
│   │   └── MediaViewModel.cpp  # 桥接播放器与 QML
│   └── utils/                  # 基础工具实现
│       ├── Logger.cpp          # spdlog 封装
│       └── ConfigParser.cpp    # JSON 解析器
├── tests/                      # [企业级] 自动化测试代码 (极其重要)
│   ├── CMakeLists.txt          # 测试模块构建脚本
│   ├── unit_tests/             # 单元测试 (基于 Google Test / GTest)
│   │   ├── test_can_parser.cpp # 测试 CAN 报文解析逻辑是否正确
│   │   └── test_config.cpp     # 测试配置读取逻辑
│   └── integration_tests/      # 集成测试
├── scripts/                    # 开发、部署、运维辅助脚本
│   ├── build.sh                # 一键编译脚本 (支持传入 x86 或 arm 参数)
│   ├── deploy_to_board.sh      # 通过 SSH/SCP 自动将程序推送到香橙派
│   ├── setup_vcan.sh           # 宿主机/开发板初始化虚拟 CAN 网络的脚本
│   └── sim_can_data.py         # 模拟发送车辆 CAN 数据的 Python 脚本
├── third_party/                # 第三方依赖库 (Git Submodule 方式管理)
│   ├── spdlog/                 # 日志库
│   ├── nlohmann_json/          # 现代 C++ JSON 库
│   └── googletest/             # 谷歌测试框架
├── .clang-format               # C++ 代码格式化规范 (极其重要，统一代码风格)
├── .clang-tidy                 # C++ 静态代码分析配置 (检查内存泄漏、现代化语法)
├── .gitignore                  # Git 忽略文件 (忽略 build/, .vscode/, *.core 等)
├── CMakeLists.txt              # 顶层 CMake 构建脚本 (定义项目、引入子目录)
└── README.md                   # 项目主页说明 (编译指南、运行截图)
```

### 6.2 版本控制规范 (Git)

- **分支策略**：采用 Git Flow。主分支 main 保持稳定，日常开发在 develop 分支，新功能拉取 feature/xxx 分支。
    
- **提交规范**：遵循 Angular 规范。
    
    - 示例：feat(CAN): implement SocketCAN read thread and signal emission
        
    - 示例：fix(Camera): resolve segmentation fault when camera is unplugged
        

### 6.3 调试与质量保证 (GDB & ASAN)

1. **远程交叉调试 (Remote GDB)**
	- 目标机 (Orange Pi)：运行 gdbserver :1234 ./ivi_app
        
    - 宿主机 (PC)：运行交叉编译链中的 aarch64-linux-gnu-gdb，连接 target remote <IP>:1234 进行源码级单步调试。
2. ：
    
    
3. **Core Dump 崩溃分析**：
    
    - 系统配置 ulimit -c unlimited。若发生段错误，利用 GDB 加载 core 文件，使用 bt (backtrace) 命令快速定位崩溃的函数调用栈。
        
4. **内存泄漏检测**：
    
    - 在 CMakeLists.txt 中配置 set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -g")。
        
    - 利用 ASAN (AddressSanitizer) 在运行时检测数组越界、野指针及内存泄漏问题。
        

## 7. 异常处理与容错机制 (Error Handling)

1. **外设热插拔容错**：V4L2 采集线程需捕获 read 或 ioctl 返回的 ENODEV 错误。当摄像头意外拔出时，线程不应崩溃，而是输出 ERROR 日志，并进入休眠轮询状态，等待设备重新接入。
    
2. **CAN 总线异常**：若超过 2 秒未接收到 CAN 报文，触发超时机制，UI 仪表盘数据归零，并弹出“车辆总线连接丢失”的警告提示。
    