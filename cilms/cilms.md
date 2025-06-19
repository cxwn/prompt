# CILMS进程与日志管理系统开发提

## 1. 角色定义

你是一名资深的Go语言开发工程师，专精于Linux系统编程、进程管理、日志管理和容器技术。你需要设计并实现一个高性能、高可靠性的进程与日志管理工具。

## 2. 项目概述

**项目名称**: CILMS (Container Initialization and Log Management System)。  
**核心目标**: 开发一款适用于Linux容器环境和非容器环境下通用的进程与日志管理工具。  
**技术要求**: 使用Go 1.24.2实现，在容器环境下支持作为容器1号进程或守护进程运行。
**开发环境**: windows 10专业版，已开启WSL2，Linux发行版为Ubuntu-24.04，已安装Docker 27.2.1，测试容器openanolis/anolisos:8.6已拉取至本地。
**运行环境**: Linux系列容器及物理机、虚拟机，操作系统内核版本高于3.10。

## 3. 核心功能需求

### 3.1 进程生命周期管理

#### 3.1.1 启动模式

- **容器模式（PID 1）**

  - **前台模式**（默认）: CILMS启动所有被管理的进程后，自身仍作为容器1号进程持续运行，阻止容器退出；
  - **后台模式**（`-d`）: CILMS启动所有子进程后转入后台，不主动阻止容器退出。
  
- **非容器模式**

  - **前台模式**（默认）: CILMS阻塞当前终端，显示实时日志；
  - **守护模式**（`-d`）: CILMS作为守护进程在后台运行，不阻塞终端。

**注意：**容器和非容器环境下，CILMS均可以root或其他用户身份运行。

#### 3.1.2 进程启动策略

- **优先级排序**: 按配置文件中的priority字段升序启动（数值越小优先级越高），相同优先级按配置文件顺序。
- **依赖管理**
  - 进程启动前必须等待其`dependsOn`中列出的所有进程启动成功、状态正常；
  - 依赖检查基于进程名称，不区分同名进程实例；
  - 如依赖的进程名有多个实例，需等待所有实例启动成功。
- **同名进程启动控制**: 同名进程按优先级进行启动，如优先级相同，则按配置文件中出现的顺序依次启动。
- **环境变量**: 支持进程级环境变量配置和变量替换，启动参数中如包含变量，需进行替换后在执行启动。

#### 3.1.3 进程监控机制

监控指标包括:

- 进程状态: 运行中/停止/重启中/失败；
- 运行时间: 启动时间/停止时间/累计运行时长；
- 重启统计: 重启次数/最后重启时间/重启原因；
- 退出信息: 退出码/退出时间/退出原因。

#### 3.1.4 异常进程处理

- **僵尸进程**: 自动检测并回收，重启失败则触发全量重启；
- **孤儿进程**: 检测后终止并触发全量重启；
- **检测周期**: 每30秒执行一次检测。

### 3.2 重启策略系统

#### 3.2.1 重启模式

- **always**: 进程无论何种原因退出都重启（默认）；
- **onFailure**: 仅在异常退出（非零退出码）时重启；
- **never**: 永不自动重启。

#### 3.2.2 重启算法

退避算法:
第1次: 立即重启
第2次: 延迟10秒
第3次: 延迟20秒
第4次: 延迟40秒
第N次: min(10 * 2^(N-2), 300) 秒 （最大延迟5分钟）

重置条件:

- 进程连续稳定运行超过600秒（10分钟）后，重启计数器归零；
- 重启计数器达到maxRetries上限后的处理：
  - 容器环境：CILMS（PID 1）退出，导致容器停止；
  - 非容器环境：CILMS进程退出，停止管理所有子进程。

#### 3.2.3 优雅停止机制

优雅停止:

  1. 发送SIGTERM信号
  2. 等待stopTimeout秒（默认10秒）
  3. 发送SIGKILL强制终止
  4. 按启动优先级逆序停止进程

### 3.3 日志管理

#### 3.3.1 系统日志

格式规范:

- 时间格式: 2023-08-01 20:00:00.000
- 日志信息以纯英语呈现
- 级别宽度: 固定5字符左对齐 [DEBUG|INFO |WARN |ERROR|FATAL]
- 模板: {时间} {级别} {PID} {文件}:{行号} {消息}
- 示例: 2023-08-01 20:00:00.000 INFO  1234 cilms/main.go:100 系统启动完成

#### 3.3.2 输出配置

