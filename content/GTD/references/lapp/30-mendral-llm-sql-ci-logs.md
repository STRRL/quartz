# LLMs Are Good at SQL: Giving an Agent Terabytes of CI Logs

- URL: https://www.mendral.com/blog/llms-are-good-at-sql
- Published: 2026-02
- Company: Mendral
- Status: Read

## Takeaway

- 给 LLM agent 直接暴露 SQL 接口，比预定义 tool API 灵活得多——agent 能问出你没预料到的问题
- LLM 天然擅长 SQL（训练数据里大量 SQL，语法与自然语言映射好）
- 用 ClickHouse 列式存储 CI logs，反范式化（每行 log 带 48 列元数据），5.3 TiB 压缩到 154 GiB（35:1）
- Agent 探索模式：平均每次调查 4.4 个 query，先查 job 元数据（便宜），再钻 raw log（精准）
- 8500+ 次调查、52000+ 次 query 的生产验证，P95 单次扫 9.4 亿行
- 重复元数据列压缩比极高（commit_message 301:1），反范式化在列式存储中几乎免费
- Skip index（bloom filter + ngram）让全文搜索只读必要数据

## Key Insight

structured data + SQL > rigid tool API。让 agent 自己决定怎么查数据，而非限制在预设的工具集里。对 LAPP 的启发：考虑将 log 数据结构化存储后暴露 SQL 查询能力给 agent。
