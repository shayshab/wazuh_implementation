## 05 — Agent Rollout

### Windows (GPO)
- Deploy MSI with transforms: `ADDRESS`, `AGENT_NAME`, `GROUPS`.
- Pair with Sysmon for process/network telemetry.

### Linux (Ansible)
- Install via role; enroll via `authd`; assign groups.

### macOS (MDM)
- Jamf/Intune pkg; PPPC for FIM.

### Kubernetes (DaemonSet)
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

### Cloud VMs (cloud-init)
- Install and enroll on boot.
