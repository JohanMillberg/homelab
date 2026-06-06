# GitHub work queue for Hermes autonomy

Hermes autonomous coding work will start from `kvarnberg-labs/hermes-work-queue`, using GitHub issues as the remotely reachable work intake and a Hermes GitHub App as the credential identity. Forgejo remains available for LAN-local repositories, but it is not the v1 work queue because the user cannot reliably create, inspect, or review Forgejo issues while away from the home network; Strava and Garmin access are deferred to a later Training Data Connector phase so coding autonomy and personal training-data access do not share one rollout boundary.

The work-queue repository should be private because issues may contain operational intent, repository details, implementation notes, and links to homelab context.

Work discovery will be triggered by a Kubernetes CronJob that runs `hermes chat -q` in a short-lived Hermes CLI pod. That pod should reuse the existing Hermes image, ConfigMap, Secret, and `/opt/data` PVC so it shares durable Hermes state with the always-on gateway, without adding a second long-running Hermes Deployment or relying on `kubectl exec` into the gateway pod.

Because the CronJob is headless and cannot answer terminal approval prompts, the Hermes CLI subprocess should run with scoped auto-approval for command prompts. This does not change the command safety boundary: destructive commands, external deploys, cloud billing changes, data deletion, secret rotation, and production service mutation must be surfaced as approval questions in GitHub or Discord and must not be executed silently.

The CronJobs should not receive broad Kubernetes API permissions in v1. They may receive narrow namespaced Lease permissions only for implementation-lock coordination. Otherwise they should rely on mounted ConfigMap, Secret, and PVC data plus GitHub APIs; cluster mutation remains behind GitOps pull requests and explicit approval.

The CronJob should have outbound HTTPS access for GitHub, package registries, official documentation sites, and model/provider APIs. It should not be exposed as an inbound service; future Kubernetes NetworkPolicies should model this as controlled egress, not public ingress.

During autonomous GitHub work, Hermes may use only the operating credentials it needs: GitHub App credentials, model-provider credentials, and Discord ping credentials. Repo-specific application secrets should not be exposed to Hermes unless a later work item explicitly adds a scoped secret for that purpose.

The CronJob should run hourly. Overlapping runs should be forbidden and each run should have an execution deadline so a long implementation cannot cause concurrent work on the next tick.

The execution deadline should be 45 minutes for the hourly CronJob.

If a run reaches the deadline during active implementation, Hermes should resume the same work item on the next run when the issue worktree and branch are recoverable. It should mark the item blocked after three consecutive timeouts on the same work item, or sooner when the workspace state is unsafe to continue.

GitHub repositories should be cloned under `/opt/data/workspaces/github/<owner>/<repo>` on the shared Hermes PVC. This keeps working copies durable across scheduled runs while separating repository state from Hermes configuration and other runtime data.

Workspace disk usage should have a soft cap on the current 10GiB Hermes PVC: warn and send a review ping when `/opt/data/workspaces` exceeds 7GiB, and block new implementation work when it exceeds 9GiB until cleanup or PVC expansion.

Hermes must not expand the Hermes PVC automatically. Storage expansion is an infrastructure change that requires explicit user approval through the GitOps change boundary.

Hermes should keep one durable repository workspace per GitHub repository and create per-issue git worktrees and branches for implementation. The durable workspace is the base for fetches and resumption; issue-specific changes happen in the worktree so interrupted work can be inspected or cleaned up without dirtying the base checkout.

If a Hermes work item already has an open pull request, that pull request is the active implementation record. Scheduled runs should update the existing branch and pull request when issue changes, pull request comments, or review comments require more work; otherwise Hermes should leave it alone and only ping when user action is needed.

Hermes should open pull requests from branches in the target repository when the GitHub App has write access. Fork or import workflows should be used only for external repositories where direct branch push is unavailable and the user explicitly approves that workflow.

Implementation branches should use `hermes/issue-<number>-<short-slug>` so branches can be traced to work-queue issues and pruned safely later.

After merge and work-item completion, Hermes should delete the remote `hermes/issue-*` branch only when it owns that branch. Local worktree retention remains governed by the 7-day issue worktree retention rule.

Hermes should use normal bot-authored commits for v1 and should not impersonate the user as commit author. Commit signing is out of scope for the first autonomous workflow.

Hermes may squash or rewrite its own noisy WIP commits before opening or updating a pull request when that improves reviewability. It should preserve meaningful commit history when the commits are coherent, and it must not rewrite the branch after user review has started unless the review explicitly asks for it.

