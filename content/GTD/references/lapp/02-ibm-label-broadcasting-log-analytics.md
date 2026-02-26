# IBM: Scalable and Efficient Large-Scale Log Analysis with LLMs

- Paper: https://arxiv.org/html/2511.14803v1
- Code: https://github.com/Log-Analyzer/LogAn
- Status: Read

## Takeaway

- Core pattern: Drain clustering first, then LLM classification on representatives only - this is the right approach for LAPP too
- The three label types are highly reusable for LAPP:
  - Golden Signal Classification (error / availability / latency / saturation / information)
  - Fault Category Prediction (application / network / I/O / etc.)
  - Named Entity Recognition (host, session ID, error code, etc.)
- Label Broadcasting = the same "control plane / data plane" split as LILAC's cache, just at a different granularity
- 3.2% edge case where template variables carry diagnostic cues - LAPP needs to handle this
- Engineering-oriented: runs on CPU (BERTOps, small fine-tuned BERT, not GPT-class), deployed across 70 IBM products for 15 months
- Report generation - important for LAPP product design:
  - Summary Report: representative lines sorted by rarity (rarest = most important), filterable by label
  - Temporal Trend: golden signal counts over time -> answers "when did it start breaking?"
  - Causal Graph: Granger causality on cluster time series -> answers "how did the fault propagate?"
  - Diagnosis Report: only show time windows containing faults, chronological, searchable by entity -> answers "what exactly happened at that time?"
  - Workflow: Summary (overview) -> Temporal Trend (locate time) -> Causal Graph (understand causality) -> Diagnosis Report (deep dive)
- 425K lines -> 74 representative lines (99.9% reduction) in real case study

<!--
## Details

(architecture notes, full analysis, etc. moved here for reference)
-->
