# GreptimeDB - Agent Observability

- Link: https://www.greptime.com/blogs/2025-12-11-agent-observability
- Status: Read

## Takeaway

- Blog post arguing traditional observability (metrics/logs/traces) still works for AI agents but needs extension
- Key shift: agent observability is about semantic quality (is the answer correct? is reasoning sound?) not just system health (is it up? whats the latency?)
- Data shape changed: agent events are semi-structured with 50-200 fields each, not clean metrics or plain text logs
- Scale example: 100K DAU agent app generates ~7.5 GB/day observability data, 1M+ DAU = 2+ TB/day
- Agent state (memory, context, reasoning steps) must be first-class observable — traditional traces treat it as a black box
- Multi-agent makes everything harder: cross-boundary trace propagation, state correlation, emergent behaviors
- Reinforces O11y 2.0 / Wide Events thesis: high-dimensionality data (200-500 dimensions) cant be covered by predefined metric aggregations
- For LAPP: less directly relevant (agent observability, not log parsing), but validates that the future of observability is high-cardinality structured events with read-time aggregation
