# Rinne-IM 即时通讯系统

<p align="center">
  <img src="https://img.shields.io/badge/Go-1.25+-00ADD8?style=for-the-badge&logo=go&logoColor=white" alt="Go Version">
  <img src="https://img.shields.io/badge/Node.js-18.x+-339933?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js Version">
  <img src="https://img.shields.io/badge/gRPC-Latest-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white" alt="gRPC">
  <img src="https://img.shields.io/badge/PostgreSQL-15.x-336791?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Redis-7.x-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis">
  <img src="https://img.shields.io/badge/Kafka-3.5-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white" alt="Kafka">
</p>

Rinne-IM 是一个现代化的高性能即时通讯系统，采用微服务架构设计，支持用户注册登录、好友管理、群组聊天、实时消息传递等核心功能。系统基于 Golang 和 Node.js 构建，使用 gRPC 作为通信协议，结合 Kafka 进行消息解耦，Redis 提供缓存支持，PostgreSQL 作为持久化存储。

## 🌟 核心特性

- **实时通讯**: 基于 gRPC 双向流实现毫秒级消息传递
- **多端支持**: Electron 桌面客户端 + Web 端支持
- **消息可靠性**: Kafka 消息队列保证消息不丢失
- **高性能**: Redis 缓存 + PostgreSQL 持久化存储
- **安全认证**: JWT Token 认证机制
- **可扩展架构**: 微服务设计，易于水平扩展

## 🏗️ 技术架构

### 服务端 (Rinne-IM-Server)
```
┌─────────────────┐    gRPC    ┌─────────────────┐
│   Electron 客户端 │ ◄─────────► │   IM 服务端    │
└─────────────────┘            └─────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
              ┌──────────┐      ┌──────────┐      ┌──────────┐
              │ PostgreSQL │      │   Redis   │      │   Kafka   │
              │  (持久化)  │      │ (缓存/状态) │      │ (消息队列) │
              └──────────┘      └──────────┘      └──────────┘
```

**技术栈**:
- **语言**: Golang 1.25+
- **通信协议**: gRPC (Protocol Buffers v3)
- **数据库**: PostgreSQL 15.x
- **缓存**: Redis 7.x
- **消息队列**: Apache Kafka 3.5
- **配置管理**: Viper
- **容器化**: Docker & Docker Compose

### 客户端 (Rinne-IM-Client)
**技术栈**:
- **框架**: React 19 + TypeScript
- **桌面应用**: Electron 40.x
- **UI库**: Tailwind CSS + Framer Motion
- **路由**: React Router DOM 7.x
- **gRPC客户端**: @grpc/grpc-js
- **构建工具**: Vite 7.x

## 🖥️ 客户端详细介绍

### 🏗️ 客户端架构设计

Rinne-IM-Client 采用现代化的前端架构设计，基于 React 和 Electron 构建跨平台桌面应用：

```
┌─────────────────────────────────────────────────────────────┐
│                    Electron 主进程                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   渲染进程   │  │   IPC 通信   │  │   系统托盘/菜单     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   React UI   │      │  gRPC 客户端 │      │   状态管理   │
│  (组件树)    │◄────►│   (通信层)   │◄────►│  (Context)   │
└─────────────┘      └─────────────┘      └─────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   路由系统   │      │   Protobuf   │      │   工具函数   │
│ (React Router)│     │  (协议定义)  │      │  (Utils)    │
└─────────────┘      └─────────────┘      └─────────────┘
```

### 🧩 主要功能模块

#### 1. 用户认证模块
- **登录注册**: JWT Token 认证机制
- **会话管理**: 自动登录状态保持
- **设备绑定**: 多设备登录支持

#### 2. 联系人管理模块
- **好友列表**: 实时好友状态显示
- **用户搜索**: 全局用户搜索功能
- **好友申请**: 好友请求发送与处理

#### 3. 聊天通信模块
- **实时消息**: 基于 gRPC 双向流的即时通讯
- **消息类型**: 文本、图片、文件、表情包支持
- **消息状态**: 发送中、已送达、已读回执

#### 4. 群组功能模块
- **群组创建**: 群聊创建与管理
- **成员管理**: 邀请、踢出群成员
- **群组设置**: 群名称、公告、权限设置

#### 5. 会话管理模块
- **最近会话**: 按时间排序的会话列表
- **会话标签**: 未读消息数、置顶会话
- **历史记录**: 消息历史查询与加载

### 🔌 与服务端交互方式

客户端通过 gRPC 协议与服务端进行高效通信：

#### 连接管理
```typescript
// gRPC 客户端连接配置
const client = new ChatServiceClient('localhost:8082', grpc.credentials.createInsecure());

// 双向流连接
const chatStream = client.Chat();
const statusStream = client.UserStatusUpdate();
```

