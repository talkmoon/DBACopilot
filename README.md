# DBACopilot · 数据库智能运维助手

> 用自然语言操作数据库的运维副驾：查数、诊断慢查询、翻故障案例。

## 为什么做这个

多数 AI 应用项目长得一样：聊天框 + 知识库。这个项目不一样——它建立在**真实的数据库运维经验**上：连接池耗尽、主备切换、慢查询、备份恢复，这些是我在华为云做 OnCall 时每天处理的问题。

投递时它要证明三件事：

1. 我懂数据库（领域深度，别人抄不走）
2. 我能把 LLM 接进企业系统（工程能力）
3. 我知道 AI 在运维场景的边界在哪（安全意识）

## 三个核心场景

| 场景 | 输入 | 输出 |
|---|---|---|
| 自然语言查数 | "上周每个租户的订单量，按降序" | 可执行 SQL + 结果表 + 用了哪些表/字段 |
| 慢查询诊断 | 慢日志 + 执行计划 | 瓶颈类型（缺索引/回表/临时表/锁等待）+ 改写建议 |
| 故障知识库 | "连接池被打满怎么排查" | 引用历史案例的分步排查路径 |

## 非目标（明确不做，避免摊子铺太大）

- ❌ 通用聊天机器人（没有差异化，面试官看腻了）
- ❌ 自动执行 DDL / 无 WHERE 的 UPDATE、DELETE（只生成 + 预检，执行永远要人确认）
- ❌ 训练或微调模型（用 API 就够，微调是 W8 之后的可选项）
- ❌ 支持所有数据库（先 MySQL，跑通再扩 PostgreSQL）

## 验收标准（v0.1，W4 末必须达到）

- [ ] 给定 schema，把 30 条中文问题转成 SQL，人工评分 **≥ 70% 可执行且语义正确**
- [ ] 危险语句拦截率 **100%**：DROP / TRUNCATE / 无 WHERE 的 UPDATE、DELETE / 跨库操作
- [ ] 每条 SQL 附带「用到的表 + 假设 + 置信度」，不确定时主动说"这个字段语义不明确"
- [ ] 慢日志输入 → 结构化诊断输出（瓶颈类型、依据、建议、置信度）
- [ ] 全链路 trace：prompt、模型、token、耗时、重试次数可查
- [ ] 30 条评测集可一键跑，输出通过率

> 数字必须自己测、可复现。简历上不写估算值，但面试时这个数字要能当场打开。

## 架构

```
┌──────────────┐     ┌──────────────────────────────┐
│  Web (简单页面) │────▶│  Spring Boot API              │
└──────────────┘     │  ├─ SchemaService  读表结构/注释   │
                     │  ├─ Nl2SqlService  Prompt + 校验   │
                     │  ├─ SqlGuard       危险语句拦截     │
                     │  ├─ DiagnoseService 慢日志分析      │
                     │  └─ RagService     故障案例检索     │
                     └───────┬──────────────┬────────┘
                             │              │
                     ┌───────▼──────┐ ┌─────▼─────────┐
                     │ MySQL (业务库) │ │ pgvector (知识库)│
                     └──────────────┘ └───────────────┘
```

关键设计：**SQL 生成与 SQL 执行分离**。LLM 只产出 SQL 文本 + 解释，执行走独立的受控通道（只读账号、超时、行数上限、EXPLAIN 预检）。

## 技术栈

- Java 21 · Spring Boot 3 · Spring AI（W2 引入）
- MySQL 8（业务库）· PostgreSQL + pgvector（向量检索）
- 前端：先 Thymeleaf 或极简 HTML，W6 再考虑 Vue
- 观测：Micrometer + Langfuse（W6）

## 里程碑

| 周 | 目标 | 完成标志 |
|---|---|---|
| W2 | 骨架 + schema 读取 | `/api/schema` 返回表结构 JSON |
| W3 | NL2SQL v1 + 危险语句拦截 | 10 条问题能出 SQL，拦截用例全过 |
| W4 | 评测集 + 诊断 MVP | 30 条评测跑出通过率；慢日志能出诊断 |
| W6 | 知识库 RAG + 审计 | 故障问题能带引用回答 |
| W7 | 上线 | 公网可访问 + 3 分钟演示视频 |

## 如何运行

前置：JDK 21、Maven 3.9+、Docker。

```bash
# 1. 起依赖
docker run -d --name dbacopilot-mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=demo mysql:8

docker run -d --name dbacopilot-pg -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres pgvector/pgvector:pg16

# 2. 跑起来
mvn spring-boot:run

# 3. 验证
curl http://localhost:8080/api/health
```

## 目录

```
src/main/java/com/talkmoon/dbacopilot/
├── DbaCopilotApplication.java
├── api/            # REST 接口
├── service/        # 业务编排（NL2SQL、诊断、RAG）
├── domain/         # 领域模型
└── infra/          # DB 访问、LLM 客户端、向量库
docs/DESIGN.md      # 架构决策记录（ADR）
```

## 状态

🚧 W1：仓库初始化，只有健康检查接口。W2 开始接 Spring AI。
