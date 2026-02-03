(milestone-0-sanitization)=
# Sanitization Details

How values are handled across repositories.


## Approach

- `business-operations` contains only example values and variables
- `demo-ops` fills variables with real, intentionally public values
- `b-ops` continues to hold private/internal data


## What Changes in `business-operations`

1. **SOPS encrypted secrets** — Remove entirely

2. **Internal values** — Replace with variables, provide RFC-compliant examples:
   - Email addresses (e.g., `user@example.com`)
   - Domain names (e.g., `example.com`, `k8s.example.com`)
   - IP addresses (e.g., `192.0.2.x` per RFC 5737)
   - LDAP base DN (e.g., `dc=example,dc=com`)


## Safe to Publish As-Is

- 21 Architecture Decision Records (ADRs)
- Kubernetes manifest structure
- Infrastructure patterns (Ansible, NixOS, k0s)
- Documentation structure
