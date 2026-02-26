# All you need is Wide Events, not "Metrics, Logs and Traces"

- Author: Ivan Burmistrov (ex-Meta)
- Link: https://isburmistrov.substack.com/p/all-you-need-is-wide-events-not-metrics
- Status: Read

## Takeaway

- Wide Event = one fat JSON per event, cram in every field you might need later
- Traces, logs, metrics are all just special cases of Wide Events (add traceId = trace, add log_message = log, periodic system snapshot = metric)
- Meta Scuba: columnar store, no pre-aggregation, keep raw events, query-time aggregation, brute-force scan
- Core workflow: pick a metric, group by any field, drill down — slice and dice until you find the problem
- Dynamic sampling controls cost: normal traffic 1/1000, errors 1/1, UI auto-upscales so users dont notice
- Storage cost is the main tradeoff: raw events vs pre-aggregated metrics differ by orders of magnitude, sampling is the bridge
- External alternatives: Honeycomb (closest to Scuba philosophy), Axiom, ClickHouse
