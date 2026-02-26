# iKnow: Intent-Guided Chatbot for Cloud Operations with RAG

- Venue: ASE'25 - ACM SIGSOFT Distinguished Paper Award
- Authors: Junjie Huang, Zhihan Jiang et al. (CUHK + Huawei Cloud)
- Code: https://github.com/Jun-jie-Huang/iKnow
- PDF: https://jun-jie-huang.github.io/assets/papers/ase25_iknow.pdf
- Status: Read

## Takeaway

- Deployed RAG chatbot at Huawei Cloud ("CloudA") for 6 months, serving thousands of engineers
- Studied 2000 real queries across 3 ops teams, found 5 intent types:
  - Symptom analysis (40.6%, most common): "this error happened, what does it mean?"
  - Multi-facet summary: "give me an overview of X"
  - Terminology explanation: "what is X?"
  - Fact verification: "is X true?"
  - Operation guidance: "how do I do X?"
- 6 root causes for chatbot failures: incomplete query (32%), lacking knowledge (27%), out-of-scope (10%), invalid query (9%), retrieval issues, generation issues
- iKnow pipeline: intent detection (prototypical network) -> intent-specific query rewriting -> retrieval -> missing knowledge detection -> LLM generation
- Key insight: different intents need different query rewriting strategies. Symptom queries need context enrichment, terminology queries need expansion
- Results: accuracy from 65.8% to 81.3%, end-to-end latency 22.5s
- Tech stack: LangChain + FAISS + BGE-M3 embedding + bce-reranker + Qwen2.5-32B
- For LAPP "chat with logs" feature: intent detection is important — users ask very different types of questions about logs, and the system should handle each differently
