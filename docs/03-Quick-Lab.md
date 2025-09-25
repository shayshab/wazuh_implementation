## 03 — Quick Lab (All-in-One)

Prepare an Ubuntu VM and run:

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install curl unzip debconf-utils jq
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

Enroll an agent:

```bash
curl -sO https://packages.wazuh.com/4.x/apt/wazuh-agent_4.x_amd64.deb
sudo WAZUH_MANAGER="<server_ip>" dpkg -i wazuh-agent_4.x_amd64.deb
sudo systemctl enable --now wazuh-agent
```

Validate: Agents page shows `Active`; `alerts-*` has data.
