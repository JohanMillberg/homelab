---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

# grill-me

The user wants to stress-test their plan before committing to it. Your job is to ask hard questions, surface gaps and hidden assumptions, and walk the entire decision tree until nothing is left unresolved.

## How to conduct the interview

**First, explore the codebase.** Before asking questions, look at the relevant parts of the repo — existing apps, Helm values, deployment patterns. This prevents wasting the user's time asking things you could answer yourself, and it gives you the context needed to ask sharper questions.

**Ask one question at a time.** Don't dump a list of 10 questions — that's overwhelming and doesn't let you follow a thread. Ask the most important unresolved question, wait for the answer, then follow up or move to the next branch.

**Follow every branch to resolution.** The design tree has branches. When an answer opens a new question (it usually does), pursue it before moving on. Don't let answers hang unresolved.

**Push back on vague answers.** "It depends" and "probably fine" are red flags. Ask what it depends on and why it's probably fine.

## Question areas for homelab planning

Work through these systematically — not all will apply, but check each:

**Problem & motivation**
- What problem does this solve that the current setup doesn't?
- What would happen if you didn't build this?

**Integration with existing apps**
- Which existing apps (forgejo, n8n, grafana, pihole, plan-assistant) does this touch?
- Does it need to read from or write to any existing data?
- Does it compete for namespace, ports, or ingress paths with anything already deployed?

**Deployment approach**
- Is there a Helm chart for this, or does it need raw manifests (deploy-raw-app)?
- If raw: does it need a database? StatefulSet or Deployment?
- What's the image source — Docker Hub, Forgejo registry, or build-your-own?

**Secrets and config**
- What secrets does this need? (API keys, passwords, database URLs)
- Where do those secrets come from? Are they already in Kubernetes?

**Persistence**
- Does it need persistent storage? If so, how much and what access mode?
- What happens to the data if the pod restarts?

**Networking and DNS**
- What hostname should it get (`.lan`)? Is that already in pihole?
- Does it need to talk to other services inside the cluster? How?

**Failure modes**
- What happens if this app goes down? Is anything else affected?
- What happens if pihole goes down while this is being deployed?
- Is there a rollback plan if the deployment breaks something?

**Observability**
- How will you know if it's healthy? (Grafana dashboard, Prometheus metrics, logs)
- Does the chart expose Prometheus metrics? Should a scrape config be added?

**Maintenance**
- How will you upgrade it? (upgrade-app, image tag bump, or manual?)
- How often does it need to be updated?

## When to stop

Stop grilling when:
- All relevant branches are resolved with concrete, specific answers
- You and the user have a shared, unambiguous understanding of the plan
- There are no dangling "it depends" answers left

Then summarize the decisions made and any open questions the user still needs to answer before implementation begins.