#### 主要交互场景
1. **用户认证**: 通过 `Login` 和 `Register` 方法进行身份验证
2. **实时聊天**: 使用 `Chat` 双向流进行消息收发
3. **状态同步**: 通过 `UserStatusUpdate` 流同步在线状态
4. **数据查询**: 调用各种 Unary 方法获取用户、好友、群组信息

### 🛠️ 客户端安装与运行

#### 环境准备
```bash
# 确保 Node.js 版本 >= 18.x
node --version

# 确保 npm 版本 >= 8.x
npm --version

# 安装 Python 3.x (用于 node-gyp)
python --version
```

#### 依赖安装
```bash
# 进入客户端目录
cd Rinne-IM-Client

# 安装项目依赖
npm install

# 安装 Electron 构建依赖
npm install --save-dev electron-builder
```

#### 开发环境运行
```bash
# 启动开发服务器 (同时运行 React 和 Electron)
npm run dev

# 或分别启动
npm run dev:react    # 启动 React 开发服务器
npm run dev:electron # 启动 Electron 应用
```

#### 生产环境构建
```bash
# 构建生产版本
npm run build

# 打包为可执行文件
npm run build:electron

# 生成各平台安装包
npm run pack
```

### 🎨 界面功能说明

#### 主要界面组件

1. **登录界面**
   - 用户名密码输入
   - 记住登录状态选项
   - 注册新账户链接

2. **主应用界面**
   ```
   ┌─────────────────────────────────────────────────┐
   │  顶部导航栏 (用户信息、设置、退出)               │
   ├─────────────────┬───────────────────────────────┤
   │  侧边栏         │  主聊天区域                   │
   │  ┌───────────┐  │  ┌─────────────────────────┐ │
   │  │ 会话列表   │  │  │ 聊天消息显示区域         │ │
   │  │           │  │  │                         │ │
   │  │           │  │  │                         │ │
   │  └───────────┘  │  ├─────────────────────────┤ │
   │  ┌───────────┐  │  │ 消息输入框               │ │
   │  │ 联系人列表 │  │  └─────────────────────────┘ │
   │  └───────────┘  │                               │
   └─────────────────┴───────────────────────────────┘
   ```

3. **聊天窗口功能**
   - 实时消息显示
   - 消息发送 (Enter 发送，Shift+Enter 换行)
   - 表情符号选择
   - 文件拖拽上传
   - 图片预览功能

4. **设置面板**
   - 个人资料编辑
   - 通知设置
   - 隐私设置
   - 主题切换

### ⚙️ 客户端配置要求

#### 配置文件结构
```
Rinne-IM-Client/
├── src/
│   ├── config/
│   │   ├── app.config.ts      # 应用基本配置
│   │   └── grpc.config.ts     # gRPC 连接配置
│   └── constants/
│       └── api.constants.ts   # API 常量定义
```

#### 主要配置项

**应用配置** (`app.config.ts`):
```typescript
export const AppConfig = {
  // 应用基本信息
  appName: 'Rinne-IM',
  version: '1.0.0',
  
  // 服务端连接配置
  serverHost: 'localhost',
  serverPort: 8082,
  
  // 存储配置
  storagePrefix: 'rinne_im_',
  
  // 界面配置
  theme: 'light', // light | dark
  language: 'zh-CN'
};
```

**gRPC 配置** (`grpc.config.ts`):
```typescript
export const GrpcConfig = {
  // 连接超时设置
  connectionTimeout: 5000,
  keepaliveTime: 30000,
  keepaliveTimeout: 20000,
  
  // 重试策略
  maxRetries: 3,
  retryDelay: 1000,
  
  // 流控设置
  maxReceiveMessageLength: 4 * 1024 * 1024, // 4MB
  maxSendMessageLength: 4 * 1024 * 1024
};
```

#### 环境变量配置
```bash
# .env 文件
REACT_APP_SERVER_HOST=localhost
REACT_APP_SERVER_PORT=8082
REACT_APP_DEBUG_MODE=true
REACT_APP_LOG_LEVEL=debug
```

### 🔧 客户端技术特性

- **响应式设计**: 支持不同屏幕尺寸自适应
- **离线支持**: 基础功能的离线可用性
- **性能优化**: 虚拟滚动、懒加载等优化技术
- **错误处理**: 完善的异常捕获和用户提示
- **国际化**: 多语言支持准备
- **Accessibility**: 无障碍访问支持

### 📱 跨平台支持

- **Windows**: 完整功能支持
- **macOS**: 原生应用体验
- **Linux**: 标准桌面环境适配
- **Web版**: 浏览器访问支持 (计划中)

