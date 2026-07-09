## 1. 文档说明

### 1.1 项目名称

Linux 设备状态监控与预警系统

英文名称：

Linux Device Monitor Agent

### 1.2 文档目的

本文档用于描述 Linux 设备状态监控与预警系统的详细设计，包括系统架构、模块划分、数据结构、数据库设计、通信协议、告警策略、日志设计、异常处理、部署方式和测试方案。

本系统面向嵌入式 Linux 设备、边缘网关、工业控制设备或普通 Linux 主机，提供设备运行状态采集、异常判断、本地存储和远程上报能力。

### 1.3 目标用户

本系统主要面向：

- 嵌入式 Linux 设备运维人员
    
- 边缘网关开发人员
    
- 设备管理平台后端服务
    
- 需要远程监控 Linux 设备状态的业务系统
    

### 1.4 设计目标

系统需要满足以下目标：

1. 能够长期稳定运行在 Linux 设备上。
    
2. 能够周期性采集 CPU、内存、磁盘、网络、温度、进程等设备状态。
    
3. 能够根据配置文件中的阈值判断异常状态。
    
4. 能够对异常状态进行分级告警。
    
5. 能够将设备状态和告警事件保存到本地 SQLite 数据库。
    
6. 能够通过 MQTT 或 HTTP 将数据上报到远程服务。
    
7. 能够以 systemd 服务方式部署，实现开机自启动和异常自动重启。
    
8. 系统结构清晰，模块解耦，便于后续扩展。
    

---

## 2. 系统总体设计

### 2.1 系统角色

本系统中的设备端程序称为 Monitor Agent。

Agent 运行在被监控 Linux 设备上，负责本机状态采集、告警判断、本地存储和远程上报。

系统整体可以分为三部分：

```text
Linux 设备端 Agent
    ↓
MQTT Broker / HTTP Server
    ↓
Web 管理后台 / 数据分析平台
```

本阶段主要完成 Linux 设备端 Agent 的设计与开发。

### 2.2 系统架构

系统采用分层架构设计。

```text
┌──────────────────────────────┐
│        应用控制层             │
│  Main / Scheduler / Signal    │
├──────────────────────────────┤
│        业务逻辑层             │
│  Alarm Engine / Rule Manager  │
├──────────────────────────────┤
│        数据采集层             │
│  CPU / Memory / Disk / Net    │
│  Temp / Process / Service     │
├──────────────────────────────┤
│        数据访问层             │
│  SQLite Storage / Log System  │
├──────────────────────────────┤
│        通信层                 │
│  MQTT Reporter / HTTP Client  │
├──────────────────────────────┤
│        系统接口层             │
│  /proc /sys statvfs socket    │
└──────────────────────────────┘
```

### 2.3 核心数据流

```text
系统启动
  ↓
加载配置文件
  ↓
初始化日志模块
  ↓
初始化数据库
  ↓
初始化采集模块
  ↓
初始化通信模块
  ↓
周期采集设备指标
  ↓
保存设备状态
  ↓
执行告警判断
  ↓
保存告警事件
  ↓
远程上报状态和告警
  ↓
继续下一轮采集
```

---

## 3. 功能需求详细设计

### 3.1 设备状态采集

系统需要周期性采集以下指标：

|指标类型|数据来源|说明|
|---|---|---|
|CPU 使用率|`/proc/stat`|通过两次采样差值计算|
|内存使用率|`/proc/meminfo`|使用 MemTotal 和 MemAvailable|
|磁盘使用率|`statvfs()`|统计指定挂载点空间|
|网络速率|`/proc/net/dev`|计算收发字节差值|
|设备温度|`/sys/class/thermal/`|读取 thermal zone 温度|
|进程状态|`/proc/[pid]/comm`|检查关键进程是否存在|
|服务状态|`systemctl is-active`|检查 systemd 服务状态|

### 3.2 告警判断

系统根据配置文件中的阈值进行判断。

支持以下告警类型：

|告警类型|触发条件|
|---|---|
|CPU_HIGH|CPU 使用率超过阈值|
|MEMORY_HIGH|内存使用率超过阈值|
|DISK_HIGH|磁盘使用率超过阈值|
|TEMP_HIGH|温度超过阈值|
|PROCESS_DOWN|关键进程不存在|
|SERVICE_DOWN|关键服务未运行|
|NETWORK_ABNORMAL|网络收发速率异常|

