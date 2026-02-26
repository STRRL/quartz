# Observability 2.0 - Honeycomb / Charity Majors

- Link: https://www.honeycomb.io/blog/time-to-version-observability-signs-point-to-yes
- Status: Read

## Takeaway

- O11y 1.0 = metrics + logs + traces as three separate pillars, each with its own tool
- O11y 2.0 = unify everything into wide structured log events, one data source to rule them all
- Shift from write-time aggregation (pre-compute metrics) to read-time aggregation (keep raw events, query on the fly)
- Honeycomb claims logs are cheaper than metrics when you count total cost (no need to maintain 3 separate systems)
- But dodges the hard question: at massive scale, raw events are way more data than pre-aggregated metrics
- In practice, sampling + columnar storage + cheaper hardware makes it work for most companies
- Industry trend is moving this direction, but metrics still win for ultra-fast pre-computed queries
