# Datadog Toto - Time Series Foundation Model for Observability

- Paper: https://arxiv.org/abs/2407.07874 (July 2024)
- GitHub: https://github.com/DataDog/toto
- Model: https://huggingface.co/Datadog/Toto-Open-Base-1.0 (open weights)
- Blog: https://www.datadoghq.com/blog/datadog-time-series-foundation-model/
- BOOM Benchmark: https://huggingface.co/spaces/Datadog/BOOM

## Summary
Datadog 开发的时序基础模型，专门针对可观测性/运维监控数据优化。

- 训练数据：近1万亿数据点（7500亿来自 Datadog 内部匿名监控数据 + 公开时序数据集 LOTSA）
- 已开源 open-weights 版本（Toto-Open-Base-1.0）
- 在 GIFT-Eval 和 LSF benchmark 上排名第一
- 专门发布了 BOOM（Benchmark of Observability Metrics）评测集
- Zero-shot 推理，无需针对单个指标训练

## 社区评价
- Reddit r/MachineLearning: 有人质疑 TSFM 在实际运维场景的价值，认为统计模型+领域知识更可靠
- 也有人认为 zero-shot 能力对大规模监控场景很有价值（不需要为每个 metric 单独训练）
