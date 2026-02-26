# LogPilot: Intent-aware and Scalable Alert Diagnosis

- Venue: ASE'25
- Authors: Zhihan Jiang, Jinyang Liu et al. (same group as L4)
- Paper: https://arxiv.org/abs/2509.25874
- Deployed at: Volcano Engine Cloud (ByteDance)
- Status: Read

## Takeaway

- Alert diagnosis, not log parsing — given an alert (e.g., PromQL firing), auto-find root cause from logs
- Intent-aware: reads the alert definition (PromQL query) to understand what the alert cares about, then scopes which logs/requests are relevant
- Builds "spatiotemporal log chains" per request (trace-like reconstruction from logs), then clusters similar chains to find patterns
- Clustering keeps LLM input compact: send representative samples instead of all logs
- Results: 50% better root cause summaries, 55% better exact localization vs baselines
- Fast and cheap: under 1 min per alert, $0.074 per diagnosis
- For LAPP: the intent-aware scoping idea is interesting — if we know what the user cares about (alert definition), we can narrow down which logs to analyze instead of boiling the ocean
