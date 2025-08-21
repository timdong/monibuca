# Monibuca 系统改进架构设计

## 概述

本文档描述了 Monibuca 流媒体服务器的系统改进架构设计，旨在通过引入现代化的技术栈和架构模式，提升系统的可扩展性、可维护性和性能。主要改进包括：

- 使用 Consul 作为统一配置中心和服务发现
- 采用 GORM 作为 ORM 框架，支持多种数据库
- 使用 Gin 作为 Web 服务框架
- 集成 Redis 作为缓存和会话存储
- 使用 MinIO 作为分布式文件存储
- 集成 RabbitMQ 作为消息队列
- 重构推拉流代理架构，实现 Media Gateway 模式
- 建立环形 Media Server 集群架构

## 技术栈升级

### 原有技术栈
- 配置管理：YAML 文件 + 内存配置
- 数据库：原生 SQL + 简单 ORM
- Web 服务：标准库 net/http
- 缓存：内存缓存
- 文件存储：本地文件系统
- 消息队列：无
- 服务发现：无

### 新技术栈
- 配置管理：Consul KV Store + 配置热更新
- 数据库：GORM + 多数据库支持
- Web 服务：Gin + 中间件架构
- 缓存：Redis + 分布式缓存
- 文件存储：MinIO + 对象存储
- 消息队列：RabbitMQ + 异步处理
- 服务发现：Consul + 健康检查

## 系统架构设计

### 整体架构图

```mermaid
graph TB
    subgraph "客户端层"
        RTMP_Client[RTMP 客户端]
        RTSP_Client[RTSP 客户端]
        WebRTC_Client[WebRTC 客户端]
        HTTP_Client[HTTP 客户端]
    end
    
    subgraph "负载均衡层"
        LB[负载均衡器/API网关]
    end
    
    subgraph "Media Gateway 层"
        MG1[Media Gateway 1]
        MG2[Media Gateway 2]
        MG3[Media Gateway 3]
    end
    
    subgraph "Media Server 集群"
        MS1[Media Server 1]
        MS2[Media Server 2]
        MS3[Media Server 3]
        MS4[Media Server 4]
    end
    
    subgraph "管理服务层"
        Admin[管理服务]
        Monitor[监控服务]
        Config[配置服务]
    end
    
    subgraph "基础设施层"
        Consul[Consul]
        Redis[(Redis)]
        MinIO[(MinIO)]
        RabbitMQ[(RabbitMQ)]
        DB[(数据库)]
    end
    
    %% 客户端到网关
    RTMP_Client --> LB
    RTSP_Client --> LB
    WebRTC_Client --> LB
    HTTP_Client --> LB
    
    %% 负载均衡到网关
    LB --> MG1
    LB --> MG2
    LB --> MG3
    
    %% 网关到媒体服务器
    MG1 --> MS1
    MG1 --> MS2
    MG2 --> MS2
    MG2 --> MS3
    MG3 --> MS3
    MG3 --> MS4
    
    %% 环形连接
    MS1 -.-> MS2
    MS2 -.-> MS3
    MS3 -.-> MS4
    MS4 -.-> MS1
    
    %% 管理服务连接
    Admin --> Consul
    Admin --> Redis
    Admin --> MinIO
    Admin --> RabbitMQ
    Admin --> DB
    
    %% 媒体服务器连接基础设施
    MS1 --> Consul
    MS1 --> Redis
    MS1 --> MinIO
    MS1 --> RabbitMQ
    MS1 --> DB
    
    MS2 --> Consul
    MS2 --> Redis
    MS2 --> MinIO
    MS2 --> RabbitMQ
    MS2 --> DB
    
    MS3 --> Consul
    MS3 --> Redis
    MS3 --> MinIO
    MS3 --> RabbitMQ
    MS3 --> DB
    
    MS4 --> Consul
    MS4 --> Redis
    MS4 --> MinIO
    MS4 --> RabbitMQ
    MS4 --> DB
```

### Media Gateway 架构

