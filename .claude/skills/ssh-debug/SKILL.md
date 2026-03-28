---
name: ssh-debug
description: Use when diagnosing node-level issues on the homelab server — disk pressure, OOMKilled pods, node NotReady, high memory/CPU, k3s service failures, or anything requiring direct SSH access to 192.168.1.205. Always use this skill for server/node health questions, not app-level Flux issues (use debug-flux for those). For pod crashes caused by hardware resource exhaustion, this skill is the right choice.
---

# ssh-debug

## Overview

Claude SSHes to `johan@192.168.1.205` and runs a full triage sequence, reports findings back to the user, then applies fixes. The node runs k3s (Ubuntu) and also hosts pihole — restarts are disruptive.

**Connection:** `ssh johan@192.168.1.205`

**Pihole DNS risk:** This node runs pihole at `192.168.1.205`. Restarting k3s or the node will take down DNS for the network. Claude must warn the user before any action that could cause downtime.

**KUBECONFIG:** The `johan` user cannot read `/etc/rancher/k3s/k3s.yaml`. Prefix every `kubectl` and `flux` command with `KUBECONFIG=/home/johan/.kube/config`.

**sudo:** Non-interactive SSH sessions cannot prompt for a sudo password (fails with "terminal required" error). Commands that require sudo (`systemctl`, `journalctl`, `crictl`) will only work if `johan` is configured for passwordless sudo. If a sudo command fails, report the output to the user and ask them to run it directly.

---

## 1 — Triage sequence

Claude runs all of these and reports the results:

```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get nodes"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl get pods -A | grep -v Running | grep -v Completed"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl top nodes"
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl top pods -A --sort-by=memory | head -20"
ssh johan@192.168.1.205 "df -h"
ssh johan@192.168.1.205 "free -h"
ssh johan@192.168.1.205 "sudo systemctl status k3s"
ssh johan@192.168.1.205 "sudo journalctl -u k3s --since '1 hour ago' --no-pager | tail -50"
```

Claude summarizes findings: which pods are unhealthy, disk usage per mount, memory pressure, and any k3s errors from the journal.

---

## 2 — Fix execution

Based on triage output, Claude applies the appropriate fix:

**Unhealthy pod (crashloop, pending):**
```bash
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config kubectl delete pod <pod-name> -n <ns>"
# If Flux-managed, also reconcile:
ssh johan@192.168.1.205 "KUBECONFIG=/home/johan/.kube/config flux reconcile helmrelease <name> -n <ns> --with-source"
```

**Disk pressure — clean up old images:**
```bash
ssh johan@192.168.1.205 "sudo k3s crictl rmi --prune"
```

**Disk pressure — Longhorn volumes:**
Report findings to user and invoke `debug-flux` skill if Longhorn HelmRelease is unhealthy.

**k3s unhealthy (non-restart fixes first):**
```bash
ssh johan@192.168.1.205 "sudo systemctl restart k3s"
```

---

## 3 — Safety gate

**Before restarting k3s or touching Longhorn volumes, Claude must confirm with the user.**

Restarting k3s causes downtime for:
- All apps (forgejo, n8n, pihole)
- DNS resolution on the local network (pihole is a pod)

Claude should say: "This will take down all apps including pihole (DNS). Confirm before I proceed?"

Claude must NOT restart k3s or delete Longhorn resources without explicit user confirmation in the current conversation.

---

## Common Mistakes

| Mistake | What Claude should watch out for |
|---------|----------------------------------|
| Restarting k3s without warning | k3s restart = network DNS outage; always confirm with user first |
| Deleting Longhorn volumes to free disk space | Longhorn volumes hold persistent data; data loss is permanent |
| Ignoring pods in `Terminating` state | Stuck terminating pods often indicate a deeper issue — check node pressure first |
| Treating high memory usage as always problematic | k3s and Longhorn use significant memory normally; only act if OOMKilled events appear |
| Running `kubectl get pods` without `-A` | Missing the `-A` flag only shows the default namespace — homelab apps are in named namespaces |
| Not checking `journalctl` for k3s errors | Pod-level logs miss node-level k3s failures; always check the service journal |
