# LogParser-LLM - Hybrid Prefix Tree + LLM

- Venue: KDD'24
- Authors: Alibaba + Harvard
- Paper: https://arxiv.org/abs/2408.13727
- Status: Read

## Takeaway

- Architecture very close to what LAPP wants: prefix tree (like Drain) handles most logs, LLM only called on new/unknown patterns
- On 14 datasets avg 3.6M logs each, LLM was only called 272.5 times on average — tree does the heavy lifting
- No labeled data needed, no hyperparameter tuning — just plug and go
- Uses NER-style prompting: instead of "extract the template", asks LLM to identify named entities (IPs, paths, numbers) as variables
- Introduced "Granularity Distance" metric: sometimes both the parser and ground truth are correct but at different detail levels (e.g., should a path be a variable or part of the template?)
- Lets users calibrate granularity with just 32 labeled examples via ICL
- Results: 90.6% F1 GA, 81.1% PA on LogPub benchmark
- For LAPP: this is the closest existing architecture to our control-plane/data-plane design. Prefix tree = data plane, LLM = control plane. Worth studying their tree implementation closely.
