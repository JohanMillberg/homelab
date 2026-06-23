# Graph Report - .  (2026-06-23)

## Corpus Check
- 103 files · ~102,000 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 219 nodes · 287 edges · 19 communities
- Extraction: 88% EXTRACTED · 12% INFERRED · 0% AMBIGUOUS · INFERRED: 34 edges (avg confidence: 0.83)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Ops Skills Toolkit|Ops Skills Toolkit]]
- [[_COMMUNITY_Act Runner & Raw Deployments|Act Runner & Raw Deployments]]
- [[_COMMUNITY_Plan Assistant Frontend|Plan Assistant Frontend]]
- [[_COMMUNITY_Forgejo Git Hosting|Forgejo Git Hosting]]
- [[_COMMUNITY_Hermes Identity & Integrations|Hermes Identity & Integrations]]
- [[_COMMUNITY_Hermes Work Queue Context|Hermes Work Queue Context]]
- [[_COMMUNITY_Cycling App Infrastructure|Cycling App Infrastructure]]
- [[_COMMUNITY_Forgejo, Pihole & Prometheus Scrape|Forgejo, Pihole & Prometheus Scrape]]
- [[_COMMUNITY_Grafana Core|Grafana Core]]
- [[_COMMUNITY_Hermes Autonomy Engine|Hermes Autonomy Engine]]
- [[_COMMUNITY_Hermes Runtime & Security|Hermes Runtime & Security]]
- [[_COMMUNITY_Grafana Dashboard Skill|Grafana Dashboard Skill]]
- [[_COMMUNITY_Flux & Infrastructure Controllers|Flux & Infrastructure Controllers]]
- [[_COMMUNITY_ADR GitHub Work Queue Design|ADR: GitHub Work Queue Design]]
- [[_COMMUNITY_Flux Helm CRD URLs|Flux Helm CRD URLs]]
- [[_COMMUNITY_Infrastructure Dashboards & Evals|Infrastructure Dashboards & Evals]]
- [[_COMMUNITY_Forgejo Monitoring Dashboard|Forgejo Monitoring Dashboard]]
- [[_COMMUNITY_N8N Monitoring Dashboard|N8N Monitoring Dashboard]]
- [[_COMMUNITY_Pihole Monitoring Dashboard|Pihole Monitoring Dashboard]]

## God Nodes (most connected - your core abstractions)
1. `ADR 0001: GitHub work queue for Hermes autonomy` - 11 edges
2. `add-app skill` - 9 edges
3. `deploy-raw-app skill` - 9 edges
4. `seal-secret skill` - 8 edges
5. `Hermes Kustomization` - 8 edges
6. `debug-live-image skill` - 7 edges
7. `hermes-autonomy-queue CronJob (7 * * * *, suspended)` - 7 edges
8. `autonomy_trigger.py (queue/fast-poll modes, GitHub App auth, lease lock)` - 7 edges
9. `Namespace hermes` - 7 edges
10. `Kubernetes cluster` - 7 edges

## Surprising Connections (you probably didn't know these)
- `In-cluster service URL vs .lan DNS rule` --semantically_similar_to--> `.lan DNS via pihole at 192.168.1.205`  [INFERRED] [semantically similar]
  .claude/skills/debug-live-image/SKILL.md → CLAUDE.md
- `DATABASE_URL ConfigMap+Secret envFrom override pattern` --applies--> `cycling-training-app Deployment`  [INFERRED]
  .claude/skills/deploy-raw-app/SKILL.md → apps/cycling-training-app/deployment.yaml
- `seal-secret skill` --constrains--> `Sealed Secret Change Boundary`  [INFERRED]
  .claude/skills/seal-secret/SKILL.md → CONTEXT.md
- `OpenCode Go Account (OPENCODE_GO_API_KEY)` --references--> `cycling-training-app-config ConfigMap`  [INFERRED]
  CONTEXT.md → apps/cycling-training-app/configmap.yaml
