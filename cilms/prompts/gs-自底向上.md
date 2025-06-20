基于您提供的文档开头部分，我来对CILMS（Container Initialization and Log Management System）项目进行模块化分解和优化。虽然我只能看到文档的开头部分，但我会基于已有信息进行合理的模块划分和提示词设计。

# CILMS项目模块化提示词设计

## 存在问题分析

根据文档开头的描述，我发现以下问题：

1. **术语不准确**：项目名称缩写CILMS与实际功能描述不完全匹配
2. **架构设计不清晰**：缺少系统架构图和组件关系说明
3. **需求描述冗余**：功能描述存在重复和交叉
4. **技术栈描述不够具体**：缺少具体的技术选型依据

## 模块化分解方案

### 1. 核心架构模块

#### 1.1 系统架构设计提示词

```markdown
# CILMS系统架构设计模块

## 角色定义
你是一名资深的系统架构师，专精于容器技术、进程管理和微服务架构设计。需要为CILMS项目设计清晰的系统架构。

## 任务目标
设计CILMS的整体系统架构，包括核心组件划分、接口定义和数据流转。

## 架构要求
1. **分层架构**：采用经典的分层架构模式
   - 表示层：CLI接口、HTTP API
   - 业务逻辑层：进程管理、日志管理、配置管理
   - 数据访问层：文件系统、进程状态存储
   - 基础设施层：系统调用封装、容器运行时接口

2. **组件职责划分**
   - 进程生命周期管理器
   - 日志收集与处理器
   - 配置管理器
   - 监控与健康检查器
   - API网关（可选）

3. **设计原则**
   - 单一职责原则
   - 依赖倒置原则
   - 接口隔离原则
   - 开闭原则

## 交付物
- 系统架构图
- 组件接口定义
- 数据流图
- 部署架构图
```

#### 1.2 核心接口设计提示词

```markdown
# CILMS核心接口设计模块

## 角色定义
你是一名资深的接口设计师，专精于Go语言接口设计和系统集成。

## 任务目标
设计CILMS各模块间的核心接口，确保模块间低耦合、高内聚。

## 接口设计要求
1. **进程管理接口**
   ```go
   type ProcessManager interface {
       Start(ctx context.Context, config ProcessConfig) error
       Stop(ctx context.Context, processName string) error
       Restart(ctx context.Context, processName string) error
       Status(processName string) ProcessStatus
       List() []ProcessInfo
   }
   ```

2. **日志管理接口**
   ```go
   type LogManager interface {
       Collect(source LogSource) error
       Process(logEntry LogEntry) error
       Store(logEntry ProcessedLogEntry) error
       Query(filter LogFilter) ([]LogEntry, error)
   }
   ```

3. **配置管理接口**
   ```go
   type ConfigManager interface {
       Load(configPath string) (*Config, error)
       Validate(config *Config) error
       Watch(configPath string) <-chan ConfigChangeEvent
   }
   ```

## 设计原则
- 接口应该小而专注
- 使用context进行超时和取消控制
- 错误处理要明确具体
- 支持并发安全
```

### 2. 进程管理模块

#### 2.1 进程生命周期管理提示词

```markdown
# 进程生命周期管理模块

## 角色定义
你是一名资深的系统程序员，专精于Linux进程管理、信号处理和系统调用。

## 任务目标
实现CILMS的进程生命周期管理功能，包括启动、停止、重启和监控。

## 核心功能
1. **进程启动管理**
   - 按优先级排序启动
   - 依赖关系检查
   - 启动参数处理
   - 环境变量设置

2. **进程监控**
   - 进程状态监控
   - 资源使用监控
   - 健康检查
   - 异常恢复

3. **进程停止管理**
   - 优雅停止（SIGTERM -> SIGKILL）
   - 依赖关系处理
   - 清理资源

## 技术要求
- 使用Go的os/exec包
- 实现信号处理机制
- 支持进程组管理
- 实现进程状态机

## 错误处理
- 启动失败重试机制
- 进程异常退出处理
- 资源泄露检测
- 错误日志记录

## 性能要求
- 启动时间优化
- 内存使用优化
- CPU使用率控制
```

#### 2.2 依赖管理提示词

```markdown
# 进程依赖管理模块

## 角色定义
你是一名资深的软件架构师，专精于依赖管理和有向无环图（DAG）算法。

## 任务目标
实现CILMS的进程依赖管理功能，确保进程按正确顺序启动和停止。

## 核心算法
1. **依赖图构建**
   - 解析依赖配置
   - 构建有向图
   - 检测循环依赖

2. **拓扑排序**
   - 实现Kahn算法
   - 处理并行启动
   - 依赖验证

3. **依赖检查**
   - 进程状态验证
   - 健康检查集成
   - 超时处理

## 数据结构
```go
type DependencyGraph struct {
    nodes map[string]*ProcessNode
    edges map[string][]string
}

