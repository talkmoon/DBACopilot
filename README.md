# DBACopilot
DBACopilot数据库智能运维助手
自然语言 → SQL（NL2SQL），带 schema 理解、执行前 dry-run、危险语句拦截
慢查询诊断：慢日志 → LLM 分析执行计划 → 索引/改写建议
运维知识库 RAG：把 MySQL/PG/MongoDB 故障案例做成可检索问答
技术栈：Spring Boot 3 + Spring AI + pgvector + MySQL/PostgreSQL + 简易前端
可量化：SQL 执行正确率（100 条测试集）、诊断建议采纳率、平均定位耗时下降比例
