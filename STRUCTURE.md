# 目录说明
- 「核心」标注为项目运行必备模块
- 「规范」标注为 Cursor AI 编程 / 团队协作必备文件
- 「工具」标注为辅助开发 / 运维脚本
- 「案例」标注为独立技术沉淀模块（不影响业务）

WinForSure/
├── README.md                 # 项目入口：概览、快速启动、规范指引（核心）
├── AI.md                     # 规范：Cursor AI 开发指令（角色/技术栈/输出约束）
├── STRUCTURE.md              # 规范：本文件（完整目录结构说明）
├── STANDARDS.md              # 规范：编码细则（CQRS/命名/分布式锁/Compose）
├── TASKS.md                  # 规范：迭代任务计划（按版本拆分开发任务）
├── LICENSE                   # 项目许可证（MIT）
├── .gitignore                # 规范：Git 忽略规则（Go/Android/Proto/临时文件）
├── docker-compose.yml        # 工具：一键启动依赖服务（MySQL/Redis/MQ）
├── docs/                     # 文档库
│   ├── api/                  # API 文档：Swagger/Postman 导出（按微服务拆分）
│   ├── cqrs/                 # CQRS+Event Sourcing 设计文档（事件流/状态重构）
│   ├── distributed-lock/     # 分布式锁设计：核心场景/超时重试策略
│   └── proto/                # Proto 接口文档：字段注释/调用示例
├── scripts/                  # 工具：运维/开发脚本（核心）
│   ├── start-gateway.sh      # 启动 API 网关
│   ├── start-services.sh     # 批量启动所有微服务
│   ├── event-replay.sh       # Event Sourcing 事件重放（状态恢复）
│   ├── lock-test.sh          # 分布式锁高并发压测
│   ├── generate-proto.sh     # 批量生成 Proto GRPC 代码
│   └── db-migrate.sh         # 数据库迁移（用户/消息/动态表）
├── backend/                  # Go-Zero 微服务后端（核心）
│   ├── go.mod                # Go 模块依赖（Go 1.22+、Go-Zero v1.6+）
│   ├── go.sum
│   ├── api-gateway/          # API 网关：统一鉴权/限流/路由转发（核心）
│   │   ├── etc/              # 网关配置
│   │   │   └── gateway.yaml  # 端口/路由规则/限流阈值
│   │   ├── handler/          # 网关拦截器
│   │   │   ├── auth_handler.go   # 统一鉴权
│   │   │   ├── log_handler.go    # 链路日志
│   │   │   └── limit_handler.go  # 接口限流
│   │   └── gateway.go        # 网关启动入口
│   ├── services/             # 业务微服务（按领域拆分，单一职责）
│   │   ├── user/             # 用户服务（CQRS 核心示例）
│   │   │   ├── cmd/          # 服务启动入口
│   │   │   │   ├── api/      # HTTP API 服务（对外 RESTful）
│   │   │   │   │   ├── etc/
│   │   │   │   │   │   └── user-api.yaml
│   │   │   │   │   └── main.go
│   │   │   │   └── rpc/      # GRPC 服务（对内微服务调用）
│   │   │   │       ├── etc/
│   │   │   │       │   └── user-rpc.yaml
│   │   │   │       └── main.go
│   │   │   ├── etc/          # 服务配置（Redis/MQ/锁超时）
│   │   │   ├── command/      # CQRS-Command：仅处理写操作（核心）
│   │   │   │   ├── create_user.go    # 创建用户（触发 UserCreateEvent）
│   │   │   │   ├── update_user.go    # 更新用户（触发 UserUpdateEvent）
│   │   │   │   ├── follow_user.go    # 关注用户（加分布式锁）
│   │   │   │   └── command_test.go   # Command 单元测试
│   │   │   ├── query/        # CQRS-Query：仅处理读操作（无状态修改）
│   │   │   │   ├── get_user.go       # 查询用户信息（读缓存）
│   │   │   │   ├── list_followers.go # 查询粉丝列表（分页）
│   │   │   │   └── query_test.go     # Query 单元测试
│   │   │   ├── event/        # Event Sourcing：事件管理（核心）
│   │   │   │   ├── user_event.go     # 事件定义（UserCreateEvent/UpdateEvent）
│   │   │   │   ├── event_handler.go  # 事件处理器（持久化/通知）
│   │   │   │   ├── event_store.go    # 事件存储（MySQL 实现）
│   │   │   │   └── event_replay.go   # 事件重放（状态恢复）
│   │   │   ├── model/        # 领域模型：纯结构体（无依赖）
│   │   │   │   ├── user.go           # 用户实体
│   │   │   │   └── user_repository.go # 数据访问层
│   │   │   ├── internal/     # Go-Zero 内部逻辑（自动生成+业务桥接）
│   │   │   │   ├── handler/  # API 处理器（转发 Command/Query）
│   │   │   │   ├── logic/    # 业务逻辑桥接
│   │   │   │   └── svc/      # 服务上下文（依赖注入：锁/事件存储）
│   │   │   └── user.sql      # 数据库脚本：用户表/用户事件表
│   │   ├── feed/             # 朋友圈服务（同 User 服务结构）
│   │   │   ├── cmd/
│   │   │   ├── etc/
│   │   │   ├── command/      # 发布动态/点赞（加分布式锁防重复）
│   │   │   ├── query/        # 查询时间线/动态详情（读缓存）
│   │   │   ├── event/        # FeedCreateEvent/FeedLikeEvent
│   │   │   ├── model/
│   │   │   ├── internal/
│   │   │   └── feed.sql      # 动态表/动态事件表
│   │   └── message/          # 消息服务（高并发核心）
│   │       ├── cmd/
│   │       ├── etc/
│   │       ├── command/      # 发送消息/撤回消息（分布式锁+事件）
│   │       ├── query/        # 查询会话/消息列表（读库分离）
│   │       ├── event/        # MessageSendEvent/MessageReadEvent
│   │       ├── model/
│   │       ├── internal/
│   │       └── message.sql   # 消息表/会话表/消息事件表
│   ├── shared/               # 共享基础组件（高内聚，全服务复用）
│   │   ├── lock/             # 分布式锁组件（Redis RedLock）
│   │   │   ├── redis_lock.go # 锁实现（超时/重试/防死锁）
│   │   │   └── lock_test.go  # 单元测试
│   │   ├── logger/           # 统一日志（链路追踪/级别控制）
│   │   │   └── logger.go
│   │   ├── eventstore/       # 事件存储核心组件（适配 Event Sourcing）
│   │   │   ├── event_store.go # 通用接口
│   │   │   └── mysql_event_store.go # MySQL 实现
│   │   ├── config/           # 全局配置（读取 YAML/环境变量）
│   │   │   └── config.go
│   │   ├── tracer/           # 链路追踪（OpenTelemetry）
│   │   │   └── tracer.go
│   │   └── util/             # 通用工具（加密/时间/Proto 转换）
│   │       ├── crypto.go
│   │       ├── time_util.go
│   │       └── proto_util.go
│   └── proto/                # 统一 Proto 管理（Protobuf 3）
│       ├── common.proto      # 公共消息体：分页/响应/错误码
│       ├── user.proto        # 用户服务 GRPC 接口
│       ├── feed.proto        # 朋友圈服务 GRPC 接口
│       ├── message.proto     # 消息服务 GRPC 接口
│       └── gen/              # 自动生成的 Proto 代码（不手动修改）
│           ├── user.pb.go
│           ├── feed.pb.go
│           └── message.pb.go
├── frontend/                 # Kotlin/Jetpack Compose 前端（核心）
│   ├── build.gradle.kts      # 项目级构建配置（AGP 8.2+）
│   ├── gradle/
│   ├── gradlew
│   ├── gradlew.bat
│   ├── settings.gradle.kts
│   └── app/                  # 主应用模块
│       ├── build.gradle.kts  # 模块配置（Kotlin 1.9+、Compose 1.6+）
│       ├── proguard-rules.pro # 混淆规则
│       └── src/
│           ├── main/
│           │   ├── java/com/winforsure/social/
│           │   │   ├── SocialApp.kt    # 应用入口（初始化组件）
│           │   │   ├── data/           # 数据层（对接后端 Proto）
│           │   │   │   ├── repository/ # 仓库层（按业务域拆分）
│           │   │   │   │   ├── UserRepository.kt
│           │   │   │   │   ├── FeedRepository.kt
│           │   │   │   │   └── MessageRepository.kt
│           │   │   │   ├── remote/     # 远程 API（Retrofit+Proto）
│           │   │   │   │   ├── ApiService.kt
│           │   │   │   │   └── interceptor/
│           │   │   │   │       ├── AuthInterceptor.kt # 鉴权拦截
│           │   │   │   │       └── LogInterceptor.kt  # 日志拦截
│           │   │   │   ├── local/      # 本地存储（Room/SharedPreferences）
│           │   │   │   │   ├── AppDatabase.kt
│           │   │   │   │   └── PreferenceManager.kt
│           │   │   │   └── model/      # 数据模型
│           │   │   │       ├── proto/  # 自动生成的 Proto Model
│           │   │   │       └── ui/     # UI 专用模型（解耦后端）
│           │   │   ├── domain/         # 领域层（业务用例）
│           │   │   │   └── usecase/
│           │   │   │       ├── user/
│           │   │   │       │   ├── LoginUseCase.kt
│           │   │   │       │   └── FollowUserUseCase.kt
│           │   │   │       ├── feed/
│           │   │   │       │   ├── PublishFeedUseCase.kt
│           │   │   │       │   └── GetTimelineUseCase.kt
│           │   │   │       └── message/
│           │   │   │           ├── SendMessageUseCase.kt
│           │   │   │           └── GetMessageListUseCase.kt
│           │   │   ├── presentation/   # 表现层（MVVM+Compose）
│           │   │   │   ├── ui/         # Compose UI 组件
│           │   │   │   │   ├── common/ # 原子组件（通用）
│           │   │   │   │   │   ├── PrimaryButton.kt
│           │   │   │   │   │   ├── InputField.kt
│           │   │   │   │   │   └── LoadingDialog.kt
│           │   │   │   │   ├── user/   # 用户业务组件
│           │   │   │   │   │   ├── LoginScreen.kt
│           │   │   │   │   │   ├── UserProfileScreen.kt
│           │   │   │   │   │   └── FollowerListScreen.kt
│           │   │   │   │   ├── feed/   # 朋友圈业务组件
│           │   │   │   │   │   ├── FeedItem.kt
│           │   │   │   │   │   ├── TimelineScreen.kt
│           │   │   │   │   │   └── PublishFeedScreen.kt
│           │   │   │   │   └── message/ # 消息业务组件
│           │   │   │   │       ├── ChatScreen.kt
│           │   │   │   │       ├── MessageItem.kt
│           │   │   │   │       └── ConversationListScreen.kt
│           │   │   │   ├── viewmodel/  # ViewModel（状态管理）
│           │   │   │   │   ├── UserViewModel.kt
│           │   │   │   │   ├── FeedViewModel.kt
│           │   │   │   │   └── MessageViewModel.kt
│           │   │   │   ├── navigation/ # Compose Navigation（单 Activity）
│           │   │   │   │   └── NavGraph.kt
│           │   │   │   └── theme/      # Compose 主题
│           │   │   │       ├── Color.kt
│           │   │   │       ├── Type.kt
│           │   │   │       └── Shape.kt
│           │   │   └── util/           # 前端工具类
│           │   │       ├── PermissionUtil.kt
│           │   │       ├── ToastUtil.kt
│           │   │       └── ProtoConverter.kt
│           │   ├── res/                 # 资源文件
│           │   │   ├── drawable/        # 图片/矢量图
│           │   │   ├── mipmap/          # 应用图标
│           │   │   ├── values/          # 字符串/颜色/尺寸
│           │   │   └── theme/           # Compose 主题资源
│           │   └── AndroidManifest.xml # 清单文件（单 Activity）
│           ├── test/                    # 单元测试（ViewModel/Usecase）
│           │   └── java/com/winforsure/social/
│           │       ├── viewmodel/
│           │       └── domain/usecase/
│           └── androidTest/             # 集成测试（Compose UI）
│               └── java/com/winforsure/social/
│                   └── ui/
└── cases/                    # Go 经典案例库（案例）
    ├── go.mod
    ├── README.md             # 案例说明
    ├── algorithm/            # 算法案例
    │   ├── sort/             # 排序：冒泡/快排/归并
    │   ├── search/           # 搜索：二分/BFS/DFS
    │   └── dp/               # 动态规划：斐波那契/背包
    ├── design_pattern/       # 设计模式（微服务适配）
    │   ├── creational/       # 创建型：工厂/单例/建造者
    │   ├── structural/       # 结构型：适配器/装饰器
    │   └── behavioral/       # 行为型：策略/观察者
    └── distributed/          # 分布式案例
        ├── lock/             # 分布式锁示例
        ├── rate_limit/       # 限流示例
        └── event_sourcing/   # 事件溯源示例