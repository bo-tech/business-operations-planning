# Public Release

Prepare the existing `b-ops` infrastructure code for public release.


## Success Criteria

- [ ] `business-operations` repository contains sanitized, usable code
- [ ] No secrets, internal IPs, or private domain names in public repo
- [ ] `b-ops` can consume/overlay `business-operations` for private operations
- [ ] `demo-ops` demonstrates how to use `business-operations`
- [ ] Documentation sufficient for others to understand the structure


## Repository Structure

- `business-operations` — Public base with generic infrastructure patterns
- `b-ops` — Private overlay for production use
- `demo-ops` — Public overlay example for testing and demonstration


## Approach

1. `business-operations` becomes the public template — sanitized code from `b-ops`
2. `b-ops` stays private — continues running actual infrastructure
3. `demo-ops` shows how to use `business-operations` as an overlay example
   - Public repository demonstrating the consumption pattern
   - Can be used for test deployments
4. Create configuration layers: generic base in public repo, site-specific
   overrides private
5. For SOPS: provide example files showing structure without actual secrets


## Sanitization

How values are handled across repositories:

- `business-operations` contains only example values and variables
- `demo-ops` fills variables with real, intentionally public values
- `b-ops` continues to hold private/internal data

**What changes in `business-operations`:**

1. **SOPS encrypted secrets** — Remove entirely

2. **Internal values** — Replace with variables, provide RFC-compliant examples:
   - Email addresses (e.g., `user@example.com`)
   - Domain names (e.g., `example.com`, `k8s.example.com`)
   - IP addresses (e.g., `192.0.2.x` per RFC 5737)
   - LDAP base DN (e.g., `dc=example,dc=com`)

**Safe to publish as-is:**

- 21 Architecture Decision Records (ADRs)
- Kubernetes manifest structure
- Infrastructure patterns (Ansible, NixOS, k0s)
- Documentation structure
