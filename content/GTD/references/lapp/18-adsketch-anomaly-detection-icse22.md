# ADSketch: Adaptive Performance Anomaly Detection

- Venue: ICSE'22
- Authors: Zhuangbin Chen, Jinyang Liu et al. (Huawei Cloud)
- Paper: https://arxiv.org/abs/2201.02944
- Code: https://github.com/OpsPAI/ADSketch
- Status: Read

## Takeaway

- Anomaly detection on metrics (not logs) — detects performance degradation like slow response
- Key idea: cluster anomalous metric patterns into "sketches", so when a similar pattern appears again you immediately know what type of issue it is
- Interpretable: instead of just "anomaly detected", tells you which pattern group it matches — engineers can map patterns to known root causes
- Adaptive: online learning algorithm discovers new patterns as services evolve, no need to retrain from scratch
- Deployed at Huawei Cloud in production
- For LAPP: the "pattern sketching" concept could apply to log anomaly patterns in Phase 2 — cluster anomalous log sequences into recognizable patterns, build a library of known failure signatures
