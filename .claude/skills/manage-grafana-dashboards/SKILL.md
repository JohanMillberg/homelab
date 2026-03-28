---
name: manage-grafana-dashboards
description: Use when adding, updating, or verifying a Grafana dashboard in the homelab. Always use this skill when the user wants to monitor something new, add panels, import a community dashboard, fix a blank/missing dashboard, or add metrics for a newly deployed app — guides sourcing JSON, panel type vetting, quality checks, kustomization wiring, deployment, and verification via SSH.
---

# manage-grafana-dashboards

## Overview

Checklist-driven workflow for adding or updating dashboards in the homelab Grafana instance. Dashboards live as JSON files in `apps/grafana/dashboards/` and are provisioned automatically by a Grafana sidecar that watches for ConfigMaps labeled `grafana_dashboard: "1"`.

Follow every step in order — missing any step means the dashboard never appears in Grafana.

## Prerequisites — grafanactl

`grafanactl` (official Grafana CLI, requires Grafana 12+) is installed at `~/.local/bin/grafanactl` on the server and pre-configured to reach the Grafana cluster IP. Verify it works:

```bash
ssh johan@192.168.1.205 "~/.local/bin/grafanactl resources get dashboards"
```

This should list all provisioned dashboards. If it fails with a network error, the Grafana ClusterIP may have changed — re-run:

```bash
GRAFANA_IP=$(ssh johan@192.168.1.205 \
  "KUBECONFIG=/home/johan/.kube/config kubectl get svc -n grafana grafana -o jsonpath='{.spec.clusterIP}'")
ssh johan@192.168.1.205 \
  "~/.local/bin/grafanactl config set contexts.default.grafana.server http://${GRAFANA_IP}"
```

If `grafanactl` is missing entirely, install it:

```bash
ssh johan@192.168.1.205 "
  mkdir -p ~/.local/bin &&
  cd /tmp &&
  curl -sL https://github.com/grafana/grafanactl/releases/download/v0.1.9/grafanactl_Linux_x86_64.tar.gz | tar xz grafanactl &&
  mv grafanactl ~/.local/bin/ &&
  GRAFANA_IP=\$(KUBECONFIG=/home/johan/.kube/config kubectl get svc -n grafana grafana -o jsonpath='{.spec.clusterIP}') &&
  ~/.local/bin/grafanactl config set contexts.default.grafana.server http://\${GRAFANA_IP} &&
  ~/.local/bin/grafanactl config set contexts.default.grafana.org-id 1 &&
  PASS=\$(KUBECONFIG=/home/johan/.kube/config kubectl get secret -n grafana grafana-secret-values -o jsonpath='{.data.values\.yaml}' | base64 -d | grep adminPassword | awk '{print \$2}' | tr -d '\"') &&
  ~/.local/bin/grafanactl config set contexts.default.grafana.user admin &&
  ~/.local/bin/grafanactl config set contexts.default.grafana.password \"\${PASS}\" &&
  ~/.local/bin/grafanactl config use-context default
"
```

## Checklist

### Step 1 — Get the dashboard JSON

**Path A — Community dashboard (grafana.com/grafana/dashboards):**

**Always pre-check panel types before downloading.** Many popular dashboards are years old and still use deprecated `singlestat`/`graph` panels even in their "latest" revision — Grafana 12 will silently show blank panels for these. Check first:

```bash
ssh johan@192.168.1.205 "curl -s 'https://grafana.com/api/dashboards/<ID>/revisions/latest/download' | python3 -c \"
import json,sys
d=json.load(sys.stdin)
bad={'singlestat','graph','grafana-piechart-panel','table-old'}
panel_types=set()
def get_types(panels):
    for p in panels:
        panel_types.add(p.get('type'))
        if p.get('panels'): get_types(p['panels'])
get_types(d.get('panels',[]))
print(d.get('title'), 'sv:', d.get('schemaVersion'), 'bad:', bool(bad & panel_types), 'types:', sorted(panel_types))
\""
```

To vet multiple candidates at once:
```bash
ssh johan@192.168.1.205 "for id in 18283 13770 6417; do
  curl -s \"https://grafana.com/api/dashboards/\${id}/revisions/latest/download\" | python3 -c \"
import json,sys
d=json.load(sys.stdin)
bad={'singlestat','graph','grafana-piechart-panel','table-old'}
p=set(); [p.update([x.get('type')] + [y.get('type') for y in x.get('panels',[])] ) for x in d.get('panels',[])]
print('\${id}', d.get('title','?'), 'sv:', d.get('schemaVersion'), 'bad:', bool(bad & p))
\"
done"
```

