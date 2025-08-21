# Monibuca 微服务项目目录结构

## 项目结构

```
monibuca/
├── cmd/                     # 主程序入口
│   ├── gateway/            # Media Gateway 服务
│   │   └── main.go         # 网关服务主程序
│   ├── server/             # Media Server 服务
│   │   └── main.go         # 媒体服务器主程序
│   └── admin/              # 管理服务
│       └── main.go         # 管理服务主程序
├── internal/                # 内部包
│   ├── core/               # 核心组件
│   │   ├── stream/         # 流管理
│   │   │   ├── manager.go  # 流管理器
│   │   │   ├── publisher.go # 推流器
│   │   │   ├── subscriber.go # 拉流器
│   │   │   └── buffer.go   # 流缓冲区
│   │   ├── media/          # 媒体处理
│   │   │   ├── codec/      # 编解码器
│   │   │   │   ├── h264.go # H.264 编解码
│   │   │   │   ├── h265.go # H.265 编解码
│   │   │   │   ├── aac.go  # AAC 编解码
│   │   │   │   └── g711.go # G.711 编解码
│   │   │   ├── filter/     # 媒体滤镜
│   │   │   │   ├── resize.go # 尺寸调整
│   │   │   │   ├── crop.go # 裁剪滤镜
│   │   │   │   └── overlay.go # 叠加滤镜
│   │   │   └── transform/  # 媒体转换
│   │   │       ├── format.go # 格式转换
│   │   │       ├── bitrate.go # 码率转换
│   │   │       └── fps.go  # 帧率转换
│   │   ├── auth/           # 认证授权
│   │   │   ├── jwt.go      # JWT 认证
│   │   │   ├── rbac.go     # 权限控制
│   │   │   └── middleware.go # 认证中间件
│   │   └── monitor/        # 监控组件
│   │       ├── metrics.go  # 指标收集
│   │       ├── health.go   # 健康检查
│   │       └── alert.go    # 告警服务
│   ├── gateway/             # 网关逻辑
│   │   ├── handler/        # 协议处理器
│   │   │   ├── rtmp.go     # RTMP 处理器
│   │   │   ├── rtsp.go     # RTSP 处理器
│   │   │   ├── webrtc.go   # WebRTC 处理器
│   │   │   ├── hls.go      # HLS 处理器
│   │   │   └── http.go     # HTTP 处理器
│   │   ├── router/         # 路由管理
│   │   │   ├── router.go   # 主路由器
│   │   │   ├── middleware.go # 路由中间件
│   │   │   └── rate_limit.go # 限流中间件
│   │   ├── balancer/       # 负载均衡
│   │   │   ├── round_robin.go # 轮询算法
│   │   │   ├── least_conn.go # 最少连接
│   │   │   ├── weighted.go # 加权算法
│   │   │   └── consistent_hash.go # 一致性哈希
│   │   └── discovery/      # 服务发现
│   │       ├── consul.go   # Consul 服务发现
│   │       ├── health.go   # 健康检查
│   │       └── registry.go # 服务注册
│   ├── server/              # 媒体服务器逻辑
│   │   ├── engine/         # 媒体引擎
│   │   │   ├── core.go     # 引擎核心
│   │   │   ├── pipeline.go # 处理管道
│   │   │   └── worker.go   # 工作线程
│   │   ├── cluster/        # 集群管理
│   │   │   ├── node.go     # 节点管理
│   │   │   ├── sync.go     # 数据同步
│   │   │   ├── election.go # 主从选举
│   │   │   └── communication.go # 节点通信
│   │   └── storage/        # 存储管理
│   │       ├── record/     # 录制管理
│   │       │   ├── manager.go # 录制管理器
│   │       │   ├── mp4.go  # MP4 录制
│   │       │   ├── hls.go  # HLS 录制
│   │       │   └── flv.go  # FLV 录制
│   │       ├── snapshot/   # 截图管理
│   │       │   ├── manager.go # 截图管理器
│   │       │   └── jpeg.go # JPEG 截图
│   │       └── archive/    # 归档管理
│   │           ├── manager.go # 归档管理器
│   │           └── cleanup.go # 清理策略
│   ├── models/              # 数据模型
│   │   ├── stream.go       # 流模型
│   │   ├── server.go       # 服务器模型
│   │   ├── user.go         # 用户模型
│   │   ├── config.go       # 配置模型
│   │   ├── record.go       # 录制模型
│   │   └── stats.go        # 统计模型
│   └── services/            # 业务服务
│       ├── stream_service.go # 流服务
│       ├── auth_service.go # 认证服务
│       ├── monitor_service.go # 监控服务
│       ├── record_service.go # 录制服务
│       └── stats_service.go # 统计服务
├── pkg/                     # 公共包
│   ├── config/              # 配置管理
│   │   ├── consul.go       # Consul 配置中心
│   │   ├── database.go     # 数据库配置
│   │   ├── cache.go        # 缓存配置
│   │   ├── storage.go      # 存储配置
│   │   └── queue.go        # 队列配置
│   ├── database/            # 数据库操作
│   │   ├── postgres.go     # PostgreSQL 连接
│   │   ├── gorm.go         # GORM 配置
│   │   ├── migration.go    # 数据库迁移
│   │   ├── connection.go   # 连接池管理
│   │   └── transaction.go  # 事务管理
│   ├── cache/               # 缓存操作
│   │   ├── redis.go        # Redis 客户端
│   │   ├── memory.go       # 内存缓存
│   │   ├── strategy.go     # 缓存策略
│   │   ├── lru.go          # LRU 缓存
│   │   └── ttl.go          # TTL 管理
│   ├── storage/             # 存储操作
│   │   ├── minio.go        # MinIO 客户端
│   │   ├── local.go        # 本地存储
│   │   ├── backup.go       # 备份管理
│   │   ├── sync.go         # 同步管理
│   │   └── compression.go  # 压缩管理
│   ├── queue/               # 消息队列
│   │   ├── rabbitmq.go     # RabbitMQ 客户端
│   │   ├── producer.go     # 消息生产者
│   │   ├── consumer.go     # 消息消费者
│   │   ├── exchange.go     # 交换机管理
│   │   └── binding.go      # 绑定管理
│   └── utils/               # 工具函数
│       ├── logger.go        # 日志工具
│       ├── crypto.go        # 加密工具
│       ├── time.go          # 时间工具
│       ├── http.go          # HTTP 工具
│       ├── json.go          # JSON 工具
│       └── validator.go     # 数据验证
├── api/                     # API 定义
│   ├── proto/               # gRPC 协议
│   │   ├── stream.proto    # 流服务协议
│   │   ├── auth.proto      # 认证服务协议
│   │   ├── monitor.proto   # 监控服务协议
│   │   ├── record.proto    # 录制服务协议
│   │   └── stats.proto     # 统计服务协议
│   ├── http/                # HTTP 路由
│   │   ├── routes.go       # 路由定义
│   │   ├── middleware.go   # 中间件
│   │   ├── handlers.go     # 处理器
│   │   ├── cors.go         # 跨域处理
│   │   └── rate_limit.go   # 限流处理
│   └── websocket/           # WebSocket 处理
│       ├── connection.go    # 连接管理
│       ├── message.go       # 消息处理
│       ├── hub.go           # 连接中心
│       ├── broadcast.go     # 广播管理
│       └── auth.go          # 认证管理
├── configs/                  # 配置文件
│   ├── consul/              # Consul 配置
│   │   ├── gateway.json    # 网关配置
│   │   ├── server.json     # 服务器配置
│   │   ├── admin.json      # 管理服务配置
│   │   └── common.json     # 通用配置
│   ├── database/            # 数据库配置
│   │   ├── postgres.yaml   # PostgreSQL 配置
│   │   ├── migration.yaml  # 迁移配置
│   │   └── backup.yaml     # 备份配置
│   ├── redis/               # Redis 配置
│   │   ├── redis.conf      # Redis 配置
│   │   └── cluster.conf    # 集群配置
│   ├── minio/               # MinIO 配置
│   │   ├── minio.yaml      # MinIO 配置
│   │   └── policy.yaml     # 存储策略
│   └── rabbitmq/            # RabbitMQ 配置
│       ├── rabbitmq.conf   # RabbitMQ 配置
│       ├── policy.conf     # 策略配置
│       └── users.conf      # 用户配置
├── deploy/                   # 部署配置
│   ├── docker/              # Docker 配置
│   │   ├── gateway.Dockerfile # 网关镜像
│   │   ├── server.Dockerfile  # 服务器镜像
│   │   ├── admin.Dockerfile   # 管理服务镜像
│   │   ├── docker-compose.yml # 编排文件
│   │   └── .dockerignore     # 忽略文件
│   ├── k8s/                  # Kubernetes 配置
│   │   ├── gateway.yaml     # 网关部署
│   │   ├── server.yaml      # 服务器部署
│   │   ├── admin.yaml       # 管理服务部署
│   │   ├── services.yaml    # 服务配置
│   │   ├── configmaps.yaml  # 配置映射
│   │   ├── secrets.yaml     # 密钥配置
│   │   └── ingress.yaml     # 入口配置
│   └── terraform/            # Terraform 配置
│       ├── main.tf          # 主配置
│       ├── variables.tf     # 变量定义
│       ├── outputs.tf       # 输出定义
│       ├── modules/         # 模块配置
│       │   ├── vpc/         # VPC 模块
│       │   ├── ecs/         # ECS 模块
│       │   └── rds/         # RDS 模块
│       └── environments/    # 环境配置
│           ├── dev/         # 开发环境
│           ├── test/         # 测试环境
│           └── prod/         # 生产环境
├── scripts/                  # 脚本文件
│   ├── build/               # 构建脚本
│   │   ├── build.sh        # 构建脚本
│   │   ├── test.sh         # 测试脚本
│   │   └── lint.sh         # 代码检查
│   ├── deploy/              # 部署脚本
│   │   ├── deploy.sh       # 部署脚本
│   │   ├── rollback.sh     # 回滚脚本
│   │   └── upgrade.sh      # 升级脚本
│   └── maintenance/         # 维护脚本
│       ├── backup.sh       # 备份脚本
│       ├── cleanup.sh      # 清理脚本
│       └── monitor.sh      # 监控脚本
├── docs/                     # 文档
│   ├── api/                 # API 文档
│   ├── deployment/          # 部署文档
│   ├── development/         # 开发文档
│   └── architecture/        # 架构文档
├── go.mod                    # Go 模块
├── go.sum                    # Go 依赖
├── Makefile                  # 构建脚本
├── .gitignore               # Git 忽略文件
├── README.md                 # 项目说明
└── LICENSE                   # 许可证
```