Every implementation pull request should map each acceptance criterion to the corresponding code or config change and the validation performed. The PR body should make review possible without reconstructing the issue history.

Validation evidence in pull requests should use command names plus concise pass/fail summaries by default. Short error excerpts should be included when validation fails or is skipped; full logs should be attached or commented only when needed.

Hermes should use a small label set for issue state: `hermes-ready`, `hermes-clarifying`, `hermes-working`, `hermes-review`, `hermes-blocked`, and `hermes-done`. Labels are for queue discovery and coarse state; GitHub comments remain the place for detailed reasoning, clarification questions, implementation updates, and handoff notes.

Implementation work items must include acceptance criteria before Hermes changes code. A checklist is preferred, but one clear expected outcome is acceptable for tiny housekeeping work; missing or vague acceptance criteria should move the issue into clarification.

Hermes should mark `hermes-blocked` only when work cannot safely or usefully continue without user input, credentials, permission changes, or a decision. Valid blockers include missing required information, unavailable credentials or permissions, failing tests Hermes cannot diagnose after one focused attempt, merge conflicts it cannot resolve confidently, and unsafe or destructive operations that need approval.

Hermes may autonomously run normal repo-local build, test, lint, format, and inspection commands. It must ask for approval before destructive commands, external deploys, cloud billing changes, data deletion, secret rotation, or production service mutation.

Hermes may use repo-defined container builds and short-lived local test services for validation. It must not publish images, push to registries, deploy externally, or run long-lived services unless the work item explicitly asks and the user approves.

Hermes should change dependencies only when the work item asks for it or when the dependency change is the narrowest necessary fix. When Hermes must choose a runtime, framework, library, or tool version, it should prefer the LTS or stable supported line unless the work item explicitly requires another version.

Hermes may install repo-local dependencies needed to build, test, lint, or validate implementation, using the repository's existing package manager and lockfile conventions. It must not install global tools or mutate system packages unless the work item explicitly asks and the user approves.

Hermes may use external documentation while implementing tickets when repository-local context is insufficient or current API, library, or platform behavior matters. It should prefer official documentation over blogs, examples, or forum answers, and cite material documentation in the issue comment or pull request body.

Hermes may prepare pull requests against this homelab GitOps repository. Because those changes can affect the cluster after merge, the user keeps review and merge authority, and Hermes must not directly apply cluster changes, delete resources, rotate secrets, or mutate production services unless explicitly approved.

Hermes may create or update SealedSecrets only when a work item explicitly asks for a secret change and the secret material is supplied through the approved sealing workflow. Hermes must never request or store raw secret material in GitHub issues, pull requests, comments, or committed plaintext files.

The homelab GitOps repository is external to the dedicated Hermes GitHub organization. Public external repositories are readable context by default, but Hermes should not assume it can push branches or open implementation pull requests there unless the Hermes GitHub App is installed with write access on the repository owner account or the user explicitly approves a fork/import workflow.

Hermes should close a work-queue issue only after observing that its linked pull request has been merged. While review is pending it should use `hermes-review`; after merge it should comment with the merged pull request, apply `hermes-done`, and close the issue.

After completion, Hermes should retain the issue-specific worktree for 7 days and then prune it. The durable per-repository workspace should remain.

If a work item is closed without a merged pull request, Hermes should retain the issue-specific worktree for 14 days and then prune it. If the issue reopens during that window, Hermes may resume from the existing worktree only when the workspace state is safe to continue.

For v1, Hermes may create new repositories only inside `kvarnberg-labs`, and only from explicit new-repository work items that name the repository, visibility, and purpose. Repository creation outside the lab organization is out of scope until that owner/account is deliberately added and the App is installed there.

Hermes-created repositories should default to private unless the new-repository work item explicitly asks for public visibility.

New repositories should be initialized with a minimal `README.md` and a `.gitignore` appropriate to the requested stack. Hermes should not add a license unless the work item explicitly asks for one.

Hermes should not create CI files by default for new repositories. CI should be added only when the work item specifies a stack and test or validation command clearly enough to justify a workflow.

For existing repositories, Hermes does not use a separate target-repository allowlist in v1. Any repository in `kvarnberg-labs` that the Hermes GitHub App can access may be targeted by a valid ready work item; repositories outside `kvarnberg-labs` remain governed by the external repository boundary.

