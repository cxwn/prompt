# Go 项目日志模块开发指南 (基于 Zap)

## 1. 角色与目标

你是一名经验丰富的 Go 语言开发者，专长于构建高可用、高性能的后端服务。

你的任务是为我们的 Go 项目设计并实现一个日志模块。这个模块将作为项目的基础设施，被所有其他服务和组件统一调用。核心目标是创建一个**可配置、高性能、线程安全且具备真正线程隔离**的日志系统。

## 2. 核心技术栈

- **日志库**: `go.uber.org/zap`
- **日志轮转**: `gopkg.in/natefinch/lumberjack.v2`

## 3. 关键特性需求 (必须全部实现)

请根据以下关键特性来设计你的代码：

1.  **全局单例 (Singleton)**:

    - 提供一个 `InitLogger(conf Config)` 函数，用于在程序启动时根据配置初始化全局 Logger。
    - 提供一个全局可访问的 Logger 实例（例如 `log.L()`），方便在项目任何地方调用，无需传递 Logger 实例。

2.  **高度可配置**:

    - 创建一个 `Config` 结构体，用于封装所有日志配置。该结构体应能方便地从 YAML 或 JSON 配置文件中反序列化。
    - 配置项应包括：
      - `Level`: 日志级别 (如: `debug`, `info`, `warn`, `error`, `fatal`)。
      - `Format`: 日志格式 (`json` 或 `text`)。
      - 支持同时输出到控制台和文件。
      - 提供结构化日志记录能力。
      - `Rotation`: 日志轮转配置 (内嵌`lumberjack`的配置)。
        - `MaxSize`: 单个日志文件的最大尺寸 (MB)。
        - `MaxAge`: 日志文件的最长保留天数。
        - `MaxBackups`: 最多保留的旧日志文件数。
        - `Compress`: 是否对轮转的日志文件进行压缩。
        - `LocalTime`: 是否使用本地时间进行轮转。

3.  **最佳性能**:

    - 使用 `zap.NewProduction()` 或 `zap.NewDevelopment()` 作为基础配置，并根据`Format`配置进行自定义。
    - 默认使用 `*zap.Logger` 以获得极致性能。同时提供一个 `*zap.SugaredLogger` 的全局实例，用于对性能要求不高的场景。
    - 编码器 (`Encoder`) 应根据配置动态选择 `zapcore.NewJSONEncoder` 或 `zapcore.NewConsoleEncoder`。

4.  **线程安全**:

    - `zap` 本身是线程安全的，确保你的封装和全局实例的访问方式不会破坏这一点，需要注意配置和初始化的并发安全。
    - 使用sync.Once确保初始化只执行一次。
    - 对于动态配置更新，使用适当的锁机制（如读写锁）。

5.  **线程隔离 (核心要求)**:

    - **目标**: 必须防止日志 I/O 操作阻塞业务逻辑的`goroutine`。例如，当磁盘写满或网络日志端点阻塞时，业务代码的执行**不应**受到任何影响。
    - **实现方案**:
      - **不要直接使用 `zapcore.AddSync(&lumberjack.Logger{...})`**，因为当 `lumberjack` 的 `Write` 方法因磁盘 I/O 而阻塞时，它会直接阻塞调用日志记录的 `goroutine`。
      - **正确做法**: 实现一个**异步、非阻塞的 `WriteSyncer`**。你可以通过一个带有缓冲区的 `channel` 和一个专用的日志写入 `goroutine` 来实现。
        1.  创建一个自定义的 `WriteSyncer` 结构体，其 `Write` 方法将日志数据 (`[]byte`) 发送到一个内部的 `chan []byte`。
        2.  在 `InitLogger` 中启动一个**单独的 `goroutine`**。这个 `goroutine` 循环地从 `channel` 中接收日志数据。
        3.  在这个专用的 `goroutine` 中，调用 `lumberjack` 的 `Write` 方法将数据写入磁盘。
        4.  **错误处理**: 如果 `channel` 已满（意味着日志生产速度远超消费速度），`Write` 方法应立即返回，可以选择丢弃该条日志，以保证业务 `goroutine` 不被阻塞。同时，可以在`stderr`中打印一条关于日志丢弃的警告信息。
        5.  这个设计将所有磁盘 I/O 的潜在阻塞都隔离在了专用的日志 `goroutine` 中，从而实现了业务与日志的线程隔离。

6.  **最佳实践**:
    - 在日志输出中自动包含调用者信息 (`zap.AddCaller()`)。
    - 对于 `Error` 级别及以上的日志，自动记录堆栈信息 (`zap.AddStacktrace(zap.ErrorLevel)`)。
    - 提供清晰的初始化失败处理逻辑。如果 `InitLogger` 失败，应该 `panic` 或者返回一个明确的 `error`，以防止程序在没有日志的情况下运行。
    - 实现优雅的关闭机制，确保日志缓冲区正确刷新。
    - 提供便捷的字段添加方法（WithField、WithFields）。
    - 集成请求ID追踪，便于分布式系统调试。
    - 实现错误堆栈追踪功能。
    - 支持敏感信息脱敏。

7.  **初始化流程**
    1. 读取配置文件或环境变量。
    2. 创建zap核心配置。
    3. 设置编码器（JSON/Console）。
    4. 配置输出目标（文件/控制台）。
    5. 设置日志轮转策略。
    6. 创建全局logger实例。

## 4. 预期输出

请提供一个完整的 Go 代码文件 (`logger.go`)，其中包含：

1.  `Config` 结构体的定义。
2.  `InitLogger(conf Config)` 函数的完整实现。
3.  获取全局`Logger`和`SugaredLogger`的函数。
4.  实现线程隔离的异步 `WriteSyncer`。
5.  详尽的代码注释，解释关键部分的设计思想，特别是关于**线程隔离**的实现方式。
6.  一个简单的 `main.go` 示例，演示如何：
    - 从一个模拟的 YAML 配置中加载 `Config`。
    - 调用 `InitLogger`。
    - 在不同的 `goroutine` 中使用全局`Logger`记录日志。
7. 提供高度封装的`Debug`、`Info`、`Warn`、`Error`、`Fatal`、`WithField`、`WithFields`、`Debugf`、`Infof`、`Warnf`、`Errorf`、`Fatalf`等函数，方便记录不同级别的日志。

请将最终的代码以完整的、可直接运行的 Go 文件形式呈现。
