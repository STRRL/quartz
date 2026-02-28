# k8sgpt - LLM-powered Kubernetes Diagnostics

- Project: https://github.com/k8sgpt-ai/k8sgpt
- Stars: 7,400+
- Status: Read (cloned and reviewed source code)

## Takeaway

- Architecture: pluggable analyzers (one per K8s resource type: pod, deployment, service, ingress, etc.) each produce structured "failures", then LLM explains them in plain English
- Log analyzer is dead simple: regex match "error|exception|fail" on last 100 lines, feed matching lines to LLM. No parsing, no templates, no structure
- This is basically the gap LAPP fills: k8sgpt finds log errors by regex, LAPP would understand log structure and find anomalies that regex cant catch
- Has MCP server built in — can be used as a tool by AI agents
- Cache layer for LLM results (file-based, S3, GCS, Azure) — same idea as LILAC/LogBatcher caching
- Custom analyzer plugin system: users can write their own analyzers — good extensibility pattern for LAPP
- For LAPP: k8sgpt proves the market (7.4K stars), but its log analysis is primitive. LAPP doing real log parsing + anomaly detection would be a massive upgrade over "grep for error"