type ProcessNode struct {
    Name     string
    Priority int
    Status   ProcessStatus
    Config   ProcessConfig
}
```

## 错误处理
- 循环依赖检测
- 依赖进程启动失败处理
- 超时处理机制
```

### 3. 日志管理模块

#### 3.1 日志收集提示词

```markdown
# 日志收集模块

## 角色定义
你是一名资深的日志系统工程师，专精于日志收集、处理和分析。

## 任务目标
实现CILMS的日志收集功能，支持多种日志源和格式。

## 核心功能
1. **日志源管理**
   - 标准输出/错误输出
   - 文件日志
   - 系统日志
   - 网络日志

2. **日志格式处理**
   - JSON格式
   - 纯文本格式
   - 结构化日志
   - 自定义格式

3. **日志缓冲管理**
   - 内存缓冲
   - 磁盘缓冲
   - 缓冲区大小控制
   - 刷新策略

## 技术实现
- 使用Go的bufio包
- 实现日志轮转
- 支持并发收集
- 内存管理优化

## 性能要求
- 低延迟收集
- 高吞吐量
- 内存使用控制
- CPU使用优化
```

#### 3.2 日志处理提示词

```markdown
# 日志处理模块

## 角色定义
你是一名资深的数据处理工程师，专精于流式数据处理和日志分析。

## 任务目标
实现CILMS的日志处理功能，包括过滤、格式化和路由。

## 核心功能
1. **日志过滤**
   - 级别过滤
   - 内容过滤
   - 时间范围过滤
   - 正则表达式过滤

2. **日志格式化**
   - 时间戳标准化
   - 字段提取
   - 格式转换
   - 结构化处理

3. **日志路由**
   - 基于规则的路由
   - 多目标输出
   - 条件分发
   - 负载均衡

## 处理流程
```
日志输入 -> 预处理 -> 过滤 -> 格式化 -> 路由 -> 输出
```

## 配置示例
```yaml
processors:
  - name: filter
    type: level
    config:
      min_level: info
  - name: format
    type: json
    config:
      timestamp_format: "2006-01-02T15:04:05Z07:00"
```
```

### 4. 配置管理模块

#### 4.1 配置解析提示词

```markdown
# 配置管理模块

## 角色定义
你是一名资深的配置管理专家，专精于配置文件解析和动态配置管理。

## 任务目标
实现CILMS的配置管理功能，支持多种配置格式和动态加载。

## 核心功能
1. **配置文件支持**
   - YAML格式（推荐）
   - JSON格式
   - TOML格式
   - 环境变量

2. **配置验证**
   - 语法验证
   - 语义验证
   - 依赖关系验证
   - 默认值处理

3. **动态配置**
   - 热重载
   - 配置监听
   - 变更通知
   - 回滚机制

## 配置结构
```go
type Config struct {
    Global    GlobalConfig    `yaml:"global"`
    Processes []ProcessConfig `yaml:"processes"`
    Logger    LoggerConfig    `yaml:"logger"`
}

type ProcessConfig struct {
    Name        string            `yaml:"name"`
    Command     string            `yaml:"command"`
    Args        []string          `yaml:"args"`
    Env         map[string]string `yaml:"env"`
    Priority    int               `yaml:"priority"`
    DependsOn   []string          `yaml:"depends_on"`
    Restart     RestartPolicy     `yaml:"restart"`
}
```

## 验证规则
- 进程名称唯一性
- 依赖关系有效性
- 命令路径存在性
- 配置项完整性
```

### 5. 监控与健康检查模块

#### 5.1 监控系统提示词

```markdown
# 监控系统模块

## 角色定义
你是一名资深的系统监控工程师，专精于系统监控、指标收集和告警管理。

## 任务目标
实现CILMS的监控功能，包括系统指标收集、进程监控和告警。

## 核心功能
1. **系统指标收集**
   - CPU使用率
   - 内存使用率
   - 磁盘I/O
   - 网络I/O

2. **进程监控**
   - 进程状态
   - 资源使用
   - 启动时间
   - 重启次数

3. **健康检查**
   - HTTP健康检查
   - TCP端口检查
   - 进程存活检查
   - 自定义检查脚本

## 指标格式
```go
type Metric struct {
    Name      string                 `json:"name"`
    Value     float64                `json:"value"`
    Labels    map[string]string      `json:"labels"`
    Timestamp time.Time              `json:"timestamp"`
    Type      MetricType             `json:"type"`
}
```

## 告警规则
- 阈值告警
- 趋势告警
- 异常检测
- 静默期管理
```

