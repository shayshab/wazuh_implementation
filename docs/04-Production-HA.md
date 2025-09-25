## 04 — Production HA

### Reference Topology
- 3× Indexer, 2× Manager, 1–2× Dashboard
- Optionally hot/warm/cold tiers

### Certs & TLS
```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-certs-tool.sh
chmod +x wazuh-certs-tool.sh
./wazuh-certs-tool.sh -A
```

### Health Checks
```bash
curl -k https://indexer-1:9200/_cluster/health?pretty
/var/ossec/bin/wazuh-control status
```

### ILM & Backups
- Hot 7–14d → Warm 30–90d → Cold 180–365d → Delete
- Snapshots nightly to S3-compatible storage

See: `../Wazuh_Implementation_Guide.md#4-production-ha-reference-architecture`