- `forgejo HelmRelease (chart 16.x)` --served_by--> `Forgejo metrics endpoint (forgejo-http.forgejo.svc.cluster.local:3000/metrics)`  [INFERRED]
  apps/forgejo/forgejo-helmrelease.yaml → /Users/johan/Documents/WS/homelab/apps/prometheus/prometheus-values.yaml

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **.lan DNS does not resolve in-cluster; use *.svc.cluster.local (CLAUDE.md convention, debug-live-image rule, act-runner init fix, eval-1 root cause)** —  [EXTRACTED 0.90]
- **Forgejo private-registry deploy flow (build/push image, registry pull SealedSecret, imagePullSecrets in Deployment)** —  [INFERRED 0.85]
- **Hermes autonomous coding loop (CLI Work Trigger + Fast Poll Trigger coordinate via Implementation Lock over the GitHub Work Queue)** —  [EXTRACTED 0.90]
- **Grafana observability stack (HelmRelease → values → Prometheus datasource + dashboard sidecar discovering dashboard ConfigMaps)** — grafana_grafana_helmrelease_helmrelease, grafana_grafana_values_configmap, grafana_grafana_values_prometheus_datasource, grafana_dashboards_dashboard_sidecar, grafana_dashboards_kustomization_kustomization [EXTRACTED 0.90]
- **Hermes autonomy queue workflow (CronJob → config → prompt → trigger script → GitHub work queue under lease)** — hermes_autonomy_cronjobs_queue, hermes_autonomy_configmap_config, hermes_autonomy_prompts_queue_scanner, hermes_autonomy_scripts_trigger, hermes_work_queue_repo, hermes_autonomy_scripts_lease_lock [EXTRACTED 0.95]
- **Hermes fast review-poll workflow (CronJob → review-response prompt → trigger script fast-poll mode)** — hermes_autonomy_cronjobs_review_poll, hermes_autonomy_prompts_review_response, hermes_autonomy_scripts_trigger, hermes_github_app [EXTRACTED 0.90]
- **Hermes autonomy identity grants cross-namespace access** — hermes_serviceaccount_autonomy, hermes_sandbox_rbac_rolebinding, hermes_rbac_lease_rolebinding [EXTRACTED 0.90]
- **Plan-assistant Flux image automation flow** — plan_assistant_backend_deployment, plan_assistant_image_policy_backend_repo, plan_assistant_image_policy_backend_policy [EXTRACTED 0.90]
- **plan-assistant three-tier app: frontend SPA + backend API + postgres, exposed via Ingress path routing** — plan_assistant_ingress, plan_assistant_frontend_service, plan_assistant_backend_service, plan_assistant_postgres_service, plan_assistant_postgres_statefulset [EXTRACTED 0.90]
- **Flux reconciliation chain: GitRepository → flux-system Kustomization → infrastructure-controllers → infrastructure-configs → apps** — flux_system_gitrepository, flux_system_kustomization, main_infrastructure_controllers_kustomization, main_infrastructure_configs_kustomization, main_apps_kustomization [EXTRACTED 0.95]
- **Hermes autonomy: two CronJob triggers sharing a Lease lock and the Hermes PVC, driven by the GitHub work queue** — adr_0001_hourly_cronjob_scanner, adr_0001_fast_preflight_cronjob, adr_0001_implementation_lock_lease, adr_0001_shared_hermes_pvc_workspaces, adr_0001_hermes_work_queue_repo [EXTRACTED 0.95]

## Communities (19 total, 0 thin omitted)

### Community 0 - "Ops Skills Toolkit"
Cohesion: 0.11
Nodes (25): add-app skill, HelmRepository scaffold (HTTP vs OCI), HelmRelease scaffold (apiVersion v2), Values ConfigMap (values.yaml key), Register app in apps/kustomization.yaml, debug-flux skill, KUBECONFIG=/home/johan/.kube/config convention, HelmRelease error reference table (+17 more)

### Community 2 - "Act Runner & Raw Deployments"
Cohesion: 0.14
Nodes (21): debug-live-image skill, In-cluster service URL vs .lan DNS rule, Pod failure classification (ImagePullBackOff/CrashLoop/Init/OOMKilled/Pending), Eval 1 — act-runner stuck Init:0/1, deploy-raw-app skill, Forgejo registry push (forgejo.lan, --output type=docker), forgejo-registry-secret imagePullSecret, Alembic base migration from-scratch rule (+13 more)

### Community 18 - "Plan Assistant Frontend"
Cohesion: 0.67
Nodes (3): grill-me skill, write-a-prd skill, Deep module design principle