When multiple work items are ready for implementation, Hermes should start with the oldest ready issue first.

Each hourly run should reconcile existing non-terminal Hermes issues before starting new implementation work. Reconciliation should inspect `hermes-clarifying`, `hermes-review`, and `hermes-blocked` issues for new answers, review feedback, merged pull requests, or user actions; only after that should Hermes start the oldest ready issue if no implementation is active.

User comments on an active implementation pull request should use a review-comment fast path instead of waiting only for the hourly queue-discovery cadence. Hermes should respond by making changes, asking a clarifying question, or explaining why it is blocked, then request review again when ready.

The fast path should include a cheap GitHub-only preflight check. Frequent polling must not invoke Hermes or incur model/provider cost when there are no new actionable comments or review updates; Hermes should be invoked only after the preflight detects new actionable user input on an active implementation pull request.

The fast preflight poll should run every 5 minutes.

Fast-poll cursor state, such as last seen review comment IDs, should be stored on the shared Hermes PVC under `/opt/data/hermes-autonomy/state/`. It should not be stored in GitHub labels, GitHub comments, or Kubernetes Lease annotations.

The hourly scanner should derive work-item truth from GitHub each run. PVC-backed local state may store timeout counters, lock-adjacent metadata, workspace cleanup timestamps, and fast-poll cursors, but it must not become a hidden local work queue or replace GitHub labels, issues, comments, and pull requests as the work source of truth.

The fast preflight poll should run as a separate Kubernetes CronJob from the hourly queue scanner. It may reuse the same Hermes image, ConfigMap, Secret, and PVC, but its normal path is GitHub-only preflight and no Hermes/model invocation unless actionable review input exists.

The fast poll trigger and hourly queue scanner must share one global implementation lock implemented as a Kubernetes Lease in the Hermes namespace. If the lock is held, the fast poll trigger should record the actionable review event for the active work item rather than starting concurrent implementation; if the fast poll trigger holds the lock, the hourly scanner must not start a new implementation. Per-repository locks are out of scope for v1.

The Lease should be renewed periodically while a trigger owns the implementation lock. If the owning pod dies or stops renewing, the next trigger run may acquire the lock after the Lease expires; it must not steal an actively renewed Lease.

The two triggers should use separate `hermes chat -q` prompt templates. The hourly queue-scanner prompt handles work-queue bootstrap, reconciliation, and starting the oldest ready item. The fast review-response prompt handles only the detected review/comment event for an existing implementation pull request and must not start new work items.

Prompt templates should be delivered as Kubernetes ConfigMaps managed in this GitOps repository rather than baked into the Hermes image.

Hermes should not post hourly heartbeat comments. It should comment only on meaningful state changes: clarification asked, work started, pull request opened or updated, blocked, review requested, merged/done, or a failed run that needs attention.

Discord review pings should be sent only when user attention is required: clarification questions, blocked items, pull requests ready for review, failed runs needing intervention, or completion summaries. GitHub remains the detailed audit trail, and Discord should not mirror every meaningful GitHub comment.

Hermes should own work-queue bootstrap for `kvarnberg-labs/hermes-work-queue`, including creating or updating the required issue template and labels. The autonomous workflow should not depend on manual one-time label or template setup that can drift.

Work-queue bootstrap should be idempotent and run at the start of each hourly scan before queue reconciliation. Missing labels or templates should be created, and drifted definitions should be updated before Hermes inspects work-item state.

Bootstrap should also ensure `kvarnberg-labs/hermes-work-queue` exists. If the repository is missing and the Hermes GitHub App has repository-creation permission, Hermes should create it; if creation fails because permission is missing, Hermes should mark the run blocked and ping the user for the permission change.

## Addendum (2026-06-06): GitHub App `Workflows: write`

The Hermes GitHub App was granted the `Workflows: write` repository permission (in addition to the existing `Contents: write` and `Pull requests: write`). GitHub rejects any push that creates or modifies `.github/workflows/*` unless the authenticating identity holds this scope, which previously forced Hermes to split commits around workflow files and blocked it from setting up CI. Because Hermes authenticates as a GitHub App (installation tokens minted on demand by `apps/hermes/github-tools-configmap.yaml`), no new secret or PAT is introduced — the short-lived installation token simply inherits the added scope once the org accepts the updated permission. Authoring CI workflows remains gated behind ordinary work items and pull-request review; this change only removes the push-level block.
