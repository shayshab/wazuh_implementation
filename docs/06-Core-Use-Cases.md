## 06 — Core Security Use Cases

### File Integrity Monitoring (FIM)
- Linux: `/etc`, `/usr/bin`, app dirs
- Windows: `C:\\Windows\\System32`, app dirs, registry hives
- Exclude high-churn/temp paths; set reasonable scan intervals

### Vulnerability Detection
- Enable syscollector; schedule daily/weekly
- Suppress noisy packages; track remediation SLAs

### Security Configuration Assessment (SCA)
- Apply CIS benchmarks per OS; tailor for golden images
- Trend pass/fail over time; export evidence for audits

### Log Collection
- Syslog for network devices; Windows Event channels; app logs (Nginx/Apache/DB)
- Normalize and tag for dashboards and routing

### Threat Intelligence
- Integrate MISP/OTX/AbuseIPDB; add confidence and source tags
- Route high confidence to pager; medium to triage queues

### Active Response
- Guard rails: allowlists, approvals, rollback
- Actions: block IP, disable account, kill process, quarantine host
