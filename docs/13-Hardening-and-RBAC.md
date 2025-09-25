## 13 — Security Hardening & RBAC/SSO

### TLS Everywhere
- mTLS for agents; HTTPS for API/Indexer/Dashboard; rotate certs

### Enrollment Security
- Restrict `authd` by IP/time; rotate keys; disable legacy non-TLS

### RBAC & SSO
- Roles bound to index patterns and saved objects
- Integrate SAML/LDAP/OIDC; require MFA

### System Hardening
- CIS baselines on nodes; firewall segmentation; bastion-only admin
