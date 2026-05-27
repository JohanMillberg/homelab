# Homelab GitOps

Use this file as the Codex startup guide for this repository. `CLAUDE.md` remains the canonical project guide; follow its repository structure, Flux ordering, conventions, sealed-secret workflow, and "Do Not" rules.

## Project Skills

Codex should load project skills through `.codex/skills/`, which is a symlink to the canonical Claude skills tree at `.claude/skills/`. Do not copy skill directories between the two locations.

Available project skills:

- `add-app` - scaffold a complete new Helm-based app.
- `deploy-raw-app` - deploy a non-Helm app with raw Kubernetes manifests and self-built images.
- `upgrade-app` - upgrade a Helm chart version or pinned image tag.
- `seal-secret` - create, update, or verify a SealedSecret.
- `debug-flux` - diagnose stuck or failing Flux/HelmRelease reconciliations.
- `debug-live-image` - debug failing pods after image or deployment updates.
- `ssh-debug` - run node-level homelab server triage.
- `manage-grafana-dashboards` - add, update, or verify Grafana dashboards.
- `grill-me` - stress-test a homelab plan or design with a structured interview.
- `write-a-prd` - create a PRD through interview, repo exploration, and issue drafting.

To verify the bridge resolves to every skill:

```bash
find -L .codex/skills -maxdepth 2 -name 'SKILL.md' | sort
```