```mermaid
graph TB
    subgraph "Media Gateway"
        subgraph "协议适配层"
            RTMP_Handler[RTMP 处理器]
            RTSP_Handler[RTSP 处理器]
            WebRTC_Handler[WebRTC 处理器]
            HTTP_Handler[HTTP 处理器]
        end
        
        subgraph "流管理核心"
            Stream_Router[流路由]
            Load_Balancer[负载均衡器]
            Health_Checker[健康检查]
        end
        
        subgraph "缓存层"
            Stream_Cache[流缓存]
            Session_Cache[会话缓存]
            Config_Cache[配置缓存]
        end
        
        subgraph "消息队列"
            Stream_Events[流事件]
            Health_Events[健康事件]
            Config_Events[配置事件]
        end
    end
    
    subgraph "外部接口"
        Client[客户端]
        Admin[管理接口]
        Monitor[监控接口]
    end
    
    Client --> RTMP_Handler
    Client --> RTSP_Handler
    Client --> WebRTC_Handler
    Client --> HTTP_Handler
    
    RTMP_Handler --> Stream_Router
    RTSP_Handler --> Stream_Router
    WebRTC_Handler --> Stream_Router
    HTTP_Handler --> Stream_Router
    
    Stream_Router --> Load_Balancer
    Load_Balancer --> Health_Checker
    
    Stream_Router --> Stream_Cache
    Stream_Router --> Session_Cache
    Stream_Router --> Config_Cache
    
    Stream_Router --> Stream_Events
    Health_Checker --> Health_Events
    Config_Cache --> Config_Events
    
    Admin --> Stream_Router
    Monitor --> Health_Checker
```

### Media Server 集群架构

```mermaid
graph TB
    subgraph "Media Server 集群"
        subgraph "节点 1"
            MS1_Core[核心服务]
            MS1_Stream[流管理]
            MS1_Process[媒体处理]
        end
        
        subgraph "节点 2"
            MS2_Core[核心服务]
            MS2_Stream[流管理]
            MS2_Process[媒体处理]
        end
        
        subgraph "节点 3"
            MS3_Core[核心服务]
            MS3_Stream[流管理]
            MS3_Process[媒体处理]
        end
        
        subgraph "节点 4"
            MS4_Core[核心服务]
            MS4_Stream[流管理]
            MS4_Process[媒体处理]
        end
    end
    
    subgraph "共享资源"
        Shared_Config[配置中心]
        Shared_Cache[分布式缓存]
        Shared_Storage[对象存储]
        Shared_Queue[消息队列]
    end
    
    %% 环形连接
    MS1_Core -.-> MS2_Core
    MS2_Core -.-> MS3_Core
    MS3_Core -.-> MS4_Core
    MS4_Core -.-> MS1_Core
    
    %% 共享资源连接
    MS1_Core --> Shared_Config
    MS1_Core --> Shared_Cache
    MS1_Core --> Shared_Storage
    MS1_Core --> Shared_Queue
    
    MS2_Core --> Shared_Config
    MS2_Core --> Shared_Cache
    MS2_Core --> Shared_Storage
    MS2_Core --> Shared_Queue
    
    MS3_Core --> Shared_Config
    MS3_Core --> Shared_Cache
    MS3_Core --> Shared_Storage
    MS3_Core --> Shared_Queue
    
    MS4_Core --> Shared_Config
    MS4_Core --> Shared_Cache
    MS4_Core --> Shared_Storage
    MS4_Core --> Shared_Queue
```

## 核心组件设计

### 1. Consul 配置中心

#### 配置结构
```yaml
# 全局配置
global:
  consul:
    address: "localhost:8500"
    datacenter: "dc1"
    token: ""
  
  # 媒体服务配置
  media:
    gateway:
      port: 8080
      workers: 4
    server:
      port: 8081
      workers: 8
    
     # 数据库配置
   database:
     driver: "postgres"
     dsn: "host=localhost user=postgres password=postgres dbname=monibuca port=5432 sslmode=disable TimeZone=Asia/Shanghai"
     max_open_conns: 100
     max_idle_conns: 10
     conn_max_lifetime: "1h"
    
  # Redis 配置
  redis:
    address: "localhost:6379"
    password: ""
    db: 0
    
  # MinIO 配置
  minio:
    endpoint: "localhost:9000"
    access_key: "minioadmin"
    secret_key: "minioadmin"
    bucket: "media"
    
  # RabbitMQ 配置
  rabbitmq:
    url: "amqp://guest:guest@localhost:5672/"
    exchange: "media.events"
```