## 功能说明

### 1. cmd/ - 服务入口
- **gateway**: Media Gateway 服务启动，负责协议处理、路由分发、负载均衡
- **server**: Media Server 服务启动，负责媒体处理、编解码、存储管理
- **admin**: 管理服务启动，负责系统管理、监控、配置管理

### 2. internal/ - 核心逻辑
- **core/stream**: 流生命周期管理、推拉流控制、缓冲区管理
- **core/media**: 音视频编解码、媒体滤镜、格式转换
- **core/auth**: JWT 认证、RBAC 权限控制、中间件
- **core/monitor**: 性能指标收集、健康检查、告警服务
- **gateway/handler**: 多协议处理器（RTMP、RTSP、WebRTC、HLS、HTTP）
- **gateway/router**: 智能路由、中间件链、限流控制
- **gateway/balancer**: 多种负载均衡算法、健康检查、服务发现
- **gateway/discovery**: Consul 服务注册发现、健康状态管理
- **server/engine**: 媒体处理引擎、工作线程池、处理管道
- **server/cluster**: 集群节点管理、数据同步、主从选举
- **server/storage**: 录制管理、截图服务、归档清理

### 3. pkg/ - 公共组件
- **config**: 统一配置管理、Consul 集成、环境变量处理
- **database**: PostgreSQL 连接池、GORM 配置、事务管理、迁移工具
- **cache**: Redis 集群、内存缓存、LRU 策略、TTL 管理
- **storage**: MinIO 对象存储、本地存储、备份同步、压缩管理
- **queue**: RabbitMQ 消息队列、生产者消费者、交换机绑定
- **utils**: 日志系统、加密工具、时间处理、HTTP 工具、数据验证

