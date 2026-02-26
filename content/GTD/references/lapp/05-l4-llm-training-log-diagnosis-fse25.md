# L4: Diagnosing Large-scale LLM Training Failures via Automated Log Analysis

- Venue: FSE'25
- Authors: Zhihan Jiang et al. (CUHK + Huawei Cloud)
- Paper: https://arxiv.org/abs/2503.20263
- Status: Read

## Takeaway

- Domain-specific: LLM training failure diagnosis, not general log parsing
- Three key log patterns:
  - Cross-job: this run failed but last run was fine — diff the two, whatever is new/different is likely the problem
  - Spatial: some machines spit out different logs than the rest — those are probably the broken ones
  - Temporal: training has phases (init, loading, iterating, saving) — find which phase/iteration things went wrong
- 428 real failure reports studied: 74.1% failures during iterative training, hardware + user faults dominate, 89.9% diagnosis relies on manual log analysis
- L4 pipeline: parse raw logs -> cross-job filtering -> spatial anomaly detection (faulty nodes) -> temporal localization (faulty iterations)
- Results: 87.3% F1 for failure-indicating log identification, 80% top-5 accuracy for faulty node detection
- Not directly applicable to LAPP Phase 1 (log parsing), but the cross-job filtering idea (comparing against known-good baselines) is relevant for Phase 2 anomaly detection

## Details

- Platform-X: Huawei production AI platform, avg job size 72.8B params, avg 941 accelerators
- Failure symptoms: launching failure 21.3%, training crash 57.5%, abnormal behavior 16.6%, others 4.7%
- Key insight: single node fault can crash entire distributed training job due to synchronization
- Fault library: engineers summarize confirmed patterns for matching future similar failures
