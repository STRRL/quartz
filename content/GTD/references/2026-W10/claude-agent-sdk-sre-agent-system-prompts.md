---
title: "Claude Agent SDK Cookbook #03 — SRE Agent System Prompts"
date: 2026-03-02
source: https://platform.claude.com/cookbook/claude-agent-sdk-03-the-site-reliability-agent
tags: [claude, agent-sdk, sre, system-prompt, mcp]
---

# Claude Agent SDK — SRE Incident Response Agent: System Prompts

来源: [The Site Reliability Agent](https://platform.claude.com/cookbook/claude-agent-sdk-03-the-site-reliability-agent)

## System Prompt

```
You are an expert SRE incident response bot. Your job is to investigate
production incidents quickly and thoroughly.

Investigation approach:
1. Start with get_service_health for a quick overview
2. Drill into error rates to identify affected services
3. Check latency — high latency often precedes errors
4. Investigate resources — DB connections, CPU, memory
5. Read container logs for specific error messages
6. Check config files for misconfigurations
7. Correlate and conclude — connect symptoms to root cause

Note: The api-server has baseline error noise (~0.1-0.2 errors/sec). Focus on significant spikes.

Be thorough but efficient. Always explain your reasoning.
```

## 关键设计洞察

- **System prompt 故意保持简单**: 只给出通用调查方法论（从 health check 开始 → 深入 metrics → 查 logs → 关联发现），不指定调用哪些 tool、什么顺序、如何解读结果
- **Tool descriptions 驱动 agent 行为**: 比精心设计的 prompt 更有效。每个 MCP tool 都有丰富的 JSON Schema description，agent 从中判断何时使用什么 tool
- **投入在 tool + 环境上，而非 prompt 上**: "invest in giving Claude access to the right tools and environment to perform tasks"

## Agent 配置

```python
options = ClaudeAgentOptions(
    system_prompt=SYSTEM_PROMPT,
    mcp_servers={
        "sre": {
            "command": sys.executable,
            "args": [str(MCP_SERVER_PATH)],
        }
    },
    allowed_tools=[
        # Investigation tools
        "mcp__sre__query_metrics",
        "mcp__sre__list_metrics",
        "mcp__sre__get_service_health",
        "mcp__sre__get_logs",
        "mcp__sre__get_alerts",
        "mcp__sre__get_recent_deployments",
        "mcp__sre__execute_runbook",
        # Remediation tools
        "mcp__sre__read_config_file",
        "mcp__sre__edit_config_file",
        "mcp__sre__run_shell_command",
        "mcp__sre__get_container_logs",
        # Documentation tools
        "mcp__sre__write_postmortem",
    ],
    hooks={
        "PreToolUse": [
            {
                "matcher": "mcp__sre__edit_config_file",
                "hooks": [{"type": "command", "command": f"bash {HOOKS_DIR}/validate_pool_size.sh"}],
            },
            {
                "matcher": "mcp__sre__run_shell_command",
                "hooks": [{"type": "command", "command": f"bash {HOOKS_DIR}/validate_config_before_deploy.sh"}],
            },
        ],
    },
    permission_mode="acceptEdits",
    model=MODEL,
)
```

## Tool Description 示例

Tool descriptions 是让 agent 自主工作的关键。示例：

### query_metrics
```
Query Prometheus metrics using PromQL.
Common investigation queries:
Error rate: rate(http_requests_total{status="500"}[1m]),
Error ratio: sum(rate(http_requests_total{status="500"}[1m])) by (service) / sum(rate(http_requests_total[1m])) by (service),
DB connections: db_connections_active,
Latency P99: http_request_duration_milliseconds{quantile="0.99"},
CPU usage: process_cpu_seconds_total.
```

### edit_config_file
```
Edit a configuration file to fix misconfigurations.
ONLY use this for remediation after confirming the root cause.
Restricted to files in the config/ directory.
```

### run_shell_command
```
Run a shell command for infrastructure management.
Restricted to docker-compose and docker commands only.
Use for: restarting services, checking container status, rebuilding images.
```

## Safety Hooks

两层防御：
1. **Tool handler 内置检查**: 目录限制、命令前缀白名单
2. **PreToolUse hooks**: shell 脚本验证变更内容（不只是位置）
   - `validate_pool_size.sh`: 检查 DB_POOL_SIZE 在 5-100 范围内
   - `validate_config_before_deploy.sh`: 部署前验证配置合理性

## Human-in-the-loop 设计

调查和修复分为两个阶段：
1. **Investigation**: 只用只读工具诊断问题
2. **Remediation**: 用写工具修复，需要人工审批后再执行

## Skills 扩展示例

```markdown
---
name: runbook
description: Execute documented runbooks for common SRE incidents.
---

## When to Use This Skill
Trigger this skill when you identify one of these patterns:

### Database Connection Exhaustion
- db_connections_active > 90
- "too many connections" in logs

### High Latency Cascade
- P99 latency > 1000ms on api-server
- Latency spreading to downstream services

## Workflow
1. Identify the incident type from symptoms
2. Call execute_runbook with phase="investigate"
3. Follow the diagnostic steps
4. Call execute_runbook with phase="remediate"
5. Present remediation options before taking action
```

## Takeaway

- not production ready, toy
- work as reference