### 4. api/ - 接口定义
- **proto**: gRPC 服务定义、流式接口、认证接口、监控接口
- **http**: RESTful API、中间件链、CORS 处理、限流控制
- **websocket**: 实时通信、连接管理、消息广播、认证管理

### 5. configs/ - 配置文件
- **consul**: 各服务配置、通用配置、环境配置
- **database**: PostgreSQL 配置、迁移配置、备份策略
- **redis**: Redis 配置、集群配置、性能调优
- **minio**: MinIO 配置、存储策略、访问控制
- **rabbitmq**: RabbitMQ 配置、策略配置、用户管理

### 6. deploy/ - 部署配置
- **docker**: 多服务镜像、编排文件、环境隔离
- **k8s**: 部署配置、服务配置、配置映射、密钥管理、入口配置
- **terraform**: 基础设施即代码、多环境支持、模块化配置

### 7. scripts/ - 脚本工具
- **build**: 自动化构建、测试执行、代码质量检查
- **deploy**: 一键部署、回滚机制、升级流程
- **maintenance**: 数据备份、系统清理、监控维护

### 8. docs/ - 文档管理
- **api**: API 接口文档、使用示例、错误码说明
- **deployment**: 部署指南、环境配置、故障排查
- **development**: 开发指南、代码规范、贡献指南
- **architecture**: 架构设计、技术选型、性能优化