#### 配置热更新机制
```go
type ConfigManager struct {
    consulClient *consul.Client
    watchers     map[string]*consul.Watcher
    callbacks    map[string][]ConfigChangeCallback
}

func (cm *ConfigManager) WatchConfig(key string, callback ConfigChangeCallback) {
    // 监听 Consul KV 变化
    // 自动更新本地配置
    // 触发回调函数
}
```

### 2. GORM 数据模型 (PostgreSQL)

#### 核心数据模型
```go
// 流信息
type Stream struct {
    gorm.Model
    StreamID      string    `gorm:"uniqueIndex;not null"`
    StreamPath    string    `gorm:"index;not null"`
    PublisherID   string    `gorm:"index"`
    Status        string    `gorm:"default:'active'"`
    StartTime     time.Time
    EndTime       *time.Time
    MediaServerID string    `gorm:"index"`
    Metadata      JSON      `gorm:"type:jsonb"`
    Tags          []string  `gorm:"type:text[]"`
    CreatedBy     string    `gorm:"index"`
}

// 媒体服务器节点
type MediaServer struct {
    gorm.Model
    ServerID      string    `gorm:"uniqueIndex;not null"`
    Host          string    `gorm:"not null"`
    Port          int       `gorm:"not null"`
    Status        string    `gorm:"default:'online'"`
    Load          float64   `gorm:"default:0"`
    Capacity      int       `gorm:"default:1000"`
    LastHeartbeat time.Time
    Region        string    `gorm:"index"`
    Datacenter    string    `gorm:"index"`
    Tags          []string  `gorm:"type:text[]"`
}

// 推流配置
type PushConfig struct {
    gorm.Model
    Name          string    `gorm:"not null"`
    StreamPath    string    `gorm:"index;not null"`
    TargetURL     string    `gorm:"not null"`
    Protocol      string    `gorm:"not null"`
    Status        string    `gorm:"default:'active'"`
    MediaServerID string    `gorm:"index"`
}

// 拉流配置
type PullConfig struct {
    gorm.Model
    Name          string    `gorm:"not null"`
    StreamPath    string    `gorm:"index;not null"`
    SourceURL     string    `gorm:"not null"`
    Protocol      string    `gorm:"not null"`
    Status        string    `gorm:"default:'active'"`
    MediaServerID string    `gorm:"index"`
}
```

### 3. Gin Web 服务

#### 路由结构
```go
func SetupRoutes(r *gin.Engine) {
    // API v1
    v1 := r.Group("/api/v1")
    {
        // 流管理
        streams := v1.Group("/streams")
        {
            streams.GET("", ListStreams)
            streams.GET("/:id", GetStream)
            streams.POST("", CreateStream)
            streams.PUT("/:id", UpdateStream)
            streams.DELETE("/:id", DeleteStream)
        }
        
        // 媒体服务器管理
        servers := v1.Group("/servers")
        {
            servers.GET("", ListServers)
            servers.GET("/:id", GetServer)
            servers.POST("", RegisterServer)
            servers.PUT("/:id", UpdateServer)
            servers.DELETE("/:id", DeregisterServer)
        }
        
        // 推流管理
        pushes := v1.Group("/pushes")
        {
            pushes.GET("", ListPushes)
            pushes.POST("", CreatePush)
            pushes.PUT("/:id", UpdatePush)
            pushes.DELETE("/:id", DeletePush)
        }
        
        // 拉流管理
        pulls := v1.Group("/pulls")
        {
            pulls.GET("", ListPulls)
            pulls.POST("", CreatePull)
            pulls.PUT("/:id", UpdatePull)
            pulls.DELETE("/:id", DeletePull)
        }
    }
    
    // WebSocket 接口
    ws := r.Group("/ws")
    {
        ws.GET("/stream/:id", StreamWebSocket)
        ws.GET("/events", EventsWebSocket)
    }
}
```

### 4. Redis 缓存策略

