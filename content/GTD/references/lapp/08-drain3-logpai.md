# Drain3 - Streaming Log Template Miner

- Project: https://github.com/logpai/Drain3
- Original paper: Drain (2017, Pinjia He et al.)
- Stars: 756
- Status: Reviewed

## Takeaway

- Best traditional (non-LLM) log parser for production use: fast, streaming, no training needed
- How it works: fixed-depth tree, first splits by token count, then by leading tokens, then matches by similarity threshold
- Tokens that vary across logs get replaced with wildcards — thats your template
- Purely string-based, zero semantic understanding — cant tell "connection refused" from "file not found" as categories
- Good baseline for LAPP: run Drain first as cheap fast pass, escalate to LLM only on cache miss or low-confidence matches
- Drain3 is the maintained Python implementation by logpai, supports persistence and streaming
