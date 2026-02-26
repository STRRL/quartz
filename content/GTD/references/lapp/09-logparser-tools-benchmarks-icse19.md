# Tools and Benchmarks for Automated Log Parsing

- Venue: ICSE'19
- Authors: Jieming Zhu, Shilin He, Jinyang Liu et al. (logpai team)
- Paper: https://arxiv.org/abs/1811.03509
- Project: https://github.com/logpai/logparser
- Status: Read

## Takeaway

- The logpai benchmark paper: evaluated 13 traditional log parsers on 16 datasets
- Datasets cover distributed systems, supercomputers, OS, mobile, server apps, standalone software
- Measured accuracy, robustness, and efficiency — the standard evaluation framework everyone cites
- Drain came out on top overall (good accuracy + fast + robust across datasets)
- This is the predecessor to Loghub-2.0 (ref 03) which expanded the benchmark
- logpai/logparser repo has reference implementations of all 13 parsers — useful for comparison
- Lesson from Huawei deployment: no single parser wins everywhere, dataset characteristics matter