### Community 16 - "Forgejo Git Hosting"
Cohesion: 0.40
Nodes (5): manage-grafana-dashboards skill, grafanactl CLI (Grafana 12+), Grafana sidecar dashboard provisioning (grafana_dashboard label), Deprecated panel type migration (singlestat/graph -> stat/timeseries), ${datasource} template variable convention

### Community 11 - "Hermes Identity & Integrations"
Cohesion: 0.40
Nodes (6): Hermes GitHub autonomy domain model (CONTEXT.md), Hermes Gateway, Discord Bot remote chat surface, Hermes Secret (sealed credentials), Hermes GitHub App, OpenCode Go Account (OPENCODE_GO_API_KEY)

### Community 9 - "Hermes Work Queue Context"
Cohesion: 0.29
Nodes (8): GitHub Work Queue (kvarnberg-labs/hermes-work-queue), Hermes Work Item, Hermes CLI Work Trigger (hourly CronJob), Fast Poll Trigger (5-min CronJob), Implementation Lock (Kubernetes Lease), Shared Hermes Home (PVC /opt/data), Implementation Pull Request, Merge Authority (user-retained)

### Community 15 - "Cycling App Infrastructure"
Cohesion: 0.80
Nodes (5): cycling-training-app Ingress (cycling.kvarnberg.labs → :8000), cycling-training-app Kustomization, cycling-training-app Namespace, cycling-training-app-data PVC (10Gi RWO), cycling-training-app Service (ClusterIP :8000)

### Community 7 - "Forgejo, Pihole & Prometheus Scrape"
Cohesion: 0.20
Nodes (15): forgejo HelmRelease (chart 16.x), forgejo-values ConfigMap (gitea, longhorn, ingress forgejo.lan), forgejo Kustomization, forgejo-sealed-secret (forgejo-secret-values), forgejo Namespace, Pihole Kustomization, Namespace pihole, HelmRelease pihole (chart 2.x, mojo2600) (+7 more)

### Community 10 - "Grafana Core"
Cohesion: 0.43
Nodes (7): grafana dashboards Kustomization (configMapGenerator), Grafana dashboard sidecar (label grafana_dashboard=1), grafana HelmRelease (chart 10.x), grafana-values ConfigMap (longhorn, Prometheus datasource, dashboard sidecar, ingress grafana.lan), Grafana Prometheus datasource (prometheus-server.prometheus.svc), grafana Kustomization, grafana Namespace

### Community 4 - "Hermes Autonomy Engine"
Cohesion: 0.20
Nodes (17): hermes-autonomy-config ConfigMap (GitHub App + schedules + disk thresholds), Hermes GitHub App (kvarnberg-callisto[bot], org kvarnberg-labs), hermes-autonomy-queue CronJob (7 * * * *, suspended), hermes-autonomy-review-poll CronJob (*/5 * * * *, suspended), hermes-autonomy-prompts ConfigMap (queue-scanner.md, review-response.md), Hermes queue-scanner prompt (hourly GitHub work trigger), Hermes review-response prompt (fast review/comment trigger), kvarnberg-labs/hermes-work-queue (GitHub work queue, source of truth) (+9 more)

### Community 3 - "Hermes Runtime & Security"
Cohesion: 0.16
Nodes (20): Role hermes-sandbox-admin, RoleBinding hermes-sandbox-admin, Hermes Kustomization, Namespace hermes, NetworkPolicy hermes-default-deny, NetworkPolicy hermes-allow-required-egress, PVC hermes-data (longhorn 10Gi), Role hermes-autonomy-lease (+12 more)

### Community 17 - "Grafana Dashboard Skill"
Cohesion: 0.50
Nodes (4): Deployment plan-assistant-frontend, ConfigMap frontend-nginx-config, ImageRepository plan-assistant-frontend, ImagePolicy frontend-image-policy

### Community 1 - "Flux & Infrastructure Controllers"
Cohesion: 0.12
Nodes (25): plan-assistant ImageUpdateAutomation, plan-assistant Ingress, plan-assistant Kustomization, plan-assistant Namespace, plan-assistant-postgres StatefulSet, plan-assistant-secret (SealedSecret), Forgejo Registry Pull Secret instructions, plan-assistant Services (+17 more)

