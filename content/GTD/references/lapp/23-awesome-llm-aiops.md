# awesome-LLM-AIOps - Curated Paper List

- Project: https://github.com/Jun-jie-Huang/awesome-LLM-AIOps
- Papers: 78
- Status: Read

## Takeaway

- Maintained by the CUHK group (same authors as L4, LogPilot, iKnow) — authoritative list
- Organized by: Incident Management (RCA, mitigation, postmortem), Log Analysis (parsing, anomaly detection, log generation), Infrastructure Management
- Notable papers we havent covered that could be relevant:
  - AIOpsLab (MLSys'25): benchmark framework for evaluating AI agents in cloud ops — could be useful for LAPP evaluation
  - RCAgent (CIKM'24): tool-augmented LLM agent for cloud RCA — the "agent with tools" pattern is what LAPP Phase 2 could evolve into
  - D-Bot (VLDB'24): database diagnosis with Tree-of-Thought prompting — structured reasoning for diagnosis
  - COCA (ICSE'25): RCA using code knowledge — ties back to LogImprover/SCLogger idea of connecting logs to source code
- Log Parsing section lists: LILAC, LogBatcher, DivLog, LogParser-LLM — confirms our reading list covers the key ones
- Log Anomaly Detection section is a potential goldmine for LAPP Phase 2, but not priority now
- For LAPP: good reference to check periodically for new papers. The "agent for RCA" trend (RCAgent, FLASH, LLexus) suggests LAPP Phase 2 could be an agentic system that uses tools (log parser, metrics fetcher, trace viewer) to do RCA
