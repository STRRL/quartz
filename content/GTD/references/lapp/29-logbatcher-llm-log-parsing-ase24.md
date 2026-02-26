# LogBatcher: Demonstration-Free Log Parsing with LLMs

- Paper: https://arxiv.org/abs/2406.06156
- Published: ASE 2024
- Code: https://github.com/LogIntelligence/LogBatcher
- Status: Unread

## Takeaway

- AI-powered Drain: the best LLM log parser alongside LILAC, but unsupervised and demo-free
- TF-IDF + DBSCAN for clustering, better than embeddings — logs are structurally similar by nature, token-level diff matters more
- GA 0.972, MLA 0.895 on 16 datasets, outperforms LILAC
- Batch-based: groups similar logs, parses representative ones, broadcasts results
- No labeled demos needed, no hyperparameter tuning
