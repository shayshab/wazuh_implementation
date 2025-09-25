## 10 — ILM & Cost Control

### Index Strategy
- Separate `alerts-*` and `wazuh-monitoring-*`
- Avoid high-cardinality fields; review mappings

### ILM Policy
- Hot 7–14d → Warm 30–90d → Cold 180–365d → Delete
- See example: `../examples/ilm/alerts-ilm.json`

### Snapshots
- Nightly to S3-compatible storage; lifecycle to archive tiers
- Regular restore tests; document RPO/RTO

### Cost Levers
- Reduce noisy modules; sampling where acceptable
- Right-size shards; prefer more small nodes vs massive heaps