### 6. CLI接口模块

#### 6.1 命令行接口提示词

```markdown
# CLI接口模块

## 角色定义
你是一名资深的CLI工具开发者，专精于用户体验设计和命令行界面开发。

## 任务目标
实现CILMS的命令行接口，提供友好的用户交互体验。

## 核心命令
1. **进程管理命令**
   ```bash
   cilms start [process-name]     # 启动进程
   cilms stop [process-name]      # 停止进程
   cilms restart [process-name]   # 重启进程
   cilms status [process-name]    # 查看状态
   cilms list                     # 列出所有进程
   ```

2. **日志管理命令**
   ```bash
   cilms logs [process-name]      # 查看日志
   cilms logs -f [process-name]   # 跟踪日志
   cilms logs --since=1h          # 时间范围日志
   ```

3. **配置管理命令**
   ```bash
   cilms config validate          # 验证配置
   cilms config reload            # 重载配置
   cilms config show              # 显示配置
   ```

## 用户体验
- 彩色输出
- 进度条显示
- 自动补全
- 帮助信息
- 错误提示优化

## 技术实现
- 使用cobra库
- 支持配置文件
- 环境变量支持
- 日志级别控制
```

### 7. 测试模块

#### 7.1 单元测试提示词

```markdown
# 单元测试模块

## 角色定义
你是一名资深的测试工程师，专精于Go语言测试、测试驱动开发和测试自动化。

## 任务目标
为CILMS各模块编写完善的单元测试，确保代码质量和功能正确性。

## 测试策略
1. **测试覆盖率**
   - 行覆盖率 > 80%
   - 分支覆盖率 > 70%
   - 函数覆盖率 > 90%

2. **测试类型**
   - 单元测试
   - 集成测试
   - 端到端测试
   - 性能测试

3. **测试工具**
   - Go标准测试库
   - testify断言库
   - GoMock模拟库
   - 基准测试

## 测试用例设计
```go
func TestProcessManager_Start(t *testing.T) {
    tests := []struct {
        name    string
        config  ProcessConfig
        wantErr bool
    }{
        {
            name: "valid process",
            config: ProcessConfig{
                Name:    "test-process",
                Command: "/bin/echo",
                Args:    []string{"hello"},
            },
            wantErr: false,
        },
        // 更多测试用例...
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // 测试逻辑
        })
    }
}
```

## 测试最佳实践
- 测试隔离
- 测试数据管理
- 错误场景测试
- 并发测试
```

### 8. 部署与运维模块

#### 8.1 容器化部署提示词

```markdown
# 容器化部署模块

## 角色定义
你是一名资深的DevOps工程师，专精于容器技术、CI/CD和运维自动化。

## 任务目标
设计CILMS的容器化部署方案，包括Docker镜像构建和Kubernetes部署。

## 镜像构建
```dockerfile
# 多阶段构建
FROM golang:1.24.2-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o cilms .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/cilms .
COPY --from=builder /app/config.yaml .
CMD ["./cilms"]
```

## Kubernetes部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cilms
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cilms
  template:
    metadata:
      labels:
        app: cilms
    spec:
      containers:
      - name: cilms
        image: cilms:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: config
          mountPath: /etc/cilms
      volumes:
      - name: config
        configMap:
          name: cilms-config
```

## 运维监控
- 健康检查端点
- 指标暴露
- 日志收集
- 告警配置
```

## 优化建议总结

### 1. 术语和命名优化
- **问题**：CILMS缩写与功能不完全匹配
- **建议**：改为PLMS（Process and Log Management System）或保持CILMS但明确定义为"Container Infrastructure and Log Management System"

### 2. 架构设计优化
- **问题**：缺少清晰的架构设计
- **建议**：采用分层架构，明确各层职责

### 3. 技术选型优化
- **问题**：技术栈描述不够具体
- **建议**：
  - 使用Cobra库构建CLI
  - 使用Viper进行配置管理
  - 使用Logrus或Zap进行日志处理
  - 使用Prometheus格式暴露监控指标

### 4. 开发流程优化
- **问题**：缺少开发流程规范
- **建议**：
  - 采用TDD开发模式
  - 使用Git Flow分支管理
  - 实施代码审查
  - 自动化CI/CD

### 5. 文档结构优化
- **问题**：需求描述冗余
- **建议**：
  - 按功能模块组织文档
  - 使用统一的格式模板
  - 添加架构图和流程图
  - 包含API文档

这个模块化的提示词设计遵循了软件工程的最佳实践，每个模块都有明确的职责边界，便于团队协作开发和维护。