Also check if there are newer revisions (some old dashboards have updated revisions years later):
```bash
ssh johan@192.168.1.205 "curl -s 'https://grafana.com/api/dashboards/<ID>/revisions' | python3 -c \"
import json,sys; items=json.load(sys.stdin).get('items',[])
[print('rev:', i.get('revision'), i.get('createdAt','')[:10]) for i in items]
\""
```

See **Known-Good Dashboard Catalog** below for pre-vetted IDs. If no modern community dashboard exists, see **Panel Type Migration**.

Download the JSON and save to `apps/grafana/dashboards/<name>.json`. Community dashboards often contain `__inputs` and `__requires` blocks that must be removed (see Step 2).

**Path B — Export from live Grafana:**

Note: `grafana.lan` does not resolve from the server (node uses 1.1.1.1, not pihole). Use the cluster IP directly:

```bash
ssh johan@192.168.1.205 "
  GRAFANA_IP=\$(KUBECONFIG=/home/johan/.kube/config kubectl get svc -n grafana grafana -o jsonpath='{.spec.clusterIP}')
  GRAFANA_PASS=\$(KUBECONFIG=/home/johan/.kube/config kubectl get secret -n grafana grafana-secret-values -o jsonpath='{.data.values\.yaml}' | base64 -d | grep adminPassword | awk '{print \$2}' | tr -d '\"')
  curl -s http://admin:\${GRAFANA_PASS}@\${GRAFANA_IP}/api/dashboards/uid/<uid> | jq '.dashboard'
"
```

Save the `.dashboard` object (not the full API response) as `apps/grafana/dashboards/<name>.json`.

---

### Known-Good Dashboard Catalog

Pre-vetted dashboards confirmed working in Grafana 12 with this homelab's Prometheus stack. Use these before searching grafana.com.

| Exporter / Use Case | gnetId | Title | sv | Notes |
|---------------------|--------|-------|----|-------|
| Kubernetes cluster (kube-state-metrics + node-exporter) | **18283** | Kubernetes Dashboard | 37 | All modern panels. Uses `${DS_SERVICEMONITOR}` — replace with `${datasource}`. Has `datasource` var. |
| Pi-hole (ekofr/pihole-exporter) | N/A | — | — | No modern community dashboard. Use Panel Type Migration guide on gnetId 10176 rev 3. |

When adding a new exporter, run the pre-check above and add confirmed results here.

---

### Step 2 — Quality checklist

Before committing, verify every item:

- `uid` field is present and is a short, meaningful slug (e.g. `"forgejo01"`)
- `uid` is unique — check no other file in `apps/grafana/dashboards/` uses the same value
- `title` is meaningful — not `"New dashboard"` or `"Copy of ..."`
- Every panel datasource references the template variable: `{"type": "prometheus", "uid": "${datasource}"}` — not a hardcoded numeric ID or string UID
- A `datasource` template variable exists in `templating.list` (type: `datasource`, query: `prometheus`)
- No panel has the placeholder title `"Panel Title"`
- No deprecated panel types: `singlestat`, `graph`, `grafana-piechart-panel`, `table-old` — Grafana 12 silently shows blank panels for these (use the pre-check command from Step 1 to confirm). `schemaVersion` ≥ 36 is a useful heuristic but not sufficient alone.
- No hardcoded pod names, node names, or IPs in PromQL — use label matchers instead
- Community JSON: remove `__inputs` and `__requires` blocks; replace numeric dummy UIDs (e.g. `000000001`) and `${DS_*}` variables with `"${datasource}"`; inject a `datasource` template variable into `templating.list` if one is not already present:
  ```json
  {"current":{},"hide":0,"includeAll":false,"label":"Data Source","multi":false,"name":"datasource","options":[],"query":"prometheus","refresh":1,"regex":"","type":"datasource"}
  ```

---

### Step 3 — Add to kustomization.yaml

Append a new entry to the `configMapGenerator` list in `apps/grafana/dashboards/kustomization.yaml`:

```yaml
  - name: grafana-dashboard-<name>
    files:
      - <name>.json
```

Where `<name>` matches the filename without `.json`. Do **not** add an `annotations:` block — the `generatorOptions.labels` block at the top of the file already applies `grafana_dashboard: "1"` to every generated ConfigMap.

Current file for reference:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: grafana

generatorOptions:
  disableNameSuffixHash: true
  labels:
    grafana_dashboard: "1"

configMapGenerator:
  - name: grafana-dashboard-k3s-nodes
    files:
      - k3s-nodes.json
  # ... existing entries ...
  - name: grafana-dashboard-<name>   # <-- add here
    files:
      - <name>.json
