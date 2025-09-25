## 08 — Cloud & SaaS Integrations

### AWS
- CloudTrail, VPC Flow, ALB/ELB, WAF, GuardDuty via Firehose → Indexer or Wazuh modules
- Secure with IAM and VPC endpoints; partition by account

### Azure
- Azure AD sign-ins, Activity Logs, Defender alerts via Event Hub → collector
- Service principals and RBAC with least privilege

### GCP
- Admin Activity, VPC Flow, GKE audit via Pub/Sub → collector
- Service accounts with minimal scopes

### SaaS (Okta/O365/GitHub)
- API collectors or webhooks → syslog/HTTP
- Throttling/backoff and checkpointing to avoid gaps
