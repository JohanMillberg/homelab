# Graph Report - .  (2026-06-23)

## Corpus Check
- 4 files · ~101,837 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 244 nodes · 313 edges · 24 communities (23 shown, 1 thin omitted)
- Extraction: 89% EXTRACTED · 11% INFERRED · 0% AMBIGUOUS · INFERRED: 34 edges (avg confidence: 0.81)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_App Scaffolding Skill|App Scaffolding Skill]]
- [[_COMMUNITY_Infrastructure Controllers|Infrastructure Controllers]]
- [[_COMMUNITY_Hermes Autonomy CronJobs|Hermes Autonomy CronJobs]]
- [[_COMMUNITY_App Kustomizations|App Kustomizations]]
- [[_COMMUNITY_Flux Helm CRD Types|Flux Helm CRD Types]]
- [[_COMMUNITY_K3s Cluster Dashboards|K3s Cluster Dashboards]]
- [[_COMMUNITY_Autonomy ADR Decisions|Autonomy ADR Decisions]]
- [[_COMMUNITY_Hermes K8s Resources|Hermes K8s Resources]]
- [[_COMMUNITY_Act Runner CI Pipeline|Act Runner CI Pipeline]]
- [[_COMMUNITY_Hermes Work Queue|Hermes Work Queue]]
- [[_COMMUNITY_Hermes Autonomy Config|Hermes Autonomy Config]]
- [[_COMMUNITY_Forgejo Deployment|Forgejo Deployment]]
- [[_COMMUNITY_Grafana Deployment|Grafana Deployment]]
- [[_COMMUNITY_Hermes External Integrations|Hermes External Integrations]]
- [[_COMMUNITY_Forgejo Monitoring|Forgejo Monitoring]]
- [[_COMMUNITY_n8n Monitoring|n8n Monitoring]]
- [[_COMMUNITY_Pihole Monitoring|Pihole Monitoring]]
- [[_COMMUNITY_Cycling Training App|Cycling Training App]]
- [[_COMMUNITY_Grafana Dashboard Skill|Grafana Dashboard Skill]]
- [[_COMMUNITY_Hermes Autonomy Prompts|Hermes Autonomy Prompts]]
- [[_COMMUNITY_Plan Assistant Frontend|Plan Assistant Frontend]]
- [[_COMMUNITY_Act Runner K8s Resources|Act Runner K8s Resources]]
- [[_COMMUNITY_Planning Skills|Planning Skills]]
- [[_COMMUNITY_Hermes Config|Hermes Config]]

## God Nodes (most connected - your core abstractions)
1. `Hermes Deployment` - 16 edges
2. `Hermes Autonomy Queue CronJob` - 14 edges
3. `ADR 0001: GitHub work queue for Hermes autonomy` - 11 edges
4. `Hermes Autonomy Review Poll CronJob` - 10 edges
5. `add-app skill` - 9 edges
6. `deploy-raw-app skill` - 9 edges
7. `seal-secret skill` - 8 edges
8. `Hermes Kustomization` - 8 edges
9. `debug-live-image skill` - 7 edges
10. `Namespace hermes` - 7 edges

## Surprising Connections (you probably didn't know these)
- `In-cluster service URL vs .lan DNS rule` --semantically_similar_to--> `.lan DNS via pihole at 192.168.1.205`  [INFERRED] [semantically similar]
  .claude/skills/debug-live-image/SKILL.md → CLAUDE.md
- `DATABASE_URL ConfigMap+Secret envFrom override pattern` --applies--> `cycling-training-app Deployment`  [INFERRED]
  .claude/skills/deploy-raw-app/SKILL.md → apps/cycling-training-app/deployment.yaml
- `seal-secret skill` --constrains--> `Sealed Secret Change Boundary`  [INFERRED]
  .claude/skills/seal-secret/SKILL.md → CONTEXT.md
- `OpenCode Go Account (OPENCODE_GO_API_KEY)` --references--> `cycling-training-app-config ConfigMap`  [INFERRED]
  CONTEXT.md → apps/cycling-training-app/configmap.yaml