### Community 8 - "ADR: GitHub Work Queue Design"
Cohesion: 0.26
Nodes (12): ADR 0001: GitHub work queue for Hermes autonomy, kvarnberg-labs/hermes-work-queue repository, Hermes GitHub App (credential identity), Hourly queue-scanner CronJob (45m deadline), Fast preflight poll CronJob (5m, GitHub-only), Global implementation lock (Kubernetes Lease), Command safety boundary (scoped auto-approval), Shared Hermes PVC workspaces (/opt/data, 10GiB) (+4 more)

### Community 5 - "Flux Helm CRD URLs"
Cohesion: 0.13
Nodes (16): longhorn HelmRepository, charts.longhorn.io (HTTP Helm chart source), Flux source.toolkit.fluxcd.io/v1 HelmRepository, mojo2600 HelmRepository, mojo2600.github.io/pihole-kubernetes (HTTP Helm chart source), n8n HelmRepository (OCI), OCI HelmRepository type, 8gears.container-registry.com/library (OCI Helm chart source) (+8 more)

### Community 6 - "Infrastructure Dashboards & Evals"
Cohesion: 0.14
Nodes (15): add-app eval: deploy Vaultwarden via OCI Helm chart, debug-live-image eval: act-runner pod stuck in Init:0/1, debug-live-image eval: plan-assistant-backend CrashLoopBackOff, debug-live-image eval: frontend pod 0/1 Ready readiness probe failure, grill-me eval: add Nextcloud to homelab, Node Exporter Full Grafana Dashboard, Prometheus datasource (k3s-nodes dashboard), k3s nodes Quick CPU / Mem / Disk panel group (+7 more)

### Community 12 - "Forgejo Monitoring Dashboard"
Cohesion: 0.40
Nodes (5): Forgejo Grafana Dashboard, Prometheus datasource (forgejo dashboard), Forgejo stats panels (Repositories, Users, Goroutines, Memory), Forgejo timeseries panels (CPU Usage, GC Duration), Forgejo (git service)

### Community 13 - "N8N Monitoring Dashboard"
Cohesion: 0.40
Nodes (5): n8n Grafana Dashboard, Prometheus datasource (n8n dashboard), n8n Pod CPU and Memory panels, n8n Container Restarts panel, n8n (workflow automation service)

### Community 14 - "Pihole Monitoring Dashboard"
Cohesion: 0.40
Nodes (5): Pi-hole Exporter Grafana Dashboard, Prometheus datasource (pihole dashboard), Pi-hole Status / Domains Blocked panels, Pi-hole Top Queries panel group, Pi-hole (DNS ad-blocker service)

## Knowledge Gaps
- **57 isolated node(s):** `HelmRepository scaffold (HTTP vs OCI)`, `HelmRelease error reference table`, `Pod failure classification (ImagePullBackOff/CrashLoop/Init/OOMKilled/Pending)`, `grill-me skill`, `grafanactl CLI (Grafana 12+)` (+52 more)
  These have ≤1 connection - possible missing edges or undocumented components.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `prometheus-values ConfigMap` connect `Forgejo, Pihole & Prometheus Scrape` to `Flux & Infrastructure Controllers`?**
  _High betweenness centrality (0.096) - this node is a cross-community bridge._
- **What connects `HelmRepository scaffold (HTTP vs OCI)`, `HelmRelease error reference table`, `Pod failure classification (ImagePullBackOff/CrashLoop/Init/OOMKilled/Pending)` to the rest of the system?**
  _70 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Ops Skills Toolkit` be split into smaller, more focused modules?**
  _Cohesion score 0.11 - nodes in this community are weakly interconnected._
- **Should `Act Runner & Raw Deployments` be split into smaller, more focused modules?**
  _Cohesion score 0.14285714285714285 - nodes in this community are weakly interconnected._
- **Should `Flux & Infrastructure Controllers` be split into smaller, more focused modules?**
  _Cohesion score 0.11666666666666667 - nodes in this community are weakly interconnected._
- **Should `Flux Helm CRD URLs` be split into smaller, more focused modules?**
  _Cohesion score 0.13333333333333333 - nodes in this community are weakly interconnected._
- **Should `Infrastructure Dashboards & Evals` be split into smaller, more focused modules?**
  _Cohesion score 0.14285714285714285 - nodes in this community are weakly interconnected._