#### 缓存结构
```go
type CacheManager struct {
    redisClient *redis.Client
}

// 流信息缓存
func (cm *CacheManager) CacheStream(stream *Stream) error {
    key := fmt.Sprintf("stream:%s", stream.StreamID)
    return cm.redisClient.Set(context.Background(), key, stream, time.Hour).Err()
}

// 会话缓存
func (cm *CacheManager) CacheSession(sessionID string, data interface{}) error {
    key := fmt.Sprintf("session:%s", sessionID)
    return cm.redisClient.Set(context.Background(), key, data, time.Minute*30).Err()
}

// 配置缓存
func (cm *CacheManager) CacheConfig(key string, value interface{}) error {
    return cm.redisClient.Set(context.Background(), key, value, time.Hour*24).Err()
}
```

### 5. MinIO 文件存储

#### 存储结构
```go
type StorageManager struct {
    minioClient *minio.Client
    bucketName  string
}

// 录制文件存储
func (sm *StorageManager) StoreRecording(streamID, filename string, data []byte) error {
    objectName := fmt.Sprintf("recordings/%s/%s", streamID, filename)
    return sm.minioClient.PutObject(context.Background(), sm.bucketName, objectName, bytes.NewReader(data), int64(len(data)), minio.PutObjectOptions{})
}

// 截图存储
func (sm *StorageManager) StoreScreenshot(streamID, filename string, data []byte) error {
    objectName := fmt.Sprintf("screenshots/%s/%s", streamID, filename)
    return sm.minioClient.PutObject(context.Background(), sm.bucketName, objectName, bytes.NewReader(data), int64(len(data)), minio.PutObjectOptions{})
}
```

### 6. RabbitMQ 消息队列

#### 消息结构
```go
type MessageQueue struct {
    amqpConn    *amqp.Connection
    amqpChannel *amqp.Channel
    exchange    string
}

// 流事件消息
type StreamEvent struct {
    EventType   string    `json:"event_type"`
    StreamID    string    `json:"stream_id"`
    Timestamp   time.Time `json:"timestamp"`
    MediaServer string    `json:"media_server"`
    Data        interface{} `json:"data"`
}

// 健康检查消息
type HealthEvent struct {
    ServerID    string    `json:"server_id"`
    Status      string    `json:"status"`
    Load        float64   `json:"load"`
    Timestamp   time.Time `json:"timestamp"`
}
```

## 主要时序图

### 1. 推流流程

```mermaid
sequenceDiagram
    participant Client as 推流客户端
    participant Gateway as Media Gateway
    participant Consul as Consul
    participant Server as Media Server
    participant Redis as Redis
    participant DB as 数据库
    participant MQ as RabbitMQ
    
    Client->>Gateway: 发起推流请求
    Gateway->>Consul: 查询可用 Media Server
    Consul-->>Gateway: 返回服务器列表
    Gateway->>Gateway: 负载均衡选择服务器
    Gateway->>Server: 转发推流请求
    Server->>Redis: 缓存流信息
    Server->>DB: 保存流记录
    Server->>MQ: 发布流创建事件
    Server-->>Gateway: 推流成功响应
    Gateway-->>Client: 返回推流地址
```

### 2. 拉流流程

```mermaid
sequenceDiagram
    participant Client as 拉流客户端
    participant Gateway as Media Gateway
    participant Redis as Redis
    participant Server as Media Server
    participant DB as 数据库
    participant MQ as RabbitMQ
    
    Client->>Gateway: 发起拉流请求
    Gateway->>Redis: 查询流信息
    alt 流在缓存中
        Redis-->>Gateway: 返回流信息
    else 流不在缓存中
        Gateway->>DB: 查询流记录
        DB-->>Gateway: 返回流信息
        Gateway->>Redis: 缓存流信息
    end
    
    Gateway->>Server: 获取流数据
    Server-->>Gateway: 返回流数据
    Gateway->>MQ: 发布拉流事件
    Gateway-->>Client: 返回流数据
```

### 3. 服务注册与发现

```mermaid
sequenceDiagram
    participant Server as Media Server
    participant Consul as Consul
    participant Gateway as Media Gateway
    participant Redis as Redis
    
    Server->>Consul: 注册服务
    Consul->>Consul: 健康检查
    Consul->>Gateway: 服务变更通知
    Gateway->>Redis: 更新服务缓存
    Gateway->>Gateway: 更新路由表
    
    loop 定期健康检查
        Server->>Consul: 心跳更新
        Consul->>Gateway: 状态变更通知
        Gateway->>Redis: 更新状态缓存
    end
```

