# Loghub 2.0: Large-scale Evaluation for Log Parsing

- Venue: ISSTA'24
- Authors: Zhihan Jiang, Jinyang Liu et al.
- Paper: https://arxiv.org/abs/2308.10828
- Project: https://github.com/logpai/Loghub-2.0
- Status: Read

## Takeaway


- 14 datasets, avg 3.6M log lines each (vs Loghub-2k's 2000 lines) - much more realistic
- Datasets: Hadoop, HDFS, OpenStack, Spark, Zookeeper, BGL, HPC, Thunderbird, Linux, Mac, Apache, OpenSSH, HealthApp, Proxifier
- 15 parsers evaluated: AEL, Drain, IPLoM, LenMa, LFA, LogCluster, LogMine, Logram, LogSig, MoLFI, SHISO, SLCT, Spell, UniParser, LogPPT
- Key finding: all parsers perform significantly worse on large-scale data vs 2k samples
- Best traditional parsers on GA (grouping accuracy): Drain (0.85), AEL (0.81)
- ML-based parsers (UniParser, LogPPT) dominate on PA (parsing accuracy): LogPPT 0.76, UniParser 0.68 vs Drain 0.47
- Rare log events are hardest to parse correctly - critical for diagnosis but poorly handled by all parsers
- New metric FGA (frequency-weighted GA) to handle imbalanced distributions
- Many parsers crash or timeout on large datasets (marked as ------ in results)
- Drain is the best balance of speed + accuracy among traditional parsers
- Most important value for LAPP: the dataset itself. Use Loghub-2.0 for integration testing and benchmarking
  - Download: https://zenodo.org/record/8275861
  - 14 real-world system log datasets with ground truth annotations
- Drain as baseline, target beating LogPPT's PA scores with LLM approach

### Parser survivability on full-scale datasets

Only 7 out of 15 parsers completed all 14 large-scale datasets without crash/timeout:
Drain, IPLoM, LFA, LogCluster, LogSig, UniParser, LogPPT

### Parser ranking (combining efficiency, FGA, PA/FTA, stability)

1st: Drain
- Highest average GA and FGA (paper conclusion)
- Strongest grouping, stable template clustering, low variance at scale
- Linear complexity, no GPU, online tree-based, streaming friendly
- Best for: real-time log streams, high throughput, production use

2nd: IPLoM (conservative/stable)
- Also statistical, decent efficiency, stable grouping
- Slightly weaker than Drain overall
- Simple implementation, interpretable, few parameters
- Best for: rule-heavy log systems, minimal tuning

3rd tier: LogCluster / LogSig / LFA
- Can finish all datasets, statistical, decent efficiency
- FGA lower than Drain, weak on rare templates and high-parameter templates
- Best for: low accuracy requirements, batch processing, legacy migration

Semantic-based: UniParser / LogPPT (conditional recommendation)
- Best PA/FTA scores (token-level accuracy): LogPPT 0.76, UniParser 0.68
- But GA/FGA (grouping) is lower
- GPU required, high compute cost
- Performance degrades significantly on Loghub-2.0 (paper explicitly states this)
- Best for: when per-line template accuracy matters more than grouping