```

---

### Step 4 — Deploy

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile kustomization apps --with-source"
```

Wait ~30 seconds for the sidecar to pick up the new ConfigMap.

---

### Step 5 — Verify provisioning

Run all three checks via SSH:

**1. ConfigMap created:**
```bash
ssh johan@192.168.1.205 \
  "KUBECONFIG=/home/johan/.kube/config kubectl get configmap -n grafana grafana-dashboard-<name>"
```

**2. Sidecar loaded it:**
```bash
ssh johan@192.168.1.205 \
  "KUBECONFIG=/home/johan/.kube/config kubectl logs -n grafana \
  -l app.kubernetes.io/name=grafana -c grafana-sc-dashboard --tail=30"
```

Look for a line mentioning the dashboard file or ConfigMap name. Errors here indicate a malformed JSON file.

**3. Dashboard live in Grafana (via grafanactl):**
```bash
ssh johan@192.168.1.205 "~/.local/bin/grafanactl resources get dashboards/<uid>"
```

Returns a table row like `Dashboard   forgejo01   default` on success. Returns an error if not provisioned yet.

To list all currently provisioned dashboards:
```bash
ssh johan@192.168.1.205 "~/.local/bin/grafanactl resources get dashboards"
```

If the ConfigMap exists but the dashboard is not showing after a minute:
```bash
ssh johan@192.168.1.205 \
  "KUBECONFIG=/home/johan/.kube/config kubectl rollout restart deployment -n grafana grafana"
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Datasource UID is a hardcoded number (`000000001`) or string (`"prometheus"`) | Replace with `"${datasource}"` and ensure a `datasource` template variable exists in `templating.list` |
| Setting `uid: prometheus` in the Grafana datasource provisioning ConfigMap | **Do not set `uid:` in datasource provisioning** — Grafana 12 crashes with "data source not found" on startup. Leave uid unset; Grafana auto-generates it. The `${datasource}` template variable resolves dynamically. |
| Missing `uid` field | Add a short unique slug, e.g. `"uid": "forgejo01"` |
| JSON file not added to `kustomization.yaml` | The sidecar never sees the ConfigMap without a `configMapGenerator` entry |
| Duplicate `uid` across two dashboards | Grafana silently overwrites one — pick a unique uid per dashboard |
| Community JSON has `__inputs` / `__requires` blocks | Remove them and replace `${DS_*}` variables with `"${datasource}"` (see Step 2) |
| ConfigMap exists but dashboard not showing | Check sidecar logs; if stale: `kubectl rollout restart deployment -n grafana grafana` |
| Adding `annotations:` to a configMapGenerator entry | Not needed — `generatorOptions.labels` already applies the sidecar label globally |
| Picking a gnetId without vetting panel types first | Many dashboards with "latest" revisions still use `singlestat`/`graph`/`grafana-piechart-panel` — run the pre-check command in Step 1 before committing to any ID |
| Trusting `schemaVersion` as a proxy for modern panels | sv 26+ dashboards can still have `singlestat`/`graph`. Always check actual panel types, not just the schema number |

---

## Panel Type Migration

Use when no modern community dashboard exists (all revisions have deprecated panels). Write a Python conversion script, run it on the server, and pipe the output to the repo file.

**Type mapping (Grafana 8+ replacements):**

| Old type | Modern type | Notes |
|----------|-------------|-------|
| `singlestat` | `stat` | Use `reduceOptions.calcs: ["lastNotNull"]` |
| `graph` | `timeseries` | Keep `targets`; rebuild `fieldConfig`/`options` |
| `grafana-piechart-panel` | `piechart` | Built-in since Grafana 8 |
| `table-old` | `table` | Rebuild `fieldConfig` with modern `cellOptions` |
| `gauge` | `gauge` | Already modern, keep as-is |

**Conversion script template** (write to `/tmp/convert.py` on server, pipe JSON through it):

```python
import json, sys
d = json.load(sys.stdin)
DS = {"type": "prometheus", "uid": "${datasource}"}

d.pop("__inputs", None); d.pop("__requires", None)
d["uid"] = "myuid01"; d["schemaVersion"] = 39

# Fix annotations + template variable datasources
for ann in d.get("annotations", {}).get("list", []):
    if isinstance(ann.get("datasource"), str): ann["datasource"] = DS
