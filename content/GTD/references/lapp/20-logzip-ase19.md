# Logzip: Log Compression via Iterative Clustering

- Venue: ASE'19
- Paper: https://arxiv.org/abs/1910.00409
- Code: https://github.com/logpai/logzip
- Status: Read

## Takeaway

- Log-specific compression: extract templates via clustering, store template + variables separately, then compress — saves ~50% over gzip
- Interesting side effect: the template extraction step is basically log parsing, just used for compression instead of analysis
- For LAPP: not directly relevant, but confirms that template extraction (parsing) is the foundation for everything — even compression benefits from it
