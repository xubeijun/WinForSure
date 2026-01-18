# 编码规范（微服务+CQRS+Event Sourcing）
## 通用规则
- 命名规范：
  - 后端Go：
    - 微服务目录：小写（user/feed/message）
    - Command/Query：动词+名词（CreateUserCommand/GetUserQuery）
    - Event：名词+操作+Event（UserCreateEvent/UserUpdateEvent）
    - 分布式锁Key：[服务名]:[业务名]:[唯一标识]（如user:login:10086）
    - Proto文件：小写+下划线（user_service.proto），消息体首字母大写
  - 前端Kotlin：
    - Compose组件：首字母大写（UserLoginScreen、FeedItem）
    - ViewModel：业务域+ViewModel（UserViewModel、FeedViewModel）
    - 常量：全大写+下划线（API_BASE_URL、PAGE_SIZE）
- 注释规范：
  - 微服务入口/核心Command/Query：必须写文档注释（说明职责、输入输出）
  - Event事件：注释说明事件触发条件、状态变更逻辑
  - 分布式锁：注释说明锁的作用范围、超时时间
  - 复杂业务逻辑：加行内注释，说明设计思路

## 后端Go-Zero特殊规则
### 微服务拆分
- 单个微服务代码量控制在5000行内，避免过大
- 跨服务调用通过Proto定义的GRPC接口，禁止直接数据库访问
### CQRS+Event Sourcing
- Command层：只处理写操作，触发Event事件，不返回业务数据
- Query层：只处理读操作，从读库/缓存获取数据，不修改状态
- Event层：
  - 事件必须包含唯一ID、时间戳、业务标识、状态快照
  - 事件存储需持久化，支持事件重放（用于状态恢复）
### 高并发&分布式锁
- 仅在核心写操作（如用户注册、消息发送）使用分布式锁
- 分布式锁必须设置超时时间（3-5s），并加重试机制（最多3次）
- 优先使用Redis RedLock，避免单点故障
### Proto管理
- 统一使用Protobuf 3语法，字段编号不重复、不修改
- 按微服务拆分Proto文件，公共消息体放在`proto/common.proto`
### 设计模式
- 基础组件（日志、锁）：单例模式（Go-Zero内置单例可直接用）
- 业务规则（如消息推送方式）：策略模式
- 领域模型创建：工厂模式
- 禁止使用装饰器、代理等过度设计的模式

## 前端Kotlin特殊规则
- Compose组件：
  - 原子组件（如Button、Input）：纯UI，无业务逻辑
  - 业务组件（如UserCard）：包含简单业务逻辑，可复用
  - 页面组件（如UserProfileScreen）：组合业务组件，处理页面逻辑
- 状态管理：
  - 仅ViewModel持有可变状态，UI层只读
  - 网络请求结果通过StateFlow通知UI更新
- 数据交互：
  - 所有后端API调用封装在Repository层
  - 使用Proto生成的Model类，避免手动定义数据结构