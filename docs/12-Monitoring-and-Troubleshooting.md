## 12 — Monitoring & Troubleshooting

### Health Checks
```bash
/var/ossec/bin/wazuh-control status
curl -k https://indexer:9200/_cluster/health?pretty
```

### Logs
- Manager: `/var/ossec/logs/ossec.log`
- Agent: `/var/ossec/logs/ossec.log`
- Dashboard/Indexer logs via systemd or container logs

### Common Issues
- Cert mismatch; enrollment denied; indexer red/yellow (disk); decoder mismatches

### Diagnostics
- Capture problematic events and test against decoders/rules
- Use index stats and shard allocation APIs for storage issues
