
刚刚结束了三天的小 Hackathon, 快速记录一下产出和收获

## 背景

又很有有意思的项目, 如果再不做感觉就没机会做了; 能从编程中获得快乐的机会不多了.

主要目的

- 个人兴趣, 好玩, keep sharp
- 新领域探索, 先不考虑制约条件, 只是用 SOTA 技术, agentic 能够做什么事情 
- 学一下 Stacked PR 实践

## Review

- 熟悉 stacked PR 流程, 在 git-flow 上的快速迭代方案; 目前 AI 产出太高, 我倾向于未来还是需要人类理解才行, 这种 stacked PR 模式需要学起来; 
	- 对比了 aviator 和 git town, 这次用了前者
	- 还不错
	- 下次有机会再试试 git town
- 字节的 eino 框架还不错, https://github.com/cloudwego/eino
- 关于项目本身
	- https://github.com/STRRL/lapp
	- 预期
		- 第一天调研, 第二天糊核心实现, 第三天做产品方向拓展
	- 实际
		- 第一天下午才起, 半天时间调研完成(超出预期)
		- 第二天糊核心实现, 流程走通, 但是代码很屎(低于预期)
		- 第三天铲屎, 铲了一天, 基本满意(低于预期)
	- 未来计划
		- Xuanwo 哥哥提到了为什么不每天抽出十分钟来看一下呢
		- 确实, 人们总是高估一天能做的事情, 但是低估一年能做的事情
		- 这两天请假集中做, 对我来说就和请假去旅游一样, 很开心
		- 但是手头上有趣项目有点多, 想雨露均沾也有点不现实
		- 先做起来, 固定时间而不是固定项目

项目效果

```
$ OTEL_TRACING_ENABLED=true go run ./cmd/lapp/ analyze ~/Library/Logs/haye/main.log "what happened about acp?"
2026/02/27 21:03:10 INFO langfuse tracing enabled host=http://localhost:3000
2026/02/27 21:03:10 INFO OpenTelemetry tracing enabled endpoint=http://localhost:4318 service=lapp
2026/02/27 21:03:10 INFO Reading logs...
2026/02/27 21:03:10 INFO Read lines lines=25472 merged_entries=4472
2026/02/27 21:03:10 INFO Labeling patterns count=83
2026/02/27 21:03:46 INFO Ingestion complete lines=4472 templates=209 patterns_with_2+_matches=83
2026/02/27 21:03:46 INFO Database stored path=lapp.duckdb
2026/02/27 21:03:46 INFO Parsing lines count=4472
2026/02/27 21:03:50 INFO Analyzing with model model=google/gemini-3-flash-preview
2026/02/27 21:04:00 INFO Based on the logs, here is the analysis of what happened with **ACP (Agentic Protocol / Anthropic Computer Protocol)**:

### 1. Key Findings
The logs reveal that ACP is used as a mechanism to provide dynamic tools (like `ls`, `find`, `cat`) to an AI agent. There are three main areas of activity:

*   **Tool Execution & Failures:**
    *   The system uses a tool named `acp.acp_provider_agent_dynamic_tool`.
    *   There are multiple instances of `AI_NoSuchToolError` (e.g., at `15:07:33.473`). This often occurs when the model tries to call a shell command (like `` `ls -la ...` ``) as if it were the tool name itself, rather than passing the command as an argument to the ACP tool.
    *   The logs show "invalid": true for many of these tool calls, indicating the LLM struggled with the specific schema expected by the ACP provider.

*   **Binary Provider Resolution:**
    *   The app attempts to locate the `claude-code-acp` binary on startup.
    *   **Earlier logs (2026-02-01):** Failed to find the packaged version and fell back to a global command:
        `[warn] Failed to locate claude-code-acp in packaged app, falling back to global command`
    *   **Later logs (2026-02-17):** Successfully located and used the packaged CLI or a dev version:
        `[info] Using packaged claude-code-acp CLI: /Applications/Haye.app/Contents/Resources/app/.webpack/main/node_modules/@zed-industries/claude-code-acp/dist/index.js`

*   **Debug Activity:**
    *   There is verbose logging prefixed with `[ACP DEBUG]` showing the raw JSON of tool calls, inputs, and errors, suggesting active troubleshooting or development of the ACP integration.

### 2. Anomalies Detected
*   **`AI_NoSuchToolError`:** The agent frequently misformats calls. Instead of calling `acp_provider_agent_dynamic_tool` with `command: "ls"`, it sometimes tries to call a tool literally named `` `ls -la ...` ``.
*   **Pathing Fallback:** The warning about failing to locate the packaged CLI suggests a potential packaging/distribution issue in earlier versions of the app (`Haye.app`).

### 3. Root Cause Analysis
The "issues" with ACP appear to be twofold:
1.  **Configuration/Environment:** On some runs, the environment didn't have the ACP CLI in the expected packaged path, forcing a fallback to a global installation which might be missing or different.
2.  **Model Prompting/Schema:** The agent (Claude) sometimes hallucinates the tool names or fails to adhere to the dynamic tool schema, leading to `invalid` tool calls.

### 4. Suggested Next Steps
*   **Prompt Engineering:** Refine the system prompt for the agent to ensure it uses the `acp.acp_provider_agent_dynamic_tool` correctly with the `command` argument rather than treating commands as tool names.
*   **Verification:** Ensure `claude-code-acp` is correctly bundled in the production build to avoid the "fallback to global" warning, which can lead to non-deterministic behavior across different user machines.
*   **Error Handling:** Improve the ACP provider's validation logic to give more constructive feedback to the model when a tool call is marked `invalid`.
```

![[Pasted image 20260228122741.png]]
![[Pasted image 20260228122808.png]]
![[Pasted image 20260228122817.png]]
