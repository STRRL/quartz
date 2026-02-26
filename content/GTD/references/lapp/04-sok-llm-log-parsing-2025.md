# SoK: System Log Parsing with Large Language Models - A Review

- Paper: https://arxiv.org/abs/2504.04877
- Code: https://github.com/viktorb-aut/SoK-LLM-Log-Parsing (benchmark code)
- Status: Read

## Takeaway

- Recommended standard metrics: GA, PA, FTA, NED
  - GA (Grouping Accuracy): do logs that share a template get grouped together correctly? Measures clustering quality. A parser can get wrong templates but still group well.
  - PA (Parsing Accuracy): fraction of log messages where the extracted template exactly matches the ground truth. Strict per-message correctness.
  - FTA (F1-score of Template Accuracy): F1 over unique templates. Measures how many ground-truth templates the parser discovers correctly (precision + recall over template set).
  - NED (Normalized Edit Distance): edit distance between predicted and ground-truth templates, normalized to 0-1. Captures partial correctness when templates are close but not exact.
- Only two LLM parsers clearly lead the pack: LogBatcher, LILAC
- For LAPP: ICL (few-shot with strategic retrieval) is the sweet spot

### Supervision
- 6 fully supervised, 9 sup+unsup, rest unsupervised
- Supervised = needs labeled log-template pairs for ICL demonstrations
- More labels -> better performance, but labels are scarce and expensive
- LILAC and DivLog use specialized sampling algorithms to maximize diversity from limited labels

### Processing modes
- Stream (18 papers): one-by-one, best for real-time
- Batch (6): chunks, can optimize within batch
- Total (3): entire dataset at once, offline only
- For LAPP: stream mode with cache is the right default

### Reproducibility issues
- 41% are preprints
- Many papers don't share code or use inconsistent metrics/datasets
- Use Loghub-2.0 + GA/FGA/PA/FTA metrics for LAPP benchmarking

### Named approaches worth tracking
- LILAC (already reviewed in 01)
- LogBatcher: unsupervised, batch-based, DPP diversity within batches
- DivLog: DPP for candidate sampling
- LUNAR: buckets by length + frequent token, then cluster
- Lemur: CoT for template merging decisions
- LogPilot (different from 13, this is a parser): caching + streaming