tvars = d.get("templating", {}).get("list", [])
if not any(v.get("name") == "datasource" for v in tvars):
    tvars.insert(0, {"current": {}, "hide": 0, "includeAll": False,
        "label": "Data Source", "multi": False, "name": "datasource",
        "options": [], "query": "prometheus", "refresh": 1, "regex": "", "type": "datasource"})
for v in tvars:
    if isinstance(v.get("datasource"), str) and "DS_" in v["datasource"]:
        v["datasource"] = DS

def fix_ds(ds):
    if isinstance(ds, str) and "DS_" in ds: return DS
    if isinstance(ds, dict) and "DS_" in str(ds.get("uid", "")): return DS
    return ds

def to_stat(p):
    return {"type": "stat", "title": p.get("title", ""), "id": p["id"],
        "gridPos": p["gridPos"], "transparent": True, "datasource": DS,
        "targets": p.get("targets", []),
        "options": {"colorMode": "value", "graphMode": "area", "justifyMode": "auto",
            "orientation": "auto", "reduceOptions": {"calcs": ["lastNotNull"], "fields": "", "values": False},
            "textMode": "auto"},
        "fieldConfig": {"defaults": {"color": {"mode": "thresholds"}, "mappings": [],
            "thresholds": {"mode": "absolute", "steps": [{"color": "green", "value": None}]}},
            "overrides": []}}

def to_timeseries(p):
    return {"type": "timeseries", "title": p.get("title", ""), "id": p["id"],
        "gridPos": p["gridPos"], "transparent": True, "datasource": DS,
        "targets": p.get("targets", []),
        "options": {"tooltip": {"mode": "multi", "sort": "none"},
            "legend": {"displayMode": "list", "placement": "bottom", "showLegend": True}},
        "fieldConfig": {"defaults": {"color": {"mode": "palette-classic"},
            "custom": {"drawStyle": "line", "fillOpacity": 10, "lineWidth": 1,
                "showPoints": "auto", "spanNulls": False,
                "stacking": {"group": "A", "mode": "none"},
                "hideFrom": {"legend": False, "tooltip": False, "viz": False}},
            "mappings": [], "thresholds": {"mode": "absolute",
                "steps": [{"color": "green", "value": None}]}},
            "overrides": []}}

def to_piechart(p):
    return {"type": "piechart", "title": p.get("title", ""), "id": p["id"],
        "gridPos": p["gridPos"], "transparent": True, "datasource": DS,
        "targets": p.get("targets", []),
        "options": {"pieType": p.get("pieType", "donut"), "displayLabels": [],
            "legend": {"displayMode": "list", "placement": "bottom", "showLegend": True},
            "tooltip": {"mode": "single", "sort": "none"}},
        "fieldConfig": {"defaults": {"color": {"mode": "palette-classic"},
            "custom": {"hideFrom": {"legend": False, "tooltip": False, "viz": False}},
            "mappings": []}, "overrides": []}}

def to_table(p):
    return {"type": "table", "title": p.get("title", ""), "id": p["id"],
        "gridPos": p["gridPos"], "transparent": True, "datasource": DS,
        "targets": p.get("targets", []),
        "options": {"cellHeight": "sm", "showHeader": True,
            "footer": {"countRows": False, "fields": "", "reducer": ["sum"], "show": False}},
        "fieldConfig": {"defaults": {"color": {"mode": "thresholds"},
            "custom": {"align": "auto", "cellOptions": {"type": "auto"}, "inspect": False},
            "mappings": [], "thresholds": {"mode": "absolute",
                "steps": [{"color": "green", "value": None}]}},
            "overrides": []}}

CONVERTERS = {"singlestat": to_stat, "graph": to_timeseries,
    "grafana-piechart-panel": to_piechart, "table-old": to_table}

def process(panels):
    result = []
    for p in panels:
        fn = CONVERTERS.get(p.get("type"))
        if fn:
            result.append(fn(p))
        else:
            if "datasource" in p: p["datasource"] = fix_ds(p["datasource"])
            for t in p.get("targets", []):
                if "datasource" in t: t["datasource"] = fix_ds(t["datasource"])
            if p.get("panels"): p["panels"] = process(p["panels"])
            result.append(p)
    return result

d["panels"] = process(d.get("panels", []))
print(json.dumps(d, indent=2))
```

**Usage:**
```bash
# Write script to server, run conversion, copy result to repo
ssh johan@192.168.1.205 "curl -s 'https://grafana.com/api/dashboards/<ID>/revisions/<rev>/download' | python3 /tmp/convert.py" \
  > apps/grafana/dashboards/<name>.json
```

**After conversion, run the full quality checklist (Step 2)** — especially verify no deprecated panel types remain and the `datasource` template variable is present.
