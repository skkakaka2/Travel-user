# travel-user

家庭自驾游规划服务的用户微服务骨架。当前仅完成底包继承、基础设施接入和 Nacos 配置模板，尚未定义用户领域模型、数据库表、接口或业务逻辑。

## 技术基线

- Java 21 / Spring Boot 3.5.16
- `zy-starter-parent:1.0.0-SNAPSHOT`
- MySQL 8 / MyBatis-Plus
- Redis / OpenFeign / Nacos
- JWT 安全、Springdoc OpenAPI、Actuator

## 使用前提

服务通过本地 Maven 仓库解析 `zy-starter-parent` 与全部 `zy-common-*` 模块。首次使用前，需要在 `zy-starter-parent` 完整 reactor 中完成本地安装。

服务坐标与应用名均为 `travel-user`，Java 基础包为 `com.zy.travel.user`。

## Nacos 配置

根目录 `nacos` 下维护两份运行期配置文件。Nacos 部署后，以 Group `DEFAULT_GROUP` 上传对应 Data ID：

| 文件 | Data ID | 环境 |
|---|---|---|
| `travel-user-dev.yaml` | `travel-user-dev.yaml` | 开发环境 |
| `travel-user-prod.yaml` | `travel-user-prod.yaml` | 生产环境 |

本地 `src/main/resources/application.yml` 仅保留 Nacos 引导参数，默认激活 `dev` 并导入 `travel-user-dev.yaml`。设置 `TRAVEL_PROFILE=prod` 后，应用改为导入 `travel-user-prod.yaml`；其中不包含数据库、Redis、JWT 等运行期配置。

两个模板使用相同的环境变量名称；开发和生产部署分别注入各自的变量值，从而隔离数据库、Redis 和密钥。

| 环境变量 | 值或用途 |
|---|---|
| Nacos 地址 | `pi.home:8848` |
| Nacos 命名空间 | `dev` |
| Nacos 分组 | `DEFAULT_GROUP` |
| Nacos 账号 | `nacos` |

两份 Nacos 配置中的连接信息均使用环境变量占位符：

| 环境变量 | 用途 |
|---|---|
| `DB_URL` | MySQL JDBC URL |
| `DB_USERNAME` | MySQL 用户名 |
| `DB_PASSWORD` | MySQL 密码 |
| `ZY_SECURITY_JWT_SECRET` | Base64URL 编码的 HS256 密钥，解码后至少 32 字节 |
| `SERVER_PORT` | 可选，HTTP 端口，默认 `8080` |

Nacos 下发的 Redis 配置使用下列环境变量：

| 环境变量 | 用途 |
|---|---|
| `REDIS_HOST` | Redis 主机 |
| `REDIS_PORT` | Redis 端口，默认 `6379` |
| `REDIS_USERNAME` | 可选的 Redis 用户名 |
| `REDIS_PASSWORD` | 可选的 Redis 密码 |
| `REDIS_DATABASE` | Redis 数据库序号，默认 `0` |

Nacos 配置使用 `spring.config.import`，不使用 `bootstrap.yml`。Nacos 引导参数已固化在本地 `application.yml`；数据库、Redis 与 JWT 密钥仍仅由运行环境注入，两个 Nacos 配置文件均不包含真实凭据。Nacos 未部署或当前环境对应的 Data ID 缺失时，服务不应启动。

## 当前基础端点

- `GET /actuator/health`
- `GET /v3/api-docs`
- `GET /swagger-ui/index.html`

除公共健康检查和文档端点外，JWT 安全模块默认要求认证。后续增加业务接口时，再按领域需要补充公开路径和授权策略。
