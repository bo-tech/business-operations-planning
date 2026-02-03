# Milestone 0: Make Current State Public

## Goal

Prepare the existing `b-ops` infrastructure code for public release.


## Success Criteria

- [ ] `business-operations` repository contains sanitized, usable code
- [ ] No secrets, internal IPs, or private domain names in public repo
- [ ] `b-ops` can consume/overlay `business-operations` for private operations
- [ ] `demo-ops` demonstrates how to use `business-operations`
- [ ] Documentation sufficient for others to understand the structure


## Approach

1. `business-operations` becomes the public template — sanitized code from `b-ops`
2. `b-ops` stays private — continues running actual infrastructure
3. `demo-ops` shows how to use `business-operations` as an overlay example
   - Public repository demonstrating the consumption pattern
   - Can be used for test deployments
4. Create configuration layers: generic base in public repo, site-specific
   overrides private
5. For SOPS: provide example files showing structure without actual secrets


## Repository Structure

- `business-operations` — Public base with generic infrastructure patterns
- `b-ops` — Private overlay for production use
- `demo-ops` — Public overlay example for testing and demonstration


## Related

- {ref}`Sanitization details <milestone-0-sanitization>`