### 3.3 告警等级

告警等级分为三级：

|等级|含义|
|---|---|
|INFO|普通状态变化|
|WARNING|资源使用率偏高，需要关注|
|CRITICAL|严重异常，可能影响业务运行|

### 3.4 告警策略

为了避免频繁误报，系统采用以下策略：

1. **连续触发机制**
    
    单次超过阈值不立即报警，连续 N 次异常才触发告警。
    
2. **告警恢复机制**
    
    当指标恢复正常后，生成恢复事件。
    
3. **告警抑制机制**
    
    同一类型告警在指定时间窗口内不重复发送。
    
4. **告警状态机**
    

```text
NORMAL
  ↓ 指标超过阈值
PENDING
  ↓ 连续 N 次异常
ALARMING
  ↓ 指标恢复正常
RECOVERED
  ↓ 记录恢复事件
NORMAL
```

---

## 4. 模块详细设计

### 4.1 主控模块 Main / Scheduler

#### 4.1.1 模块职责

主控模块负责程序启动、初始化、主循环调度和退出处理。

主要职责：

- 加载配置文件
    
- 初始化日志系统
    
- 初始化 SQLite 数据库
    
- 初始化采集器
    
- 初始化告警引擎
    
- 初始化通信模块
    
- 周期性调度采集任务
    
- 捕获退出信号
    
- 释放系统资源
    

#### 4.1.2 执行流程

```text
main()
  ↓
parse_args()
  ↓
load_config()
  ↓
init_logger()
  ↓
init_storage()
  ↓
init_collectors()
  ↓
init_reporter()
  ↓
run_scheduler()
  ↓
cleanup()
```

---

### 4.2 配置管理模块 ConfigManager

#### 4.2.1 模块职责

配置管理模块负责读取、解析和校验配置文件。

配置项包括：

- 设备 ID
    
- 采集周期
    
- 上报周期
    
- 告警阈值
    
- MQTT 配置
    
- HTTP 配置
    
- 监控进程列表
    
- 监控服务列表
    
- 日志级别
    
- 数据库路径

#### 4.2.3 异常处理

|异常情况|处理方式|
|---|---|
|配置文件不存在|使用默认配置，记录 WARN 日志|
|配置格式错误|程序拒绝启动，记录 ERROR 日志|
|配置字段缺失|使用默认值补齐|
|阈值不合法|使用默认阈值|

---

### 4.3 指标采集模块 Collector

#### 4.3.1 模块职责

采集模块负责从 Linux 系统中读取运行状态信息，并转换为统一的数据结构。

#### 4.3.2 采集器接口设计

```cpp
class ICollector {
public:
    virtual ~ICollector() = default;
    virtual bool collect(DeviceMetrics& metrics) = 0;
};
```

各采集器实现统一接口：

```cpp
class CpuCollector : public ICollector;
class MemoryCollector : public ICollector;
class DiskCollector : public ICollector;
class NetworkCollector : public ICollector;
class TemperatureCollector : public ICollector;
class ProcessCollector;
class ServiceCollector;
```

#### 4.3.3 设备状态数据结构

```cpp
struct DeviceMetrics {
    std::string device_id;
    double cpu_usage;
    double memory_usage;
    double disk_usage;
    double temperature;
    double rx_rate_kbps;
    double tx_rate_kbps;
    long timestamp;
};
```

#### 4.3.4 进程状态数据结构

```cpp
struct ProcessStatus {
    std::string process_name;
    int pid;
    bool running;
    long timestamp;
};
```

#### 4.3.5 服务状态数据结构

```cpp
struct ServiceStatus {
    std::string service_name;
    std::string status;
    bool active;
    long timestamp;
};
```

---

### 4.4 告警模块 AlarmEngine

#### 4.4.1 模块职责

告警模块负责根据设备指标和配置规则判断是否产生告警事件。

主要职责：

- 阈值判断
    
- 连续异常计数
    
- 告警等级判断
    
- 告警抑制
    
- 告警恢复
    
- 生成告警事件对象
    

#### 4.4.2 告警事件结构

```cpp
struct AlarmEvent {
    std::string device_id;
    std::string alarm_type;
    std::string level;
    std::string message;
    long timestamp;
    bool recovered;
};
```

#### 4.4.3 告警规则结构

