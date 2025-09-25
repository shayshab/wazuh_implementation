## 15 — Real-World Projects (Step-by-Step)

### Small SOC (200 Employees)
1. All-in-one install; ILM hot 14d → warm 60d → cold 180d
2. Enroll Windows via GPO; deploy Sysmon
3. Enable SCA; build Failed Logons/Sysmon dashboards
4. Active response for ransomware indicators

### MSP Multi-Tenant
1. 3 Indexers, 2 Managers A/P, 2 Dashboards behind LB
2. Separate via groups, index patterns, roles
3. Per-tenant ILM and S3 snapshots
4. Webhook routing to tenant ticketing

### Cloud-First (AWS + K8s)
1. DaemonSet on nodes; API audit webhook → collector
2. Firehose → Indexer: CloudTrail/GuardDuty
3. Rules for risky AssumeRole, privileged exec, unknown images
4. ILM hot 7d/warm 30d; dashboards for IAM risk

### PCI-DSS Compliance
1. Tight FIM on CDE; SCA mapped to PCI; exceptions documented
2. Retain 1y with snapshots; immutable backups
3. PCI dashboards; scheduled reports
4. Auditor runbooks
