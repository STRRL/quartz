# LAPP Design Notes

Aggregated highlights from paper reviews. Will be reorganized after all papers are reviewed.

## From [01-lilac](01-lilac-log-parsing-llm-cache.md)

- Control plane (LLM) / data plane (cache tree) split
- Prefix tree cache with wildcard matching for fast log-to-template lookup
- Template merging: LCS similarity > 0.8 -> generalize differing tokens to `<*>`
- Self-validation: LLM template must match its own source log or get rejected
- Hierarchical candidate sampling + Jaccard similarity for ICL demonstration selection
- Breakpoint resume for large datasets

## From [02-ibm-label-broadcasting](02-ibm-label-broadcasting-log-analytics.md)

- Drain clustering first, then LLM on representatives only, broadcast labels back
- Three label types: Golden Signal (error/availability/latency/saturation/info), Fault Category (app/network/IO), NER (host/session/error code)
- Small fine-tuned BERT (BERTOps) runs on CPU, no GPU needed
- 3.2% edge case: template variables carry diagnostic cues lost in templatization
- Report types:
  - Summary: rarest lines first
  - Temporal Trend: golden signal over time -> when did it break?
  - Causal Graph: Granger causality on cluster time series -> how did fault propagate?
  - Diagnosis Report: fault-containing time windows only, searchable by entity
  - Workflow: Summary -> Temporal -> Causal -> Diagnosis

## From [03-loghub-2.0](03-loghub-2.0-issta24.md)

- Dataset
- future is semantic + global statistical hybrid model
- feasible efficient approach: Drain, IPLoM, LogCluster / LogSig / LFA, UniParser / LogPPT

## From [04-sok-llm-log-parsing](04-sok-llm-log-parsing-2025.md)

- Recommended standard metrics for log parsing: GA, PA, FTA, NED
- Only two LLM parsers clearly lead: LogBatcher, LILAC

## From [05-l4-llm-training-log-diagnosis](05-l4-llm-training-log-diagnosis-fse25.md)

- Three log analysis patterns (all useful for LAPP Phase 2):
  - Cross-job: this run broke but last run was fine — diff them, new stuff is likely the cause
  - Spatial: most machines log the same thing, the odd one out is probably broken
  - Temporal: find which phase or iteration things went sideways
- Domain-specific to LLM training, but the three patterns generalize well

## From [06-wide-events-scuba](06-wide-events-scuba-burmistrov.md)

- Scuba product UX is a future reference: pick log source, set filters, aggregate, render chart
- Nice-to-have feature for LAPP, not core

## From [07-observability-2.0](07-observability-2.0-honeycomb.md)

- Nothing actionable for LAPP

## From [08-drain3](08-drain3-logpai.md)

- Drain is an important algorithm for LAPP, likely need to implement it from scratch

## From [11-logparser-llm](11-logparser-llm-kdd24.md)

- Essentially a better Drain, LLM-enhanced Drain
- Prefix tree handles bulk, LLM only on new patterns (272 calls for 3.6M logs)

## From [12-llmloganalyzer](12-llmloganalyzer.md)

- Reference for future "chat with logs" feature
- Key idea: cluster logs first to fit context window, then LLM summarizes/answers over clusters instead of raw logs

## From [14-iknow-rag-chatbot](14-iknow-rag-chatbot-cloud-ops-ase25.md)

- 5 user intent types for ops QA: symptom analysis (40.6%), multi-facet summary, terminology explanation, fact verification, operation guidance
- 6 failure modes: incomplete query (32%), lacking knowledge (27%), out-of-scope (10%), invalid query (9%), retrieval issues, generation issues
- Different intents need different query rewriting — important for LAPP "chat with logs" feature

## From [26-logimprover](26-logimprover-logging-quality-ase25.md) and [27-sclogger](27-sclogger-logging-generation-fse24.md)

- Future feature: if LAPP can access source code, auto-improve/inject logs to help profiling and exploration

## From [29-logbatcher](29-logbatcher-llm-log-parsing-ase24.md)

- AI-powered Drain, top parser alongside LILAC
- TF-IDF + DBSCAN for clustering, beats embeddings — logs are structurally similar, token-level diff matters more

## From [22-k8sgpt](22-k8sgpt.md)

- k8sgpt log analysis is just regex "error|exception|fail" on last 100 lines — this is the gap LAPP fills
- Plugin/analyzer architecture worth referencing for LAPP extensibility

## From [23-awesome-llm-aiops](23-awesome-llm-aiops.md)

- Industry trend: RCA becoming agentic (RCAgent, FLASH, LLexus) — agent with tools does diagnosis. LAPP Phase 2 direction
- Log Anomaly Detection papers in the list are a goldmine for Phase 2, revisit later
- AIOpsLab (MLSys'25) benchmark could be useful for LAPP evaluation