```cpp
struct AlarmRule {
    std::string alarm_type;
    double warning_threshold;
    double critical_threshold;
    int continuous_count;
    int suppress_seconds;
};
```

#### 4.4.4 告警引擎接口

```cpp
class AlarmEngine {
public:
    std::vector<AlarmEvent> checkMetrics(const DeviceMetrics& metrics);
    std::vector<AlarmEvent> checkProcesses(const std::vector<ProcessStatus>& processes);
    std::vector<AlarmEvent> checkServices(const std::vector<ServiceStatus>& services);
};
```

---

### 4.5 数据存储模块 SQLiteStorage

#### 4.5.1 模块职责

数据存储模块负责保存设备状态、告警事件、进程状态和服务状态。

#### 4.5.2 存储接口

```cpp
class IStorage {
public:
    virtual ~IStorage() = default;
    virtual bool saveMetrics(const DeviceMetrics& metrics) = 0;
    virtual bool saveAlarm(const AlarmEvent& alarm) = 0;
    virtual bool saveProcessStatus(const ProcessStatus& status) = 0;
    virtual bool saveServiceStatus(const ServiceStatus& status) = 0;
};
```

#### 4.5.3 数据库表设计

##### device_metrics 表

```sql
CREATE TABLE IF NOT EXISTS device_metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    device_id TEXT NOT NULL,
    cpu_usage REAL,
    memory_usage REAL,
    disk_usage REAL,
    temperature REAL,
    rx_rate_kbps REAL,
    tx_rate_kbps REAL,
    created_at INTEGER NOT NULL
);
```

##### alarm_events 表

```sql
CREATE TABLE IF NOT EXISTS alarm_events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    device_id TEXT NOT NULL,
    alarm_type TEXT NOT NULL,
    level TEXT NOT NULL,
    message TEXT,
    recovered INTEGER DEFAULT 0,
    created_at INTEGER NOT NULL
);
```

##### process_status 表

```sql
CREATE TABLE IF NOT EXISTS process_status (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    device_id TEXT NOT NULL,
    process_name TEXT NOT NULL,
    pid INTEGER,
    running INTEGER NOT NULL,
    created_at INTEGER NOT NULL
);
```

##### service_status 表

```sql
CREATE TABLE IF NOT EXISTS service_status (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    device_id TEXT NOT NULL,
    service_name TEXT NOT NULL,
    status TEXT,
    active INTEGER NOT NULL,
    created_at INTEGER NOT NULL
);
```

---

### 4.6 通信模块 Reporter

#### 4.6.1 模块职责

通信模块负责将设备状态、心跳信息和告警事件上报到远程服务。

支持两种通信方式：

- MQTT
    
- HTTP
    

#### 4.6.2 通信接口

```cpp
class IReporter {
public:
    virtual ~IReporter() = default;
    virtual bool reportMetrics(const DeviceMetrics& metrics) = 0;
    virtual bool reportAlarm(const AlarmEvent& alarm) = 0;
    virtual bool reportHeartbeat(const std::string& device_id) = 0;
};
```

#### 4.6.3 MQTT Topic 设计

```text
device/{device_id}/status
device/{device_id}/alarm
device/{device_id}/heartbeat
device/{device_id}/process
device/{device_id}/service
```

#### 4.6.4 状态上报 JSON

```json
{
  "device_id": "linux_gateway_001",
  "timestamp": 1720000000,
  "cpu_usage": 45.6,
  "memory_usage": 62.1,
  "disk_usage": 70.3,
  "temperature": 55.2,
  "network": {
    "rx_rate_kbps": 128.5,
    "tx_rate_kbps": 64.2
  }
}
```

#### 4.6.5 告警上报 JSON

```json
{
  "device_id": "linux_gateway_001",
  "timestamp": 1720000000,
  "alarm_type": "CPU_HIGH",
  "level": "WARNING",
  "message": "CPU usage exceeded threshold",
  "recovered": false
}
```

#### 4.6.6 HTTP API 设计

```text
POST /api/device/register
POST /api/device/metrics
POST /api/device/alarm
POST /api/device/heartbeat
GET  /api/device/config/{device_id}
```

---

### 4.7 日志模块 Logger

#### 4.7.1 模块职责

日志模块负责记录系统运行信息、错误信息和调试信息。

#### 4.7.2 日志等级

