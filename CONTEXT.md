# Homelab GitOps

This context describes user-facing deployment concepts in the homelab cluster so new services are described consistently across GitOps manifests, operations, and docs.

## Language

**Remote Chat Surface**:
A messaging platform the user can reach from outside the LAN to talk to an agent running in the homelab.
_Avoid_: Web UI, API endpoint

**Hermes Gateway**:
The always-on Hermes process that receives messages from a remote chat surface and routes them into Hermes agent sessions.
_Avoid_: Hermes UI, bot frontend

**Discord Bot**:
The first remote chat surface for Hermes, authorized by a Discord bot token and restricted to approved Discord user IDs.
_Avoid_: Telegram bot, LAN chat

**Hermes Secret**:
The sealed Kubernetes secret that provides Hermes with messaging credentials and model-provider credentials.
_Avoid_: Plain env file, ConfigMap credentials, committed token

**Ollama Cloud Account**:
The v1 model-provider account for Hermes, authenticated through an `OLLAMA_API_KEY` stored in the **Hermes Secret**.
_Avoid_: Nous OAuth state, plaintext API key, ConfigMap credentials

## Relationships

- A **Discord Bot** is one **Remote Chat Surface**.
- A **Hermes Gateway** connects to exactly one v1 **Discord Bot**.
- A **Discord Bot** sends user messages to the **Hermes Gateway**.
- A **Hermes Secret** supplies credentials to the **Hermes Gateway**.
- An **Ollama Cloud Account** supplies model access to the **Hermes Gateway**.

## Current State

- Hermes v1 runs in the `hermes` namespace as a raw Kubernetes Deployment.
- The **Hermes Gateway** uses the **Discord Bot** as the active **Remote Chat Surface**.
- The **Discord Bot** is installed in a private Discord server and is allowlisted by numeric Discord user ID.
- The **Hermes Gateway** is authenticated to the **Ollama Cloud Account** through `OLLAMA_API_KEY` in the **Hermes Secret**.
- Hermes v1 pins its main model from Kubernetes config to `kimi-k2.6`, with the startup wrapper syncing that value into Hermes' persisted `config.yaml`.

## Example Dialogue

> **Dev:** "Should Hermes be exposed through a web UI first?"
> **Domain expert:** "No - v1 needs a **Remote Chat Surface**, and that surface is the **Discord Bot**."
> **Dev:** "Can I put the Telegram token in the ConfigMap while testing?"
> **Domain expert:** "No - credentials belong in the **Hermes Secret**."
> **Dev:** "Can I put the Ollama API key in the ConfigMap while testing?"
> **Domain expert:** "No - model-provider credentials belong in the **Hermes Secret**."

## Flagged Ambiguities

- "Access Hermes remotely" could mean a public web UI, an API endpoint, or a messaging integration; resolved for v1 as a **Discord Bot** remote chat surface.
- "Credentials" covers both the Telegram bot token and model-provider keys; resolved as **Hermes Secret** material, not ConfigMap data.
- "Model provider" could mean a direct API key provider or a subscription provider; resolved for v1 as **Ollama Cloud Account**.
- "Messaging service" was initially resolved as Telegram, then changed because the user decided to delete Telegram; v1 is now Discord.
