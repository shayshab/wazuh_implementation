## Wazuh Implementation Guide: Beginner to Advanced

[Prefer modular docs? See docs index](./docs/README.md)

- This end-to-end guide takes you from planning to production-grade Wazuh (SIEM/XDR) with HA, tuning, detection engineering, cloud/Kubernetes integrations, RBAC, cost control, and runbooks.
- Audience: security engineers, SREs, platform teams, and MSPs.
- Core components: `Wazuh Server (Manager)`, `Wazuh Indexer` (OpenSearch-based), `Wazuh Dashboard`, and `Wazuh Agents`.

---

### Table of Contents
- [1. Architecture Overview](#1-architecture-overview)
- [2. Planning, Sizing, and Requirements](#2-planning-sizing-and-requirements)
- [3. Quick Lab (All-in-One) for Learning](#3-quick-lab-all-in-one-for-learning)
- [4. Production HA Reference Architecture](#4-production-ha-reference-architecture)
- [5. Secure Certificates and TLS Everywhere](#5-secure-certificates-and-tls-everywhere)
- [6. Agent Rollout at Scale (Windows/Linux/macOS/K8s/Cloud)](#6-agent-rollout-at-scale-windowslinuxmacosk8scloud)
- [7. Core Security Use Cases](#7-core-security-use-cases)
- [8. Detection Engineering: Decoders, Rules, MITRE Mapping](#8-detection-engineering-decoders-rules-mitre-mapping)
- [9. Cloud and SaaS Integrations (AWS/Azure/GCP/Okta/O365)](#9-cloud-and-saas-integrations-awsazuregcpoktao365)
- [10. Kubernetes and Containers Visibility](#10-kubernetes-and-containers-visibility)
- [11. Data Lifecycle, ILM, and Cost Control](#11-data-lifecycle-ilm-and-cost-control)
- [12. Performance and Scalability Tuning](#12-performance-and-scalability-tuning)
- [13. Monitoring, Health Checks, and Troubleshooting](#13-monitoring-health-checks-and-troubleshooting)
- [14. Security Hardening and RBAC/SSO](#14-security-hardening-and-rbacsso)
- [15. SOC Workflows and Incident Runbooks](#15-soc-workflows-and-incident-runbooks)
- [16. Real-World Implementation Projects (Step-by-Step)](#16-real-world-implementation-projects-step-by-step)
- [17. Validation Checklists](#17-validation-checklists)
- [18. Capacity Planning Cheat Sheets](#18-capacity-planning-cheat-sheets)
- [19. FAQs and Common Pitfalls](#19-faqs-and-common-pitfalls)
- [20. Next Steps and References](#20-next-steps-and-references)

---

## 1. Architecture Overview

- **Wazuh Server (Manager)**: decoders, rules, FIM, SCA, vulnerability detection, active response, API; orchestrates agents.
- **Wazuh Indexer**: scalable datastore for `alerts-*` and `wazuh-monitoring-*`; supports sharding, replication, ILM, snapshots.
- **Wazuh Dashboard**: visualizations, RBAC, saved objects, cases.
- **Agents**: Windows/Linux/macOS/Kubernetes nodes; modules include FIM, SCA, Syscollector, Vulnerability detector, Log collection, Command monitoring, Active response.
- **Data Flow**: Agents → TLS → Manager → Shipper → Indexer → Dashboard.

Notes:
- Keep manager duties CPU-bound; indexer duties IO/heap-bound.
- Use TLS (mTLS) for agent-manager, manager-indexer, and dashboard-indexer.

---

## 2. Planning, Sizing, and Requirements

### 2.1 Define Goals and Scope
- Compliance (PCI, CIS hardening, ISO27001), Threat Detection (MITRE ATT&CK), Cloud Audit (AWS/Azure/GCP), Container visibility, Incident Response.
- Prioritize initial modules (e.g., FIM + SCA + Vulnerability + basic logs) to control noise.

### 2.2 Estimate EPS, Data Volume, and Retention
- Events per second (EPS) is the headline driver for CPU, RAM, and storage.
- Rough daily data: `daily_gb ≈ EPS × 86,400 × avg_event_size_bytes / 1e9`.
- Typical `avg_event_size_bytes`: 600–1600 bytes (varies by modules and fields).

Example estimations:
- Small: ≤2k EPS, 30–60 days retention → 1–3 Indexer nodes; 1–2 Managers.
- Medium: 2k–10k EPS → 3–5 Indexer nodes; 2 Managers in A/P; mandatory ILM.
- Large: >10k EPS → 5+ Indexer nodes; 2–3 Managers; hot/warm/cold tiers + snapshots.

### 2.3 Network, Ports, and Security
- Agent enrollment/auth: `1514/udp` or `1515/tcp` (varies), agent TCP/TLS commonly `1514/tcp`.
- Manager API: `55000/tcp`.
- Indexer: `9200/tcp` (HTTPs), `9300/tcp` (cluster).
- Dashboard: `5601/tcp` (HTTPs).
- Restrict access with firewalls and SGs; prefer private networks and bastions.

### 2.4 Storage and ILM Strategy
- Fast SSD/NVMe for Indexer hot data; warm/cold on cheaper storage.
- Snapshots to S3-compatible storage (minIO, AWS S3, GCS, Azure Blob).
- Plan ILM at design time (see section 11).

### 2.5 High Availability (HA)
- Indexer: 3+ nodes for quorum; replica shards≥1; dedicated master-eligible nodes when large.
- Manager: Wazuh cluster (master/worker); or A/P behind VIP; ensure shared registration data.
- Dashboard: 1–2 nodes behind LB; stateless, rely on Indexer.

---

## 3. Quick Lab (All-in-One) for Learning

A fast way to try Wazuh on a single Ubuntu VM.

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install curl unzip debconf-utils jq
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

- Access dashboard at: `https://<host>:5601`.
- Enroll a Linux agent:

```bash
curl -sO https://packages.wazuh.com/4.x/apt/wazuh-agent_4.x_amd64.deb
sudo WAZUH_MANAGER="<server_ip>" dpkg -i wazuh-agent_4.x_amd64.deb
sudo systemctl enable --now wazuh-agent
```

Validation:
- Dashboard → Agents shows status `Active`.
- Discover shows data in `alerts-*`.

---

## 4. Production HA Reference Architecture

### 4.1 Topology
- 3× Indexer nodes (quorum, replica=1+)
- 2× Manager nodes (cluster or A/P)
- 1–2× Dashboard nodes behind LB
- Optional: separate master-eligible indexer nodes; hot/warm/cold data nodes

### 4.2 Installation Outline
- Use OS-native packages; automate with Ansible/Terraform.
- Version pinning and change control.

### 4.3 Health Checks
```bash
curl -k https://indexer-1:9200/_cluster/health?pretty
/var/ossec/bin/wazuh-control status
```

---

## 5. Secure Certificates and TLS Everywhere

Generate a CA and node certificates for indexers, managers, and dashboards.

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-certs-tool.sh
chmod +x wazuh-certs-tool.sh
./wazuh-certs-tool.sh -A
```

- Distribute certs securely to each node.
- Enforce HTTPS for Indexer and Dashboard; mTLS for agents.
- Rotate certificates annually or on compromise.

---

## 6. Agent Rollout at Scale (Windows/Linux/macOS/K8s/Cloud)

### 6.1 Windows (GPO)
- Deploy MSI via GPO with transforms for `ADDRESS`, `AGENT_NAME`, `GROUPS`.
- Pair with Sysmon for rich telemetry (process, DNS, network events).

### 6.2 Linux (Ansible)
- Use an Ansible role to install and enroll agents via `authd`.
- Assign agents to groups by role (e.g., `linux-web`, `linux-db`).

### 6.3 macOS (MDM)
- Deploy notarized pkg via Jamf/Intune; grant PPPC for FIM paths.

### 6.4 Kubernetes (DaemonSet)
- Deploy as a DaemonSet; use node name as agent name; enable required modules only.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: wazuh-agent
spec:
  selector:
    matchLabels:
      app: wazuh-agent
  template:
    metadata:
      labels:
        app: wazuh-agent
    spec:
      containers:
      - name: agent
        image: wazuh/wazuh-agent:4.x
        env:
        - name: WAZUH_MANAGER
          value: "wazuh-manager-svc"
        - name: WAZUH_AGENT_GROUP
          value: "k8s-nodes"
        securityContext:
          runAsUser: 0
```

### 6.5 Cloud VMs (cloud-init)
- Install and enroll on first boot via cloud-init scripts.

---

## 7. Core Security Use Cases

### 7.1 File Integrity Monitoring (FIM)
- Linux: monitor `/etc`, `/usr/bin`, application directories.
- Windows: monitor `C:\\Windows\\System32`, application directories, registry keys.
- Tune by excluding high-churn paths.

### 7.2 Vulnerability Detection
- Enable `syscollector` inventory; schedule daily/weekly scans; suppress noisy packages via exceptions.

### 7.3 Security Configuration Assessment (SCA)
- Apply CIS benchmarks per OS; tailor policies for golden images; track pass/fail deltas over time.

### 7.4 Log Collection
- Ingest syslog from network devices; Windows Event channels; app logs (Nginx/Apache/DB).
- Tag sources with groups and fields for routing and dashboards.

### 7.5 Threat Intelligence
- Integrate sources (MISP/OTX/AbuseIPDB); enrich with tags and confidence scores; route high-confidence alerts.

### 7.6 Active Response
- Implement guarded containment (firewall rules, kill process, service disable).
- Require allowlists, approvals, and rollback plans.

---

## 8. Detection Engineering: Decoders, Rules, MITRE Mapping

### 8.1 Custom Decoder Example

```xml
<decoder name="myapp-decoder">
  <prematch>myapp:</prematch>
  <regex offset="after_prematch">client=(\S+) action=(\S+) status=(\d+)</regex>
  <order>client,action,status</order>
</decoder>
```

### 8.2 Rule Example with MITRE Tag

```xml
<rule id="100201" level="8">
  <if_decoder_name>myapp-decoder</if_decoder_name>
  <field name="action">login</field>
  <field name="status">\b(401|403)\b</field>
  <description>MyApp failed login</description>
  <mitre>
    <id>T1110</id>
  </mitre>
  <group>authentication, myapp,</group>
</rule>
```

- Reload rules: `sudo systemctl restart wazuh-manager`.
- Validate with sample logs and confirm in Dashboard.

### 8.3 Engineering Tips
- Start with lenient decoders; iterate to reduce false positives.
- Map to MITRE techniques; maintain a detection catalog.

---

## 9. Cloud and SaaS Integrations (AWS/Azure/GCP/Okta/O365)

### 9.1 AWS
- Collect CloudTrail, VPC Flow, ALB/ELB, WAF, GuardDuty.
- Use Firehose → Indexer or Wazuh modules; secure endpoints with IAM.

### 9.2 Azure
- Event Hub → collector; integrate Azure AD sign-ins, Activity Logs, Microsoft Defender alerts.

### 9.3 GCP
- Pub/Sub → collector; Admin activity, VPC Flow, GKE audit logs.

### 9.4 SaaS
- Okta/O365/GitHub via APIs or webhooks → syslog/HTTP; apply rate limits and retries.

---

## 10. Kubernetes and Containers Visibility

- DaemonSet agents on nodes (host-level FIM/auditd, process/network telemetry).
- K8s API server audit logs via webhook to syslog/HTTP collector.
- Combine with Falco/OPA Gatekeeper for runtime/policy alerts; ingest into Wazuh with decoders.

---

## 11. Data Lifecycle, ILM, and Cost Control

### 11.1 Index Strategy
- Separate indices: `alerts-*` and `wazuh-monitoring-*`.
- Control cardinality: avoid high-card fields and unneeded geoip; review mappings.

### 11.2 ILM Example (conceptual)
- Hot: 7–14d (fast SSD) → Warm: 30–90d → Cold: 180–365d → Delete.
- Snapshots nightly to object storage; lifecycle to Glacier/Archive tiers.

### 11.3 Snapshot Policy
- Daily full or daily incrementals with weekly full.
- Test restore regularly; document RTO/RPO.

---

## 12. Performance and Scalability Tuning

### 12.1 Indexer
- JVM heap ≈ 50% RAM (max 31GB); monitor GC.
- `indices.memory.index_buffer_size: 10%` as a baseline.
- Right-size shards; avoid oversharding.

### 12.2 Manager
- Tune analysis threads and queues; disable unused modules.
- Scale out workers if EPS grows.

### 12.3 Agents
- Narrow FIM paths; batch intervals; avoid deep recursion on large trees.

---

## 13. Monitoring, Health Checks, and Troubleshooting

### 13.1 Quick Health
```bash
/var/ossec/bin/wazuh-control status
curl -k https://indexer:9200/_cluster/health?pretty
```

### 13.2 Logs
- Manager: `/var/ossec/logs/ossec.log`
- Agent: `/var/ossec/logs/ossec.log` (Linux), Windows path varies

### 13.3 Common Issues
- Certificate mismatches; enrollment denied; indexer red/yellow (disk watermarks, shards); noisy decoders.

---

## 14. Security Hardening and RBAC/SSO

- Enforce TLS/mTLS across all components.
- Restrict `authd` to enrollment windows & IP allowlists; rotate agent keys.
- Dashboard RBAC: least privilege bound to index patterns and saved objects.
- SSO: integrate SAML/LDAP/OIDC; require MFA.
- Harden nodes per CIS; isolate networks; use bastion hosts.

---

## 15. SOC Workflows and Incident Runbooks

### 15.1 Triage
- Prioritize by severity, MITRE tag, asset criticality, and threat intel confidence.

### 15.2 Enrichment
- IP/domain/file reputation; process lineage; user and asset context.

### 15.3 Response
- Contain host (AR script), disable account, block IP, open ticket.
- Post-incident: refine rules, tune noise, update dashboards.

---

## 16. Real-World Implementation Projects (Step-by-Step)

### 16.1 Small SOC for 200-Employee Company
- Goals: Windows login monitoring, ransomware detection, FIM/SCA.
- Steps:
  1. All-in-one install; create initial ILM (hot 14d → warm 60d → cold 180d → delete).
  2. Enroll Windows via GPO; deploy Sysmon with curated config.
  3. Enable SCA CIS policies; review weekly.
  4. Build dashboards: Failed Logons, New Admins, Sysmon Process Tree.
  5. Active response to block known ransomware hashes and suspicious scripts.
- Outcome: Lowered MTTR ~40%, audit-ready CIS posture.

### 16.2 Multi-Tenant MSP Deployment
- Goals: Isolate 15 customers with RBAC and per-tenant retention.
- Steps:
  1. 3-node Indexer, 2 Managers (A/P), 2 Dashboards behind LB.
  2. Tenant separation via agent groups, index patterns, and roles.
  3. Per-tenant ILM and snapshots to dedicated S3 buckets.
  4. Alert routing via webhooks to each tenant’s ticketing system.
- Outcome: Scalable isolation with unified operations.

### 16.3 Cloud-First Startup (AWS + Kubernetes)
- Goals: CloudTrail, GuardDuty, EKS audit, node telemetry.
- Steps:
  1. K8s DaemonSet for nodes; API server audit webhook → syslog/HTTP collector.
  2. AWS Firehose → Indexer for CloudTrail/GuardDuty.
  3. Custom rules for risky `AssumeRole`, privileged `kubectl exec`, image pulls from unknown registries.
  4. ILM (hot 7d, warm 30d); snapshots to S3; dashboards for IAM risk.
- Outcome: Visibility across cloud and cluster; detections on lateral movement.

### 16.4 Compliance Project (PCI-DSS)
- Goals: FIM, SCA, centralized logging, evidence retention.
- Steps:
  1. Tight FIM on CDE; SCA mapped to PCI; exceptions documented.
  2. Retain `alerts-*` 1 year across hot/warm/cold; immutable snapshots.
  3. Dashboards aligned to PCI sections; scheduled PDF exports.
  4. Runbooks for evidence collection and auditor requests.
- Outcome: Successful audit with defensible artefacts.

---

## 17. Validation Checklists

Architecture
- TLS, HA, RBAC, and ILM designed and documented
- Capacity sizing justified; cost model approved

Install and Access
- Nodes healthy; certs valid; API/Dashboard reachable
- Backup and snapshot jobs enabled and tested

Agents
- ≥95% connected; grouped by role; baseline modules enabled
- Enrollment secured; keys rotated on schedule

Detections
- FIM/SCA/Vulnerability alerts are relevant; FP rate acceptable
- Custom decoders/rules tested with sample events

Cloud/K8s
- All feeds ingested; audit logs complete; Falco/Gatekeeper integrated if used

Performance
- Cluster green; CPU <60% during peak; heap stable; disk below watermarks

Security
- Restricted enroll; bastion-only admin; SSO with MFA; least privilege

Operations
- Runbooks published; alert routing to ticketing/Slack tested; on-call rotations set

---

## 18. Capacity Planning Cheat Sheets

### 18.1 EPS to Storage (Back-of-the-Envelope)
- `daily_gb ≈ EPS × 86,400 × event_size_bytes / 1e9`
- Example: `3k EPS × 86400 × 900B ≈ 233 GB/day` (pre-ILM/snapshots)

### 18.2 Indexer Sizing Hints
- Start with 3 nodes for quorum; keep shard size 20–50 GB for hot indices.
- Heap: 50% RAM (≤31GB); prefer more nodes over very large heaps.

### 18.3 Manager Sizing Hints
- Scale with EPS and enabled modules; consider separate worker nodes at high EPS.

---

## 19. FAQs and Common Pitfalls

- Agents connect but no alerts: check rules/decoders and module enablement; verify indexer ingestion.
- Indexer yellow/red: disk watermark exceeded, shards unassigned, cert trust issues.
- Noisy FIM: constrain paths; exclude temp/cache; set reasonable intervals.
- Enrollment failures: IP/port/firewall, CA trust, incorrect manager address, clock skew.
- RBAC gaps: verify index patterns and saved object permissions.

---

## 20. Next Steps and References

- Start with the lab for hands-on familiarity.
- Draft your production architecture and ILM policy.
- Pilot with 10–20 endpoints; tune noise; scale in waves.
- Add one cloud integration at a time; validate detections.
- Automate with Ansible/Terraform/Helm; implement backups early.

References
- Wazuh documentation: `https://documentation.wazuh.com`
- OpenSearch/Index lifecycle management concepts: `https://opensearch.org`

---

Prepared for implementation teams to adapt and extend. Keep this living document updated as your environment evolves.