```text
DEBUG
INFO
WARN
ERROR
FATAL
```

#### 4.7.3 日志格式

```text
[2026-07-09 10:30:21] [INFO] [collector] cpu usage: 42.5%
[2026-07-09 10:30:26] [WARN] [alarm] CPU_HIGH warning triggered
[2026-07-09 10:30:31] [ERROR] [mqtt] connect failed
```

#### 4.7.4 日志路径

```text
/var/log/device-monitor/device-monitor.log
```

---

## 5. 线程模型设计

使用多线程模型

---

## 6. 异常处理设计

|异常类型|处理方式|
|---|---|
|配置文件读取失败|使用默认配置或终止启动|
|`/proc` 文件读取失败|当前指标标记无效，记录日志|
|数据库打开失败|记录 ERROR，程序退出|
|数据库写入失败|记录 ERROR，继续运行|
|MQTT 连接失败|定时重连|
|HTTP 上报失败|记录失败，后续可补传|
|关键进程不存在|触发 PROCESS_DOWN 告警|
|systemd 服务异常|触发 SERVICE_DOWN 告警|
|程序异常退出|由 systemd 自动重启|

---

## 7. 部署设计

### 7.1 安装路径

```text
/usr/local/bin/device-monitor
/etc/device-monitor/config.json
/var/log/device-monitor/device-monitor.log
/var/lib/device-monitor/monitor.db
```

### 7.2 systemd 服务文件

```ini
[Unit]
Description=Linux Device Monitor Agent
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/device-monitor -c /etc/device-monitor/config.json
Restart=always
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
```

### 7.3 常用管理命令

```bash
sudo systemctl daemon-reload
sudo systemctl enable device-monitor
sudo systemctl start device-monitor
sudo systemctl status device-monitor
sudo systemctl restart device-monitor
sudo journalctl -u device-monitor -f
```

---

## 8. 目录结构设计

```text
linux-device-monitor/
├── CMakeLists.txt
├── README.md
├── docs/
│   ├── requirement.md
│   ├── detail_design.md
│   ├── database.md
│   ├── api.md
│   └── deployment.md
├── config/
│   └── config.json
├── scripts/
│   ├── install.sh
│   ├── uninstall.sh
│   └── run.sh
├── service/
│   └── device-monitor.service
├── include/
│   ├── common/
│   ├── config/
│   ├── collector/
│   ├── alarm/
│   ├── storage/
│   └── communication/
├── src/
│   ├── main.cpp
│   ├── common/
│   ├── config/
│   ├── collector/
│   ├── alarm/
│   ├── storage/
│   └── communication/
├── tests/
└── build/
```

---

## 9. 开发阶段划分

### 9.1 第一阶段：基础采集版本

目标：

完成本地设备状态采集。

实现内容：

- CMake 工程搭建
    
- 配置文件读取
    
- 日志输出
    
- CPU 采集
    
- 内存采集
    
- 磁盘采集
    
- 控制台打印状态
    

验收标准：

程序能够每隔 5 秒输出一次 CPU、内存和磁盘使用率。

---

### 9.2 第二阶段：告警版本

目标：

实现本地阈值告警。

实现内容：

- 告警规则配置
    
- 告警等级判断
    
- 连续触发机制
    
- 告警恢复机制
    
- 告警日志输出
    

验收标准：

当 CPU、内存或磁盘超过阈值时，系统能够产生 WARNING 或 CRITICAL 告警。

---

### 9.3 第三阶段：数据库版本

目标：

实现本地数据持久化。

实现内容：

- SQLite 初始化
    
- 设备状态入库
    
- 告警事件入库
    
- 进程状态入库
    

验收标准：

程序运行后能够在 SQLite 数据库中查询到历史状态和告警记录。

---

### 9.4 第四阶段：远程上报版本

目标：

实现设备状态和告警事件远程上报。

实现内容：

- MQTT 连接
    
- 状态上报
    
- 告警上报
    
- 心跳上报
    
- 断线重连
    

验收标准：

MQTT Broker 能够收到设备状态、心跳和告警消息。

---

### 9.5 第五阶段：服务部署版本

目标：

实现系统级部署。

实现内容：

- systemd 服务文件
    
- 安装脚本
    
- 卸载脚本
    
- 开机自启动
    
- 异常自动重启
    

验收标准：

