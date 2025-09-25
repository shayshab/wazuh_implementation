## 01 — Basics & Concepts

- What is Wazuh: components, data flow, agents, modules.
- Concepts: decoders, rules, SCA, FIM, Vulnerability, Active Response, Indexer, Dashboard.

### Components
- Manager: analysis, orchestration, API
- Indexer: storage, search, ILM
- Dashboard: UI, RBAC
- Agent: OS telemetry (Windows/Linux/macOS/K8s)

### Data Flow
Agent → TLS → Manager → Shipper → Indexer → Dashboard

### Installation Modes
- All-in-one for labs
- Three-tier (HA) for production

See also: `../Wazuh_Implementation_Guide.md#1-architecture-overview`