## 📁 项目结构

```
Rinne-IM/
├── Rinne-IM-Server/           # 服务端代码
│   ├── cmd/
│   │   ├── im_server/         # 主服务入口
│   │   └── client_test/       # 测试客户端
│   ├── config/                # 配置管理
│   ├── internal/
│   │   ├── repository/        # 数据访问层
│   │   └── service/           # 业务逻辑层
│   ├── models/                # 数据模型
│   ├── queue/                 # 消息队列封装
│   ├── service/pb/            # Protobuf 定义
│   ├── sql/migrations/        # 数据库迁移脚本
│   ├── tests/                 # 测试代码
│   ├── Dockerfile             # 服务端 Docker 镜像
│   └── docker-compose.yaml    # 本地开发环境编排
│
├── Rinne-IM-Client/           # 客户端代码
│   ├── src/
│   │   ├── components/        # React 组件
│   │   ├── proto/             # Protobuf 生成代码
│   │   ├── routes/            # 路由配置
│   │   ├── services/          # 服务层
│   │   └── utils/             # 工具函数
│   ├── electron/              # Electron 主进程
│   └── public/                # 静态资源
│
└── README.md                  # 项目文档
```

## 🚀 功能特性

### ✅ 已实现功能

| 功能模块 | 状态 | 说明 |
|---------|------|------|
| 用户注册/登录 | ✅ | JWT Token 认证 |
| 好友管理 | ✅ | 添加好友、好友列表 |
| 群组聊天 | ✅ | 创建群组、邀请成员 |
| 实时消息 | ✅ | 文本、图片、文件传输 |
| 会话管理 | ✅ | 最近会话、历史消息 |
| 在线状态 | ✅ | 用户在线状态同步 |
| 消息已读回执 | ✅ | 消息阅读状态标记 |

### 🔧 技术特性

- **双向流通信**: 支持实时消息推送
- **消息持久化**: 所有消息存储到 PostgreSQL
- **热点缓存**: 用户状态、会话信息缓存到 Redis
- **异步处理**: Kafka 处理消息投递和通知
- **连接管理**: 自动重连、心跳检测
- **安全性**: 密码加密、Token 验证

## 🛠️ 环境要求

### 服务端依赖
- **Golang**: 1.25 或更高版本
- **PostgreSQL**: 15.x 或更高版本
- **Redis**: 7.x 或更高版本
- **Apache Kafka**: 3.5 或更高版本
- **Docker**: 最新版本（推荐用于本地开发）

### 客户端依赖
- **Node.js**: 18.x 或更高版本
- **npm**: 8.x 或更高版本
- **Python**: 3.x（用于 node-gyp 构建原生模块）

## 📦 安装部署

### 1. 克隆项目

```bash
git clone https://github.com/yurin-kami/Rinne-IM.git
cd Rinne-IM
```

### 2. 配置服务端

编辑 `Rinne-IM-Server/config/config.toml`:

```toml
[database]
user = "postgres"
password = "your_password"
host = "localhost"
port = "5432"
database = "Rinne-IM"

[redis]
host = "localhost"
port = "6379"
password = "your_redis_password"

[kafka]
brokers = ["localhost:9094"]

[server]
port = ":8082"

[jwt]
secret = "your-jwt-secret-key"
```

### 3. 启动基础设施

```bash
# 进入服务端目录
cd Rinne-IM-Server

# 启动数据库和消息队列
docker-compose up --build -d

# 等待服务启动完成（约30秒）
```



### 4. 启动客户端

```bash
# 进入客户端目录
cd ../Rinne-IM-Client

# 安装依赖
npm install

# 启动开发服务器
npm run dev:start
```

## 📡 API 接口

### gRPC 服务定义

系统基于 `service/pb/im.proto` 定义了完整的 gRPC 接口：

#### 核心服务方法

| 方法名 | 类型 | 说明 |
|--------|------|------|
| `Chat` | 双向流 | 实时聊天消息传输 |
| `UserStatusUpdate` | 客户端流 | 用户状态更新 |
| `Login` | Unary | 用户登录 |
| `Register` | Unary | 用户注册 |
| `SearchUser` | Unary | 搜索用户 |
| `AddFriend` | Unary | 添加好友 |
| `GetFriendList` | Unary | 获取好友列表 |
| `CreateGroup` | Unary | 创建群组 |
| `InviteToGroup` | Unary | 邀请入群 |
| `CreateSession` | Unary | 创建会话 |
| `GetRecentSessions` | Unary | 获取最近会话 |
| `GetHistory` | Unary | 获取历史消息 |
| `MarkMessageAsRead` | Unary | 标记消息已读 |