### 4. 配置热更新

```mermaid
sequenceDiagram
    participant Admin as 管理界面
    participant Consul as Consul
    participant Gateway as Media Gateway
    participant Server as Media Server
    participant Redis as Redis
    
    Admin->>Consul: 更新配置
    Consul->>Gateway: 配置变更通知
    Gateway->>Redis: 更新配置缓存
    Gateway->>Gateway: 应用新配置
    
    Consul->>Server: 配置变更通知
    Server->>Redis: 更新配置缓存
    Server->>Server: 应用新配置
```

## 部署架构

### 容器化部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    volumes:
      - ./consul:/consul/data
    command: consul agent -server -bootstrap-expect=1 -ui -client=0.0.0.0
    
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - ./redis:/data
      
  minio:
    image: minio/minio:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - ./minio:/data
    command: server /data --console-address ":9001"
    
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
      
     postgres:
     image: postgres:15-alpine
     ports:
       - "5432:5432"
     environment:
       POSTGRES_PASSWORD: postgres
       POSTGRES_USER: postgres
       POSTGRES_DB: monibuca
       POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --lc-collate=C --lc-ctype=C"
     volumes:
       - ./postgres:/var/lib/postgresql/data
       - ./init-scripts:/docker-entrypoint-initdb.d
     command: postgres -c shared_preload_libraries=pg_stat_statements -c pg_stat_statements.track=all
      
  media-gateway:
    build: ./gateway
    ports:
      - "8080:8080"
    depends_on:
      - consul
      - redis
      - rabbitmq
    environment:
      CONSUL_ADDRESS: consul:8500
      REDIS_ADDRESS: redis:6379
      RABBITMQ_URL: amqp://admin:admin@rabbitmq:5672/
      
  media-server-1:
    build: ./server
    ports:
      - "8081:8081"
    depends_on:
      - consul
      - redis
      - minio
      - rabbitmq
      - postgres
    environment:
      CONSUL_ADDRESS: consul:8500
      REDIS_ADDRESS: redis:6379
      MINIO_ENDPOINT: minio:9000
      RABBITMQ_URL: amqp://admin:admin@rabbitmq:5672/
      DATABASE_DSN: "host=postgres user=postgres password=postgres dbname=monibuca port=5432 sslmode=disable TimeZone=Asia/Shanghai"
      
  media-server-2:
    build: ./server
    ports:
      - "8082:8081"
    depends_on:
      - consul
      - redis
      - minio
      - rabbitmq
      - postgres
    environment:
      CONSUL_ADDRESS: consul:8500
      REDIS_ADDRESS: redis:6379
      MINIO_ENDPOINT: minio:9000
      RABBITMQ_URL: amqp://admin:admin@rabbitmq:5672/
      DATABASE_DSN: "host=postgres user=postgres password=postgres dbname=monibuca port=5432 sslmode=disable TimeZone=Asia/Shanghai"
```

### Kubernetes 部署

```yaml
# k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: media-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: media-gateway
  template:
    metadata:
      labels:
        app: media-gateway
    spec:
      containers:
      - name: media-gateway
        image: monibuca/media-gateway:latest
        ports:
        - containerPort: 8080
        env:
        - name: CONSUL_ADDRESS
          value: "consul-service:8500"
        - name: REDIS_ADDRESS
          value: "redis-service:6379"
        - name: RABBITMQ_URL
          value: "amqp://admin:admin@rabbitmq-service:5672/"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: media-server
spec:
  replicas: 5
  selector:
    matchLabels:
      app: media-server
  template:
    metadata:
      labels:
        app: media-server
    spec:
      containers:
      - name: media-server
        image: monibuca/media-server:latest
        ports:
        - containerPort: 8081
        env:
        - name: CONSUL_ADDRESS
          value: "consul-service:8500"
        - name: REDIS_ADDRESS
          value: "redis-service:6379"
        - name: MINIO_ENDPOINT
          value: "minio-service:9000"
        - name: RABBITMQ_URL
          value: "amqp://admin:admin@rabbitmq-service:5672/"
        - name: DATABASE_DSN
          value: "host=postgres-service user=postgres password=postgres dbname=monibuca port=5432 sslmode=disable TimeZone=Asia/Shanghai"