- `add-app eval: deploy Vaultwarden via OCI Helm chart` --references--> `Kubernetes cluster`  [INFERRED]
  .claude/skills/add-app/evals/evals.json → apps/grafana/dashboards/kubernetes-overview.json

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Hermes Autonomy Shared Secrets and Config** — hermes_autonomy_cronjobs_hermesconfig, hermes_autonomy_cronjobs_hermesautonomyconfig, hermes_autonomy_cronjobs_hermesdiscordsecret, hermes_autonomy_cronjobs_hermesmodelsecret, hermes_autonomy_cronjobs_hermesgithubappsecret [EXTRACTED 1.00]
- **Act Runner DinD CI Pipeline** — act_runner_deployment_actrunnerdeployment, act_runner_deployment_dindsidecar, docker_daemon_configmap_dockerdaemonconfig [EXTRACTED 1.00]
- **Hermes Deployment Init Container Chain** — hermes_deployment_fixpermissionsinit, hermes_autonomy_cronjobs_installghclinit, hermes_deployment_configurehermesinit [EXTRACTED 0.95]

## Communities (24 total, 1 thin omitted)

### Community 0 - "App Scaffolding Skill"
Cohesion: 0.08
Nodes (37): Register app in apps/kustomization.yaml, HelmRelease scaffold (apiVersion v2), HelmRepository scaffold (HTTP vs OCI), add-app skill, Values ConfigMap (values.yaml key), AGENTS.md Codex startup guide, .lan DNS via pihole at 192.168.1.205, CLAUDE.md canonical project guide (+29 more)

### Community 1 - "Infrastructure Controllers"
Cohesion: 0.11
Nodes (26): infrastructure/configs Kustomization (empty), forgejo HelmRepository (OCI), grafana HelmRepository (HTTP), infrastructure/controllers Kustomization, longhorn HelmRelease (chart 1.8.x), flux-system GitRepository, flux-system Kustomization (clusters/main), Flux apps Kustomization (+18 more)

### Community 2 - "Hermes Autonomy CronJobs"
Cohesion: 0.14
Nodes (25): Hermes Autonomy Prompts ConfigMap, Hermes Autonomy Scripts ConfigMap, Autonomy Trigger Fast-Poll Script, Autonomy Trigger Queue Script, Hermes GitHub Tools ConfigMap, Hermes Agent Image (forgejo.lan/kvarnberg/hermes-agent), Hermes Autonomy Config ConfigMap, Hermes Autonomy ServiceAccount (+17 more)

### Community 3 - "App Kustomizations"
Cohesion: 0.17
Nodes (16): Apps Root Kustomization, HelmRelease n8n (chart 2.x), n8n Kustomization, Namespace n8n, ConfigMap n8n-values, Pihole .lan DNS resolver (192.168.1.205), HelmRelease pihole (chart 2.x, mojo2600), Pihole Kustomization (+8 more)

### Community 4 - "Flux Helm CRD Types"
Cohesion: 0.13
Nodes (16): Flux helm.toolkit.fluxcd.io/v2 HelmRelease, Flux source.toolkit.fluxcd.io/v1 HelmRepository, longhorn HelmRepository, charts.longhorn.io (HTTP Helm chart source), mojo2600 HelmRepository, mojo2600.github.io/pihole-kubernetes (HTTP Helm chart source), OCI HelmRepository type, n8n HelmRepository (OCI) (+8 more)

### Community 5 - "K3s Cluster Dashboards"
Cohesion: 0.14
Nodes (15): k3s cluster nodes, Kubernetes cluster, Node Exporter Full Grafana Dashboard, Prometheus datasource (k3s-nodes dashboard), k3s nodes Quick CPU / Mem / Disk panel group, k3s nodes Pressure (PSI) panel group, Kubernetes Dashboard (Overview), Prometheus datasource (kubernetes-overview dashboard) (+7 more)

