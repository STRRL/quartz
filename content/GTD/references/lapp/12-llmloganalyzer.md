# LLMLogAnalyzer - Clustering-Based Log Analysis Chatbot

- Paper: https://arxiv.org/abs/2510.24031
- Status: Read

## Takeaway

- A chatbot for log analysis, not a log parser — different goal from LAPP Phase 1
- Pipeline: router (classify user query) -> log recognizer (identify log type) -> log parser (cluster + extract) -> LLM generates answer
- Solves LLM context window problem by clustering logs first, then feeding summaries instead of raw logs
- Tested on 4 log domains: beats ChatGPT/ChatPDF/NotebookLM by 39-68% on summarization/pattern/anomaly tasks
- 93% less output variability (more consistent answers)
- For LAPP: the "cluster first, then summarize" idea is relevant for Phase 2 reporting, but the chatbot interface itself is not our focus