```

## 性能优化策略

### 1. PostgreSQL 性能优化
- **连接池管理**: 使用 PgBouncer 或内置连接池
- **查询优化**: 使用 EXPLAIN ANALYZE 分析查询计划
- **索引优化**: 复合索引、部分索引、GIN 索引
- **分区表**: 按时间分区提高查询性能
- **并行查询**: 启用并行查询处理
- **统计信息**: 定期更新表统计信息

### 2. 缓存策略
- 流信息缓存：TTL 1小时
- 会话缓存：TTL 30分钟
- 配置缓存：TTL 24小时
- 热点数据：内存缓存 + Redis 二级缓存

### 2. 负载均衡
- 轮询算法：基础负载均衡
- 加权轮询：考虑服务器性能
- 最少连接：动态负载均衡
- 一致性哈希：流亲和性

### 3. 连接池管理
- Redis 连接池：最大连接数 100
- PostgreSQL 连接池：最大连接数 50（使用 PgBouncer 可扩展到 200+）
- RabbitMQ 连接池：最大连接数 20

### 4. PostgreSQL 配置优化
```yaml
# postgresql.conf 关键配置
shared_buffers = 256MB                    # 共享内存缓冲区
effective_cache_size = 1GB                # 有效缓存大小
work_mem = 4MB                           # 工作内存
maintenance_work_mem = 64MB              # 维护工作内存
checkpoint_completion_target = 0.9        # 检查点完成目标
wal_buffers = 16MB                       # WAL 缓冲区
default_statistics_target = 100          # 默认统计目标
random_page_cost = 1.1                   # 随机页面成本
effective_io_concurrency = 200           # 有效 I/O 并发
```

### 5. 异步处理
- 流事件：异步发布到消息队列
- 健康检查：异步执行
- 配置更新：异步通知
- 文件操作：异步处理

## 监控与告警

### 1. 监控指标
- 系统指标：CPU、内存、磁盘、网络
- 业务指标：并发流数、推拉流成功率、延迟
- 服务指标：响应时间、错误率、吞吐量

### 2. 告警规则
- 服务器离线：立即告警
- 高负载：CPU > 80% 持续 5 分钟
- 高延迟：平均延迟 > 1000ms
- 错误率：错误率 > 5%

### 3. 日志管理
- 结构化日志：JSON 格式
- 日志级别：DEBUG、INFO、WARN、ERROR
- 日志轮转：按大小和时间
- 日志聚合：ELK Stack

## 安全考虑

### 1. 认证授权
- JWT Token 认证
- RBAC 权限控制
- API 访问控制
- 流访问鉴权

### 2. 网络安全
- HTTPS/TLS 加密
- WebSocket 安全
- 防火墙配置
- DDoS 防护

### 3. 数据安全
- 敏感信息加密
- 数据备份策略
- 访问日志审计
- 数据脱敏处理

## 迁移计划

### 阶段一：基础设施搭建
1. 部署 Consul、Redis、MinIO、RabbitMQ
2. 搭建数据库集群
3. 配置监控和日志系统

### 阶段二：核心服务重构
1. 重构配置管理系统
2. 实现数据模型迁移
3. 重构 Web 服务框架
4. 集成缓存和存储系统

### 阶段三：网关和集群
1. 实现 Media Gateway
2. 建立 Media Server 集群
3. 实现服务发现和负载均衡
4. 测试集群功能

### 阶段四：优化和测试
1. 性能测试和优化
2. 压力测试和稳定性测试
3. 安全测试和漏洞修复
4. 生产环境部署

## PostgreSQL 优化特性

### 1. 数据类型优化
- **JSONB**: 使用 `jsonb` 类型存储流元数据，支持索引和高效查询
- **数组类型**: 使用 `text[]` 存储标签，支持数组操作和搜索
- **UUID**: 使用 `uuid-ossp` 扩展生成唯一标识符
- **时间类型**: 使用 `timestamptz` 存储时区感知的时间戳

### 2. 索引策略
```sql
-- 流信息复合索引
CREATE INDEX idx_stream_status_server ON streams(status, media_server_id) WHERE status = 'active';