### Community 6 - "Autonomy ADR Decisions"
Cohesion: 0.26
Nodes (12): Command safety boundary (scoped auto-approval), Fast preflight poll CronJob (5m, GitHub-only), GitHub as work source of truth, ADR 0001: GitHub work queue for Hermes autonomy, GitOps change boundary (user keeps merge authority), Hermes GitHub App (credential identity), Hermes issue label set (hermes-ready..hermes-done), kvarnberg-labs/hermes-work-queue repository (+4 more)

### Community 7 - "Hermes K8s Resources"
Cohesion: 0.33
Nodes (11): Hermes Kustomization, Namespace hermes, NetworkPolicy hermes-default-deny, NetworkPolicy hermes-allow-required-egress, PVC hermes-data (longhorn 10Gi), Role hermes-autonomy-lease, RoleBinding hermes-autonomy-lease, Role hermes-sandbox-admin (+3 more)

### Community 8 - "Act Runner CI Pipeline"
Cohesion: 0.22
Nodes (10): Act Runner Deployment, Act Runner Secret (Registration Token), Docker-in-Docker Sidecar, Docker Host TCP Config, Forgejo LAN Host Alias (192.168.1.205), Forgejo Runner Container, Act Runner Register InitContainer, Act Runner Data PVC (+2 more)

### Community 9 - "Hermes Work Queue"
Cohesion: 0.29
Nodes (8): Fast Poll Trigger (5-min CronJob), GitHub Work Queue (kvarnberg-labs/hermes-work-queue), Hermes CLI Work Trigger (hourly CronJob), Hermes Work Item, Implementation Lock (Kubernetes Lease), Implementation Pull Request, Merge Authority (user-retained), Shared Hermes Home (PVC /opt/data)

### Community 10 - "Hermes Autonomy Config"
Cohesion: 0.29
Nodes (8): hermes-autonomy-config ConfigMap (GitHub App + schedules + disk thresholds), hermes-autonomy-scripts ConfigMap (autonomy_trigger.py), Hermes disk policy (7 GiB warn / 9 GiB block workspace usage), LeaseLock (coordination.k8s.io Lease via SA token), autonomy_trigger.py (queue/fast-poll modes, GitHub App auth, lease lock), Hermes GitHub App (kvarnberg-callisto[bot], org kvarnberg-labs), hermes-github-tools ConfigMap (install-gh, github-app-token, gh wrapper), kvarnberg-labs/hermes-work-queue (GitHub work queue, source of truth)

### Community 11 - "Forgejo Deployment"
Cohesion: 0.48
Nodes (7): forgejo HelmRelease (chart 16.x), forgejo-values ConfigMap (gitea, longhorn, ingress forgejo.lan), forgejo Kustomization, forgejo-sealed-secret (forgejo-secret-values), forgejo Namespace, Forgejo metrics endpoint (forgejo-http.forgejo.svc.cluster.local:3000/metrics), Prometheus scrape job: forgejo

### Community 12 - "Grafana Deployment"
Cohesion: 0.43
Nodes (7): Grafana dashboard sidecar (label grafana_dashboard=1), grafana dashboards Kustomization (configMapGenerator), grafana HelmRelease (chart 10.x), grafana-values ConfigMap (longhorn, Prometheus datasource, dashboard sidecar, ingress grafana.lan), Grafana Prometheus datasource (prometheus-server.prometheus.svc), grafana Kustomization, grafana Namespace

### Community 13 - "Hermes External Integrations"
Cohesion: 0.40
Nodes (6): Discord Bot remote chat surface, Hermes Gateway, Hermes GitHub App, Hermes Secret (sealed credentials), Hermes GitHub autonomy domain model (CONTEXT.md), OpenCode Go Account (OPENCODE_GO_API_KEY)

### Community 14 - "Forgejo Monitoring"
Cohesion: 0.40
Nodes (5): Forgejo (git service), Forgejo Grafana Dashboard, Prometheus datasource (forgejo dashboard), Forgejo stats panels (Repositories, Users, Goroutines, Memory), Forgejo timeseries panels (CPU Usage, GC Duration)

