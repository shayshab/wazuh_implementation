## 11 — Performance & Scalability

### Indexer
- JVM heap ≈ 50% RAM (max 31GB); monitor GC
- Right-size shards (20–50 GB hot); avoid oversharding
- SSD/NVMe for hot data; tune index buffer size (~10%)

### Manager
- Tune analysis threads/queues; disable unused modules
- Scale out workers for high EPS

### Agents
- Scope FIM paths; adjust intervals; avoid deep recursion

### Benchmarking
- Measure ingestion latency, CPU, heap, disk watermarks under load
- Load-test before each major change