-- 流路径全文搜索索引
CREATE INDEX idx_stream_path_gin ON streams USING gin(to_tsvector('english', stream_path));

-- 元数据 JSONB 索引
CREATE INDEX idx_stream_metadata_gin ON streams USING gin(metadata);

-- 标签数组索引
CREATE INDEX idx_stream_tags_gin ON streams USING gin(tags);
```

### 3. 分区表设计
```sql
-- 按时间分区的流记录表
CREATE TABLE stream_records (
    id BIGSERIAL,
    stream_id VARCHAR(255),
    record_data JSONB,
    created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

-- 创建月度分区
CREATE TABLE stream_records_2024_01 PARTITION OF stream_records
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE stream_records_2024_02 PARTITION OF stream_records
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### 4. 连接池配置
```yaml
# 数据库连接池优化
database:
  max_open_conns: 100
  max_idle_conns: 10
  conn_max_lifetime: "1h"
  conn_max_idle_time: "30m"
  
  # PostgreSQL 特有配置
  postgres:
    statement_cache_size: 1000
    default_query_exec_mode: "cache_statement"
    ssl_mode: "disable"
    timezone: "Asia/Shanghai"
```

### 5. 性能监控
```sql
-- 启用 pg_stat_statements 扩展
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 查询性能统计
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;

-- 表大小统计
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### 6. 数据模型迁移示例

```go
// 从 MySQL 迁移到 PostgreSQL 的数据模型
package models

import (
    "time"
    "gorm.io/gorm"
    "gorm.io/driver/postgres"
)

// 数据库迁移管理器
type MigrationManager struct {
    db *gorm.DB
}

// 执行迁移
func (mm *MigrationManager) Migrate() error {
    // 启用必要的扩展
    if err := mm.db.Exec("CREATE EXTENSION IF NOT EXISTS \"uuid-ossp\"").Error; err != nil {
        return err
    }
    
    if err := mm.db.Exec("CREATE EXTENSION IF NOT EXISTS \"pg_stat_statements\"").Error; err != nil {
        return err
    }
    
    // 自动迁移表结构
    return mm.db.AutoMigrate(
        &Stream{},
        &MediaServer{},
        &PushConfig{},
        &PullConfig{},
        &StreamRecord{},
        &User{},
    )
}

// 流记录表（分区表）
type StreamRecord struct {
    ID        uint      `gorm:"primarykey"`
    StreamID  string    `gorm:"index;not null"`
    RecordData JSON     `gorm:"type:jsonb"`
    CreatedAt time.Time `gorm:"index"`
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}

// 用户表
type User struct {
    gorm.Model
    Username    string    `gorm:"uniqueIndex;not null"`
    Password    string    `gorm:"not null"`
    Role        string    `gorm:"default:'user'"`
    Email       string    `gorm:"uniqueIndex"`
    LastLogin   *time.Time
    Permissions []string  `gorm:"type:text[]"`
}
```

### 7. 备份和恢复策略
```bash
#!/bin/bash
# PostgreSQL 备份脚本
BACKUP_DIR="/backup/postgres"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="monibuca"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
pg_dump -h localhost -U postgres -d $DB_NAME \
    --format=custom \
    --compress=9 \
    --file=$BACKUP_DIR/${DB_NAME}_${DATE}.dump

# 清理7天前的备份
find $BACKUP_DIR -name "*.dump" -mtime +7 -delete
```

## 总结

通过引入现代化的技术栈和架构模式，新的 Monibuca 系统将具备：

1. **高可用性**：集群架构 + 服务发现 + 健康检查
2. **高扩展性**：微服务架构 + 负载均衡 + 水平扩展
3. **高性能**：缓存策略 + 异步处理 + 连接池管理 + PostgreSQL 优化
4. **易维护性**：配置中心 + 监控告警 + 日志管理
5. **强安全性**：认证授权 + 网络安全 + 数据保护
6. **数据优势**：PostgreSQL 高级特性 + JSONB 支持 + 分区表 + 全文搜索

这种架构设计为 Monibuca 的未来发展奠定了坚实的基础，使其能够更好地应对大规模流媒体服务的挑战。PostgreSQL 的选择为系统提供了强大的数据存储能力、优秀的性能和丰富的扩展功能。
