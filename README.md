# Rinne-IM 开发文档

Rinne-IM 是一个基于 Golang (1.25+) 开发的高性能即时通讯系统，采用 gRPC 作为通信协议，结合 Kafka 进行消息解耦与削峰，Redis 作为热点缓存与状态存储，PostgreSQL 作为持久化存储。

本文档详细介绍了系统的开发环境搭建、配置说明、协议生成及部署流程。

---

## 🛠 技术栈概览

| 组件           | 版本   | 说明                                    |
| :------------- | :----- | :-------------------------------------- |
| **Golang**     | 1.25.2 | 主要开发语言                            |
| **gRPC**       | 1.64+  | 高性能 RPC 框架 (Protobuf v3)           |
| **PostgreSQL** | 15.x   | 核心关系型数据库 (User/Message/Session) |
| **Redis**      | 7.x    | 缓存、Session 状态、PUBSUB              |
| **Kafka**      | 3.5    | 消息队列 (异步消息投递)                 |
| **Viper**      | 1.21   | 配置文件读取 (支持热更、环境变量覆盖)   |
| **Docker**     | Latest | 容器化部署与编排                        |

---

## 📂 项目结构说明

```
Rinne-IM/
├── cmd/
│   ├── im_server/          # IM 服务端主入口 (main.go)
│   └── client_test/        # gRPC 客户端测试工具 (用于联调)
├── config/                 # 配置结构定义及加载逻辑 (Viper)
├── internal/
│   ├── repository/         # 数据访问层 (DAO), 封装 DB 操作
│   └── service/            # 业务逻辑层, 实现 gRPC 接口
├── models/                 # 基础数据模型与 DB 连接初始化
├── queue/                  # 消息队列封装 (Kafka Producer/Consumer)
├── service/pb/             # Protobuf IDL 定义 (.proto) 及生成代码 (.pb.go)
├── sql/migrations/         # 数据库初始化 SQL 脚本
├── tests/                  # 单元测试与集成测试
├── docker-compose.yaml     # 本地开发环境编排 (DB, Redis, Kafka)
├── Dockerfile              # 应用构建镜像文件
└── go.mod                  # 依赖管理
```

---

## ⚡ 快速开发指南

### 1. 前置要求

确保本地已安装以下工具：

- **Golang** (>= 1.25)
- **Docker** & **Docker Compose** (用于启动依赖设施)
- **Protoc** (Protocol Buffers 编译器, 用于重新生成 gRPC 代码)
  - 插件: `protoc-gen-go`, `protoc-gen-go-grpc`

### 2. 启动基础设施

在开发过程中，建议使用 Docker 启动数据库和消息队列，而在本地直接运行 Go 代码以方便调试。

```powershell
# 启动 Postgres, Redis, Kafka, Zookeeper
docker-compose up -d postgres redis kafka zookeeper
```

_等待约 30 秒确保 Kafka 与 Zookeeper 完成启动与选举。_

### 3. 环境配置

项目默认读取 `config/config.toml`。本地开发时需确认配置指向正确。

**默认开发配置 (config.toml)**:

```toml
[database]
host = "localhost" # 若在 WSL/容器外运行，请确保端口映射正确
port = "5432"
...

[kafka]
brokers = ["localhost:9094"] # Docker 暴露在宿主机的端口
...
```

> **注意**: 如果在 WSL2 中运行 Go 程序，连接 Docker Desktop 容器时应使用宿主机 IP (如 `192.168.x.x`) 或确保 `localhost` 转发正常。

### 4. 运行服务

#### 启动 IM 服务端

```powershell
# 下载依赖
go mod tidy

# 运行服务
go run cmd/im_server/main.go
```

_成功启动后日志显示: `IM Server ... listening at :8082`_

#### 运行测试客户端

我们提供了一个简单的 CLI 客户端用于测试注册、登录和消息发送。

```powershell
go run cmd/client_test/main.go
```

---

## 🔧 配置详解

配置系统支持 `config.toml` 文件及环境变量覆盖（适合 Docker 部署）。

**环境变量规则**: 前缀 `RINNE_`，层级用 `_` 分隔。
例如：`database.host` -> `RINNE_DATABASE_HOST`

| 配置项          | TOML Key         | 环境变量 (Example)     | 说明                       |
| :-------------- | :--------------- | :--------------------- | :------------------------- |
| **DB Host**     | `database.host`  | `RINNE_DATABASE_HOST`  | 数据库地址                 |
| **Redis Pwd**   | `redis.password` | `RINNE_REDIS_PASSWORD` | Redis 密码                 |
| **Kafka Host**  | `kafka.brokers`  | `RINNE_KAFKA_BROKERS`  | Kafka 节点列表 (逗号分隔)  |
| **Server Port** | `server.port`    | `RINNE_SERVER_PORT`    | gRPC 监听端口 (默认 :8082) |

---

## 📝 Protocol Buffers 协议更新

当修改了 `service/pb/im.proto` 文件后，需要重新生成 Go 代码。

**执行命令 (在项目根目录)**:

```powershell
protoc --go_out=. --go_opt=paths=source_relative `
       --go-grpc_out=. --go-grpc_opt=paths=source_relative `
       service/pb/im.proto
```

_Linux/Mac 用户请将反引号 The `替换为`\`_

---

## 💾 数据库迁移

SQL 初始化脚本位于 `sql/migrations/public.sql`。

- **首次启动**: `docker-compose` 会自动挂载此脚本到 Postgres 容器的 `/docker-entrypoint-initdb.d/` 目录进行初始化。
- **后续更新**: 需手动连接数据库执行 SQL 语句。

**核心表结构**:

- `users`: 用户信息 (ID, Username, PasswordHash)
- `sessions`: 会话关系 (UserID, TargetID, GroupID)
- `messages`: 聊天记录 (MsgID, Sender, Receiver, Content)

---

## ❓ 常见问题排查

### 1. Kafka 连接失败 "Broker may not be available"

- 检查 `config.toml` 中的 `kafka.brokers`。如果是 Docker 启动，外部访问一定要用 `9094` 端口（我们在 docker-compose 中配置了 EXTERNAL listener）。
- 检查 Docker 容器状态: `docker ps` 查看 `rinne-kafka` 是否处于 Up 状态。

### 2. gRPC 客户端报错 "connection refused"

- 确认服务端端口。目前默认配置为 **8082** (避免与 Kafka-UI 的 8080 冲突)。
- 检查防火墙或网络连通性 (如 WSL 与 Windows 之间的网络桥接)。

### 3. 数据库 "duplicate key value"

- 测试客户端生成的 UserID 是基于 Snowflake 算法的，但 Username 有唯一约束。测试代码中已加入时间戳后缀已避免冲突。

### 4. 怎样查看 Kafka 消息？

- 访问本地部署的 **Kafka UI**: [http://localhost:8080](http://localhost:8080)
- 可以直接查看 Topic `im_msg_transfer_group` 中的消息流转情况。

---

## 🐳 全容器化部署

如果需要在服务器部署完整环境：

```bash
# 构建并后台运行
docker-compose up -d --build

# 查看日志
docker-compose logs -f im-server
```
