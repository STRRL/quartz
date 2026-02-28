# TimesFM - Google Time Series Foundation Model

- GitHub: https://github.com/google-research/timesfm/
- Paper: https://arxiv.org/abs/2310.10688 (ICML 2024)
- HuggingFace: https://huggingface.co/collections/google/timesfm-release-66e4be5fdb56e960c1e482a6

## Summary
Google Research 的预训练时序预测基础模型。Decoder-only 架构。

- 最新版 2.5: 200M 参数，16K 上下文，连续分位数预测
- 开源：推理代码 + 模型权重（PyTorch/Flax），训练代码未开源
- BigQuery 集成，支持异常检测（Preview）
- 运维监控场景表现一般，Datadog 的 Toto 模型专门针对可观测性优化