## 服务启动

```bash
# 启动网关服务
go run cmd/gateway/main.go

# 启动媒体服务器
go run cmd/server/main.go

# 启动管理服务
go run cmd/admin/main.go
```

## 目录结构特点

### 1. **分层架构设计**
- **cmd/**: 服务入口层，每个服务独立启动
- **internal/**: 核心业务逻辑层，不对外暴露
- **pkg/**: 公共组件层，可复用和扩展
- **api/**: 接口定义层，统一对外接口
- **configs/**: 配置管理层，集中配置管理
- **deploy/**: 部署配置层，支持多种部署方式

### 2. **微服务化组织**
- **服务独立**: 每个服务有独立的启动入口和配置
- **职责分离**: 网关、媒体服务器、管理服务职责明确
- **接口统一**: 通过 API 层统一对外接口
- **配置分离**: 各服务配置独立管理

### 3. **模块化设计**
- **核心模块**: 流管理、媒体处理、认证、监控
- **网关模块**: 协议处理、路由、负载均衡、服务发现
- **服务器模块**: 媒体引擎、集群管理、存储管理
- **公共模块**: 数据库、缓存、存储、消息队列、工具

### 4. **技术栈集成**
- **配置中心**: Consul 统一配置管理
- **数据库**: PostgreSQL + GORM ORM
- **缓存**: Redis 分布式缓存
- **存储**: MinIO 对象存储
- **消息队列**: RabbitMQ 异步通信
- **容器化**: Docker + Kubernetes 部署

### 5. **开发友好**
- **标准规范**: 符合 Go 项目标准目录结构
- **清晰命名**: 目录和文件命名语义化
- **文档完整**: 包含开发、部署、架构文档
- **脚本支持**: 构建、部署、维护脚本

## 总结

这个详细的目录结构具有以下优势：

1. **符合Go规范**: 遵循Go项目标准目录结构和命名约定
2. **微服务友好**: 支持独立部署、扩展和维护
3. **模块化清晰**: 职责分离明确，便于团队协作开发
4. **配置集中化**: 统一配置管理，支持环境隔离
5. **部署灵活**: 支持Docker、Kubernetes、Terraform等多种部署方式
6. **扩展性强**: 模块化设计便于功能扩展和定制
7. **维护性好**: 清晰的目录结构便于代码维护和问题排查

这种架构设计为 Monibuca 的微服务化改造提供了完整的项目组织框架，使其能够更好地应对大规模、高并发的流媒体服务需求。
