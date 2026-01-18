# 迭代任务计划

## V1.0 核心微服务搭建
1. 后端：搭建Go-Zero基础框架，完成API网关开发
2. 后端：用户服务拆分（Command/Query/Event），实现用户注册/登录Command
3. 后端：用户服务Event Sourcing落地（UserCreateEvent存储/重放）
4. 后端：编写user.proto，生成GRPC代码
5. 前端：搭建Compose基础架构，实现登录/注册UI页面
6. 前后端联调：用户注册/登录接口对接