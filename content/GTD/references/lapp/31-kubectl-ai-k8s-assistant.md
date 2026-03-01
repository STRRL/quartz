# kubectl-ai: AI Powered Kubernetes Assistant

- URL: https://github.com/GoogleCloudPlatform/kubectl-ai
- Source: Google Cloud Platform (官方)
- Status: Read

## Takeaway

- 自然语言驱动 kubectl 操作，查询+变更都支持，内置 kubectl + bash 工具，可自定义扩展
- 支持几乎所有主流 LLM（Gemini、OpenAI、Claude/Bedrock、Grok、ollama 本地）
- 可做 MCP server 也可做 MCP client

## 对 LAPP 的参考价值

### Session 持久化
- 支持 `--new-session`、`--resume-session`、`--list-sessions`
- 排障不是一次性的，保存/恢复对话上下文，跨多次交互追踪同一个问题
- 实现方式：session 保存到本地文件系统，不同接口间可共享

### Pipe 使用（Unix 哲学）
- `cat error.log | kubectl-ai "explain the error"` — stdin 作为上下文输入
- `kubectl-ai < query.txt` — 文件输入
- `echo "list pods" | kubectl-ai` — pipe 组合
- positional arg + stdin 可以同时使用（arg 作为 prefix）
- 让 AI 工具融入现有工作流而不是替代它
