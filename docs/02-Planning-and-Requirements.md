## 02 — Planning & Sizing

### Goals
- Compliance, Threat Detection, Cloud Audit, Containers, IR.

### Sizing Inputs
- EPS, event size, retention, growth rate, criticality.

### Rough Sizing
- Small: ≤2k EPS; Medium: 2k–10k; Large: >10k EPS.

### Network & Ports
- Agents: 1514/1515; API: 55000; Indexer: 9200/9300; Dashboard: 5601.

### Storage & ILM
- Hot SSD; warm/cold tiers; snapshots to object storage.

### HA
- Indexer 3+ nodes; Manager cluster A/P; Dashboard behind LB.

See: `../Wazuh_Implementation_Guide.md#2-planning-sizing-and-requirements`
