## 09 — Kubernetes & Containers

### Node Visibility
- DaemonSet agents for host FIM, process, network, auditd

### Cluster Audit
- API server audit webhook → syslog/HTTP collector
- Detect privileged pods, exec/portforward, image pulls from unknown registries

### Complementary Tools
- Falco for runtime; Gatekeeper for policy
- Ingest Falco events via decoders/rules

### Minimal DaemonSet
See `../examples/k8s/wazuh-agent-daemonset.yaml`