- **双重输出**: 同时支持控制台和文件输出;
- **格式支持**: text和json两种格式, 默认为text;
- **级别控制**: debug/info/warn/error/fatal五个级别;
- **颜色支持**: 控制台输出支持颜色区分。

#### 3.3 日志轮转(被管理的子进程的日志及自身日志轮转)

- **系统日志轮转**: 基于文件大小和时间的自动轮转；
- **应用日志轮转**: 管理受控进程的日志文件轮转；
- **压缩归档**: 支持历史日志压缩存储。

## 4. 接口设计规范

### 4.1 命令行接口

#### 4.1.1 主命令

```bash
# 启动CILMS服务
cilms start [-d|--daemon] [-p|--port PORT] [-c|--config FILE]

# 停止被管理的进程，最终停止CILMS服务  
cilms stop

# 重启CILMS服务或重启指定PID进程
cilms restart [--pid PID]

# 状态查询
cilms list [--all|--pid PID]

# 日志查看
cilms logs [--follow] [--number NUM]

# 僵尸进程管理
cilms zombies [--clean] [--restart]

# 孤儿进程管理  
cilms orphans [--clean] [--restart]
```

#### 4.1.2 参数说明

- `--daemon`: 守护进程模式运行
- `--port`: HTTP监听端口（默认10168）
- `--config`: 配置文件路径（默认/etc/cilms/cilms.yaml）
- `--follow`: 实时跟踪日志输出
- `--number`: 显示日志行数（默认100）

### 4.2 HTTP API接口

#### 4.2.1 响应格式标准

- 成功响应

  ```json
  {
    "status": 200,
    "message": "Success", 
    "data": {}
  }

- 错误响应

```json
{
  "status": 400,
  "message": "Error description",
  "error": "Detailed error reason"
}
```

#### 4.2.2 API端点

```text
GET    /cilms/processes              # 获取所有进程列表
GET    /cilms/processes/{pid}        # 获取指定进程信息
POST   /cilms/processes/{pid}/restart # 重启指定进程
POST   /cilms/stop                   # 停止所有进程
POST   /cilms/restart                # 重启所有进程
GET    /cilms/zombies               # 获取僵尸进程列表
POST   /cilms/zombies/clean         # 清理僵尸进程
GET    /cilms/orphans               # 获取孤儿进程列表
POST   /cilms/orphans/clean         # 清理孤儿进程
```

## 5. 技术规范

### 5.1 技术栈要求

- **语言版本**: Go 1.24.2
- **CLI框架**: urfave/cli/v2
- **日志库**: logrus
- **日志轮转**: lumberjack.v2
- **HTTP框架**: gin-gonic/gin
- **配置解析**: gopkg.in/yaml.v3

### 5.2 配置文件规范

```yaml
global:
  port: 10168                    # HTTP监听端口
  logger:
    file: "logs/cilms.log"       # 日志文件路径
    level: "info"                # 日志级别
    rotate: true                 # 启用日志轮转
    maxSize: 100                 # 单文件最大大小(MB)
    maxBackups: 5                # 保留历史文件数
    maxAge: 10                   # 保留天数
    compress: true               # 启用压缩
    console: true                # 控制台输出
    format: "text"               # 输出格式: text/json，二者选其一

processes:
  - name: "app"                  # 进程名称(可能重复)
    environment:
      enable: true               # 是否启用该进程
      variables:                 # 环境变量配置
        - name: "VAR_NAME"       # 环境变量名称
          value: "var_value"     # 环境变量值
    dependsOn: []                # 依赖进程列表
    directory: "/cilms/app"      # 工作目录
    command: "app"               # 可执行文件
    args: ["-p", "${APP_PORT}"]  # 命令行参数（支持环境变量替换）
    priority: 0                  # 启动优先级(数值越小优先级越高)
    restartPolicy: "always"      # 重启策略: always/onFailure/never
    maxRetries: 5                # 最大重试次数
    stopTimeout: 10              # 优雅停止超时时间（秒）
    logRotate:                   # 被启动进程日志轮转配置
      rotate: true               # 是否启动被管理进程的日志轮转
      file: "/var/log/app/app.log" # 被管理进程的日志路径
      maxSize: 128                 # 单文件最大大小(MB)
      maxDays: 7                   #  保留天数
      maxBackups: 5                # 保留历史文件数
      compress: true               # 是否压缩
      checkInterval: 60            # 日志轮转检查周期，默认60秒
  - name: "demo"
      environment: 
        enable: true
        variables:
          - name: "PORT"
            value: "8080"
          - name: "DB_HOST"
            value: "localhost"
      dependsOn: [ "app" ] 
      directory: "/cilms/demo"
      command: "demo"
      args: ["--port", "${PORT}", "--db-host", "${DB_HOST}"]
      priority: 0 
      restartPolicy: "always" 
      maxRetries: 5 
      stopTimeout: 10 
      logRotate:
        rotate: false  # 不启动被管理应用进程的日志轮转
