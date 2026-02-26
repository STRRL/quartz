# LLMParser - LLM-based Log Parsing

- Venue: ICSE'24
- Paper: https://arxiv.org/abs/2404.18001
- Status: Read

## Takeaway

- Fine-tuning small LLMs on log parsing: Flan-T5-base (240M params) gets PA 0.96, on par with LLaMA-7B
- Few-shot fine-tuning >> in-context learning for log parsing (PA 0.96 vs 0.46)
- Bigger model does not always mean better: Flan-T5-base matches LLaMA-7B at a fraction of the cost
- Pre-training on logs from other systems can actually hurt (LLaMA accuracy dropped 55% with cross-system pre-training)
- Data diversity matters more than data quantity — more examples dont always help if they are similar
- Fine-tuning only takes 1-5 min on A100, so the overhead is small
- For LAPP: fine-tuning approach is an alternative to ICL, but requires labeled data per system. ICL (LILAC/LogBatcher style) is more practical for zero-setup scenarios
