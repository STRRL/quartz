# LILAC: Log Parsing using LLMs with Adaptive Parsing Cache

- Venue: FSE'24
- Authors: Zhihan Jiang, Jinyang Liu, Zhuangbin Chen et al. (CUHK / logpai)
- Paper: https://arxiv.org/abs/2310.01796
- Project: https://github.com/logpai/LILAC
- Status: Done

## Summary
Uses LLM for log parsing with an Adaptive Parsing Cache to reduce LLM invocation cost. Directly relevant to LAPP Phase 1.

## Takeaway

### Background

- Old approach
  - Rule
    - Drain, AEL
    - Good: concise, precise, efficient at runtime
    - Bad: handcraft, human bottleneck, slow, expensive
    - Bad: need specialist domain knowledge
  - ML
    - UniParser, LogPPT
    - Good: I do not know, maybe decouple with human domain knowledge
    - Bad: require lots of labeled training data
    - Bad: limited generalization, hard to iterate
  - Direct LLM
    - no LLMs are designed for log parsing for now
    - inconsistency outputs, hallucinations
    - high computational overhead for each message, expensive

### Specific technology selection

- ICL / Context Engineering
- Dynamic Prompt Composer
  - optimize for generalization: diverse pattern pool
  - optimize for specificity query: query by "similarity"
- Match Engine / Cache
  - trie tree
  - self-refine
    - templating prune / merging
    - self validation, re-check, multi AI cross-check after the template generated, make it as invalid if it can not pass
  - match engine as data plane
