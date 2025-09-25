## 17 — Capacity Planning

### EPS → Storage (Estimates)
- `daily_gb ≈ EPS × 86,400 × event_size_bytes / 1e9`
- Example: `3k EPS × 86400 × 900B ≈ 233 GB/day`

### Indexer Sizing
- Start 3 nodes; shard size 20–50 GB; heap 50% RAM (≤31GB)

### Manager Sizing
- Scale with EPS and modules; consider workers at high EPS