### Community 15 - "n8n Monitoring"
Cohesion: 0.40
Nodes (5): n8n (workflow automation service), n8n Grafana Dashboard, Prometheus datasource (n8n dashboard), n8n Container Restarts panel, n8n Pod CPU and Memory panels

### Community 16 - "Pihole Monitoring"
Cohesion: 0.40
Nodes (5): Pi-hole (DNS ad-blocker service), Pi-hole Exporter Grafana Dashboard, Prometheus datasource (pihole dashboard), Pi-hole Top Queries panel group, Pi-hole Status / Domains Blocked panels

### Community 17 - "Cycling Training App"
Cohesion: 0.80
Nodes (5): cycling-training-app Ingress (cycling.kvarnberg.labs → :8000), cycling-training-app Kustomization, cycling-training-app Namespace, cycling-training-app-data PVC (10Gi RWO), cycling-training-app Service (ClusterIP :8000)

### Community 18 - "Grafana Dashboard Skill"
Cohesion: 0.40
Nodes (5): manage-grafana-dashboards skill, ${datasource} template variable convention, grafanactl CLI (Grafana 12+), Deprecated panel type migration (singlestat/graph -> stat/timeseries), Grafana sidecar dashboard provisioning (grafana_dashboard label)

### Community 19 - "Hermes Autonomy Prompts"
Cohesion: 0.50
Nodes (4): hermes-autonomy-prompts ConfigMap (queue-scanner.md, review-response.md), Hermes queue-scanner prompt (hourly GitHub work trigger), Hermes review-response prompt (fast review/comment trigger), hermes-sandbox Namespace

### Community 20 - "Plan Assistant Frontend"
Cohesion: 0.50
Nodes (4): Deployment plan-assistant-frontend, ConfigMap frontend-nginx-config, ImagePolicy frontend-image-policy, ImageRepository plan-assistant-frontend

### Community 21 - "Act Runner K8s Resources"
Cohesion: 0.67
Nodes (3): act-runner kustomization, act-runner Namespace, act-runner-data PVC

### Community 22 - "Planning Skills"
Cohesion: 0.67
Nodes (3): grill-me skill, Deep module design principle, write-a-prd skill

## Knowledge Gaps
- **68 isolated node(s):** `HelmRepository scaffold (HTTP vs OCI)`, `HelmRelease error reference table`, `Pod failure classification (ImagePullBackOff/CrashLoop/Init/OOMKilled/Pending)`, `grill-me skill`, `grafanactl CLI (Grafana 12+)` (+63 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **1 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `prometheus-values ConfigMap` connect `Infrastructure Controllers` to `App Kustomizations`, `Forgejo Deployment`?**
  _High betweenness centrality (0.059) - this node is a cross-community bridge._
- **Why does `Prometheus scrape job: pihole` connect `App Kustomizations` to `Infrastructure Controllers`?**
  _High betweenness centrality (0.042) - this node is a cross-community bridge._
- **Why does `Apps Root Kustomization` connect `App Kustomizations` to `Hermes K8s Resources`?**
  _High betweenness centrality (0.038) - this node is a cross-community bridge._
- **Are the 2 inferred relationships involving `Hermes Deployment` (e.g. with `Forgejo LAN Host Alias (192.168.1.205)` and `Hermes Autonomy Queue CronJob`) actually correct?**
  _`Hermes Deployment` has 2 INFERRED edges - model-reasoned connections that need verification._
- **What connects `HelmRepository scaffold (HTTP vs OCI)`, `HelmRelease error reference table`, `Pod failure classification (ImagePullBackOff/CrashLoop/Init/OOMKilled/Pending)` to the rest of the system?**
  _86 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `App Scaffolding Skill` be split into smaller, more focused modules?**
  _Cohesion score 0.08108108108108109 - nodes in this community are weakly interconnected._
- **Should `Infrastructure Controllers` be split into smaller, more focused modules?**
  _Cohesion score 0.11384615384615385 - nodes in this community are weakly interconnected._