程序可以通过 systemctl 进行启动、停止、重启和状态查看。

---

## 10. 测试设计

### 10.1 单元测试

重点测试以下模块：

- 配置解析模块
    
- CPU 采集模块
    
- 内存采集模块
    
- 告警判断模块
    
- SQLite 存储模块
    

### 10.2 集成测试

测试内容：

- 程序完整启动流程
    
- 指标采集到数据库保存流程
    
- 指标采集到告警触发流程
    
- 告警事件到 MQTT 上报流程
    
- systemd 服务启动流程
    

### 10.3 异常测试

测试内容：

- 删除配置文件后程序行为
    
- MQTT Broker 关闭后的重连行为
    
- 数据库路径无权限时的错误处理
    
- 关键进程退出后的告警行为
    
- CPU 压力升高后的告警行为
    

可以使用以下命令模拟 CPU 压力：

```bash
stress-ng --cpu 2 --timeout 60s
```

可以使用以下命令模拟磁盘压力：

```bash
dd if=/dev/zero of=test_large_file bs=100M count=10
```

---

## 11. 非功能性设计

### 11.1 稳定性

系统应支持长时间运行，异常退出后可由 systemd 自动拉起。

### 11.2 可扩展性

新增监控项时，只需要新增 Collector 类，并接入主控调度模块。

### 11.3 可配置性

采集周期、告警阈值、上报地址、监控进程和服务均应支持配置文件修改。

### 11.4 可维护性

系统采用模块化设计，不同功能之间通过接口交互，降低耦合。

### 11.5 资源占用

Agent 自身应尽量轻量化，避免对被监控设备造成明显负担。

建议目标：

- CPU 占用低于 3%
    
- 内存占用低于 50 MB
    
- 单次采集耗时低于 500 ms
    

---

## 12. 项目风险与应对方案

|风险|应对方案|
|---|---|
|功能范围过大|先完成 Agent 端核心功能|
|多线程开发复杂|第一版采用单线程主循环|
|MQTT 调试困难|先使用本地 Mosquitto 测试|
|采集指标不准确|对比 top、free、df 等命令验证|
|systemd 部署失败|先手动运行，再编写服务文件|
|数据库写入异常|增加错误日志和返回值检查|

---

## 13. 最终交付物

项目最终应包含以下内容：

```text
1. Linux 设备监控 Agent 源码
2. CMake 构建文件
3. JSON 配置文件
4. SQLite 数据库初始化逻辑
5. MQTT / HTTP 上报模块
6. systemd 服务文件
7. install.sh / uninstall.sh 脚本
8. README 使用说明
9. 详细设计文档
10. 测试说明文档
```

---

## 14. 简历描述建议

项目名称：

Linux 设备状态监控与预警系统

项目描述：

基于 C++ 开发 Linux 设备状态监控 Agent，作为 systemd 守护进程运行，周期性采集 CPU、内存、磁盘、网络、温度和关键进程状态，支持阈值告警、本地 SQLite 存储、MQTT/HTTP 远程上报和异常自动重启。

项目职责：

- 负责 Linux 设备状态采集模块设计与实现，通过 `/proc`、`/sys` 和 `statvfs` 获取系统运行指标。
    
- 设计告警引擎，支持连续触发、告警恢复、告警抑制和多级告警策略。
    
- 使用 SQLite 实现设备状态和告警事件本地持久化。
    
- 集成 MQTT/HTTP 通信模块，实现设备状态、心跳和告警事件远程上报。
    
- 使用 CMake 构建项目，并通过 systemd 实现后台服务部署和异常自动重启。
    
- 按照 Git 分支管理和规范化提交流程进行项目开发。
    

---

## 15. 版本规划

### V1.0 基础版

- CPU、内存、磁盘采集
    
- JSON 配置
    
- 本地日志
    
- 阈值告警
    

### V1.1 存储版

- SQLite 数据库存储
    
- 告警历史记录
    
- 进程状态监控
    

### V1.2 通信版

- MQTT 上报
    
- HTTP 上报
    
- 心跳机制
    
- 断线重连
    

### V1.3 部署版

- systemd 服务
    
- 安装脚本
    
- 日志路径规范
    
- 开机自启动
    

### V2.0 扩展版

- Web 后台展示
    
- 远程配置下发
    
- 多设备管理
    
- 数据可视化