```

## 6. 实现要求

### 6.1 代码质量标准

- **架构设计**: 提供系统架构图和主要流程图;
- **模块划分**: CLI和HTTP接口独立实现，不交叉调用;
- **错误处理**: 完善的错误处理和异常恢复机制;
- **并发安全**: 使用sync包确保并发安全;
- **单元测试**: 核心功能提供单元测试;
- **注释文档**: 以纯正英语进行注释，包括详尽的代码注释和文档说明。

### 6.2 性能要求

- **启动时间**: 系统启动时间不超过5秒；
- **资源占用**: 空闲状态下内存占用不超过50MB；
- **响应速度**: HTTP接口响应时间不超过500ms；
- **稳定性**: 支持7×24小时连续运行。

### 6.3 兼容性要求

- **操作系统**: Linux内核3.10+
- **容器运行时**: Docker、Podman、containerd
- **架构支持**: amd64、arm64

## 7 交付物要求

### 7.1 必需交付内容

1. **完整源码**: 包含所有功能模块的Go源代码
2. **架构文档**: 系统架构图、核心流程图、时序图
3. **构建脚本**: Makefile和Dockerfile
4. **配置示例**: 完整的配置文件示例
5. **使用文档**: README.md和用户手册
6. **测试代码**: 单元测试和集成测试

### 7.2 代码组织结构（可根据实际情况进行扩展，不要过渡发挥）

```text
cilms/
├── bin/                           # 编译结果输出目录
├── cmd/
│   ├── start.go                   # 启动命令入口，具体实现在cli/commands.go
│   ├── stop.go                    # 停止命令入口，具体实现在cli/commands.go
│   ├── restart.go                 # 重启命令入口，具体实现在cli/commands.go
│   ├── list.go
│   ├── zombies.go
│   ├── orphans.go
│   └── logs.go  
├── internal/
│   ├── api/                        # HTTP API 实现，实际执行时调用cli/commands.go下主要实现函数
│   │   └── server.go
│   ├── cli
│   │   └── commands.go             # 命令行接口主要实现函数
│   ├── process/                    # 核心业务逻辑
│   │   ├── manager.go              # 进程管理器
│   │   ├── process.go              # 进程定义和状态
│   │   ├── monitor.go              # 进程监控
│   │   └── dependency.go           # 依赖管理
│   ├── config/                     # 配置管理
│   │   ├── config.go               # 全局加载一次，在读取命令行入参或配置文件值时，使用默认值进行初始化
│   │   └── validation.go           # 配置校验
│   ├── logger/                     # 日志系统
│   │   ├── logger.go               # 配置管理初始化后，使用配置管理初始化后的配置进行初始化，全局初始化一次，线程安全
│   │   └── rotator.go              # 日志轮转
│   └── utils/                      # 工具函数
│       ├── signal.go
│       ├── pid.go
│       └── daemon.go
├── pkg/                            # 对外暴露的包
├── configs/
│   └── cilms.yaml                  # 示例配置文件
├── scripts/               # 构建、演示脚本
├── docs/                  # 架构图、流程图、时序图及其他文档
├── go.mod
├── go.sum
├── cilms.go               # 主程序入口
├── Dockerfile
├── Makefile
├── README.md
└── tests/                 # 测试文件
```

## 8. 参考资源

- **开源项目**: <https://github.com/Supervisor/supervisor.git>，<https://github.com/ochinchina/supervisord.git>，从以上仓库获取灵感，但不要简单复制代码。您的实现应具有原创性，并针对本项目的特定需求进行定制。
- **设计理念**: 参考supervisord的设计思想，但针对容器环境优化;
- **最佳实践**: 遵循Go语言和Linux系统编程最佳实践。

**注意**: 实现时请确保每个功能模块都有清晰的接口定义和完整的错误处理机制。代码应当具备良好的可读性、可维护性和可扩展性。对于任何来自没有同行评审机制或者原始的来源的代码，应该仔核实、验证，确保代码逻辑正确、可运行。一次性输出所有交付内容。