### 示例请求

```protobuf
// 用户登录
message LoginRequest {
  string username = 1;
  string password = 2;
  string device_id = 3;
  string device_type = 4;
}

// 聊天消息
message ChatMessage {
  int64 msg_id = 1;
  int64 sender_id = 2;
  int64 receiver_id = 3;
  string content = 4;
  int64 timestamp = 5;
  int32 msg_type = 6; // 0:text, 1:image, 2:audio, 3:video, 4:file
  string media_url = 7;
  bool is_group_msg = 8;
  string group_id = 9;
}
```

## 🗄️ 数据库设计

### 核心表结构

#### 用户表 (users)
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    avatar_url TEXT,
    status INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 会话表 (sessions)
```sql
CREATE TABLE sessions (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    target_id BIGINT, -- 对方用户ID或群组ID
    is_group BOOLEAN DEFAULT FALSE,
    last_message_id BIGINT,
    last_message_time TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 消息表 (messages)
```sql
CREATE TABLE messages (
    id BIGSERIAL PRIMARY KEY,
    session_id BIGINT REFERENCES sessions(id),
    sender_id BIGINT REFERENCES users(id),
    content TEXT,
    msg_type INTEGER DEFAULT 0,
    media_url TEXT,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 好友关系表 (friends)
```sql
CREATE TABLE friends (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    friend_id BIGINT REFERENCES users(id),
    status INTEGER DEFAULT 0, -- 0:pending, 1:accepted, 2:blocked
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 群组表 (groups)
```sql
CREATE TABLE groups (
    id BIGSERIAL PRIMARY KEY,
    group_name VARCHAR(100) NOT NULL,
    owner_id BIGINT REFERENCES users(id),
    avatar_url TEXT,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 💻 使用说明

### 用户操作流程

1. **注册账户**
   - 填写用户名、密码等基本信息
   - 系统验证用户名唯一性
   - 注册成功后自动登录

2. **添加好友**
   - 通过搜索功能查找用户
   - 发送好友申请
   - 对方接受后建立好友关系

3. **创建群组**
   - 选择群组名称和初始成员
   - 邀请更多用户加入群组
   - 群主可以管理群成员

4. **发送消息**
   - 选择好友或群组开始对话
   - 支持文本、图片、文件等多种消息类型
   - 实时接收对方回复

### 客户端界面

- **登录页面**: 用户认证入口
- **主界面**: 包含联系人列表、聊天窗口、设置面板
- **聊天窗口**: 支持表情、图片、文件发送
- **设置页面**: 个人资料、隐私设置、通知配置

## 📝 开发规范

### 代码风格

**Go 代码规范**:
- 遵循官方 [Effective Go](https://golang.org/doc/effective_go.html)
- 使用 `gofmt` 格式化代码
- 函数和变量命名采用驼峰命名法
- 包名使用小写字母和下划线

**TypeScript 代码规范**:
- 使用 ESLint 和 Prettier
- 严格模式 (`strict: true`)
- 接口命名使用 PascalCase
- 组件文件使用 `.tsx` 扩展名

## 🤝 贡献指南

欢迎参与 Rinne-IM 项目的开发！

### 贡献流程

1. Fork 项目到你的 GitHub 账户
2. 创建功能分支: `git checkout -b feature/your-feature-name`
3. 提交更改: `git commit -am 'feat: add some feature'`
4. 推送到分支: `git push origin feature/your-feature-name`
5. 创建 Pull Request

### 开发环境设置

```bash
# 1. 克隆仓库
git clone https://github.com/your-username/Rinne-IM.git

# 2. 安装依赖
cd Rinne-IM-Server && go mod tidy
cd ../Rinne-IM-Client && npm install

# 3. 启动开发环境
# 终端1: 启动基础设施
cd Rinne-IM-Server && docker-compose up -d

# 终端2: 启动服务端
cd Rinne-IM-Server && go run cmd/im_server/main.go

# 终端3: 启动客户端
cd Rinne-IM-Client && npm run dev
```

### 代码质量要求

- 所有新功能必须包含单元测试
- 提交前运行测试: `go test ./...` 和 `npm test`
- 遵循现有的代码风格和架构模式
- 更新相关文档

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

```
MIT License

Copyright (c) 2026 Rinne-IM Project

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 🆘 支持与反馈

- **GitHub Issues**: [提交问题](https://github.com/your-username/Rinne-IM/issues)
- **邮件**: support@rinne-im.com
- **QQ群**: 123456789
- **微信群**: 扫描二维码加入

---

<p align="center">
  Made with ❤️ by the Rinne-IM Team
</p>