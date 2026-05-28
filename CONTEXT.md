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

**Hermes Runtime Secret Boundary**:
The rule for which secrets Hermes may use during autonomous GitHub work.
_Avoid_: Catch-all project secret access, repo-specific app credentials by default, plaintext secret handoff

**OpenCode Go Account**:
The v1 model-provider account for Hermes, authenticated through an `OPENCODE_GO_API_KEY` stored in the **Hermes Secret**.
_Avoid_: Nous OAuth state, plaintext API key, ConfigMap credentials

**Remote Work Intake**:
The internet-reachable place where the user creates work requests for Hermes while away from the home network.
_Avoid_: LAN-only backlog, Discord-only instruction

**GitHub Work Queue**:
The GitHub issue queue Hermes polls for approved autonomous coding work.
_Avoid_: Forgejo-only tickets, chat backlog

**Work Queue Visibility**:
The repository visibility for the GitHub Work Queue.
_Avoid_: Public operational backlog, hidden unmanaged queue, accidental disclosure

**Hermes GitHub Organization**:
The dedicated GitHub organization that owns Hermes-managed repositories and the GitHub Work Queue.
_Avoid_: Personal account sprawl, mixed-owner repos

**Hermes GitHub App**:
The GitHub App identity Hermes uses to discover work, create repositories, push branches, comment, and open pull requests.
_Avoid_: Personal access token, user-owned bot password

**Hermes Work Item**:
A single issue in the GitHub Work Queue that asks Hermes to make a code change, create a repository, or prepare a pull request.
_Avoid_: Ticket, prompt, task

**Work Item Template**:
The required issue body structure that tells Hermes whether work targets an existing repository or a new repository.
_Avoid_: Free-form-only request, inferred target

**Acceptance Criteria Gate**:
The requirement that implementation work has clear expected outcomes before Hermes changes code.
_Avoid_: Inferred success, vague done state, implementation from intent alone

**Work Queue Bootstrap**:
The Hermes-owned setup of the GitHub Work Queue labels and issue template required for autonomous processing.
_Avoid_: Manual-only queue setup, undocumented labels, drifted issue template

**Work Readiness Gate**:
The explicit label, author, and template-validity check that lets Hermes start processing a Hermes Work Item.
_Avoid_: Every open issue, implicit priority, issue assignment

**Work State Labels**:
The small GitHub label set Hermes uses to make work item state visible and discoverable.
_Avoid_: State hidden only in comments, large workflow taxonomy, project-board dependency

**Clarification Loop**:
The question-and-answer phase Hermes runs when a Hermes Work Item is ambiguous before it changes code.
_Avoid_: Best-effort guessing, silent assumptions

**Blocked Work Item**:
A Hermes Work Item Hermes cannot safely or usefully advance without user input, credentials, permission changes, or a decision.
_Avoid_: Routine failing test, first debugging obstacle, vague discomfort

**Command Safety Boundary**:
The rule separating repo-local implementation commands Hermes may run autonomously from destructive or externally mutating commands that need approval.
_Avoid_: Arbitrary shell authority, approval for every harmless test, silent production mutation

**Headless Command Approval Mode**:
The scoped auto-approval setting used only by short-lived Hermes CronJob subprocesses so autonomous runs do not block on terminal prompts.
_Avoid_: Gateway-wide YOLO mode, silent destructive operations, replacing the Command Safety Boundary

**Container Validation Boundary**:
The rule for when Hermes may use repo-defined containers during implementation validation.
_Avoid_: Registry push, long-running service, external deploy disguised as validation

**Dependency Change Boundary**:
The rule for when Hermes may add, upgrade, or change project dependencies.
_Avoid_: Opportunistic upgrade, latest-by-default churn, broad dependency rewrite

**Dependency Install Boundary**:
The rule for when Hermes may install existing project dependencies while implementing and validating work.
_Avoid_: Global tool install, system package mutation, ignoring lockfile convention

**Dependency Version Default**:
The preferred dependency version line when Hermes must choose a runtime, framework, library, or tool version.
_Avoid_: Unstable latest, unsupported old version, unspecified version drift

**External Documentation Use**:
The rule for when Hermes may use internet documentation while implementing a work item.
_Avoid_: Unsourced API assumptions, blog-first implementation, stale examples

**GitOps Change Boundary**:
The rule for Hermes-authored changes to this homelab GitOps repository, where a pull request may affect deployed cluster state after merge.
_Avoid_: Direct cluster mutation, auto-merge deploy, unreviewed secret rotation

**Sealed Secret Change Boundary**:
The rule for Hermes-authored SealedSecret changes in the homelab GitOps repository.
_Avoid_: Raw secret in GitHub issue, plaintext secret commit, inferred secret rotation

**External Repository Boundary**:
The rule for repositories outside the Hermes GitHub Organization.
_Avoid_: Assuming org installation grants write access, silent fork creation, treating public repos as writable

**Review Ping**:
A Discord notification that points the user to a GitHub issue, clarification question, or pull request that needs attention.
_Avoid_: Source of truth, hidden chat state

**Attention Ping Threshold**:
The rule for which GitHub work events should produce a Discord Review Ping.
_Avoid_: Mirroring every comment, silent user-blocking state, Discord as audit log

**Code Host**:
The git forge where Hermes creates repositories, pushes branches, and opens pull requests.
_Avoid_: Ticket system, work queue

**New Repository Request**:
A Hermes Work Item that explicitly asks Hermes to create a repository before doing implementation work.
_Avoid_: Implicit repo creation, scratch repo

**Repository Creation Scope**:
The GitHub owner where Hermes is allowed to create new repositories.
_Avoid_: Personal-account repo creation, external owner sprawl, inferred destination

**New Repository Visibility Default**:
The default visibility Hermes uses when creating a new repository.
_Avoid_: Accidental public repository, inferred openness, private-only hard stop

**New Repository Initialization**:
The initial files Hermes creates when it creates a repository.
_Avoid_: Empty repo unless requested, inferred license, overbuilt scaffold

**New Repository CI Default**:
The rule for whether Hermes creates CI configuration when initializing a repository.
_Avoid_: Wrong generated workflow, CI without test command, hidden maintenance burden

**Target Repository Scope**:
The set of existing repositories Hermes may target from ready work items.
_Avoid_: Separate v1 allowlist, implicit external write access, hidden repo denylist

**Queue Ordering**:
The rule Hermes uses to choose which ready work item to start implementing next.
_Avoid_: Random issue selection, newest-first churn, parallel starts

**Queue Reconciliation**:
The scan Hermes performs before starting new implementation work to update existing non-terminal work items.
_Avoid_: Ignoring answered clarifications, stale review state, starting new work before reconciling current work

**Review Comment Fast Path**:
The faster reaction path Hermes uses for user comments on active implementation pull requests.
_Avoid_: Waiting for hourly queue discovery, ignoring review feedback, treating review comments as new tickets

**Fast Poll Preflight**:
The low-cost GitHub-only check that detects whether the Review Comment Fast Path needs to invoke Hermes.
_Avoid_: Calling Hermes with no new work, model-cost heartbeat, blind frequent agent run

**Fast Poll Cadence**:
The schedule for the GitHub-only preflight that watches active implementation pull requests.
_Avoid_: Hourly review lag, sub-minute polling churn, model-call polling

**Fast Poll Cursor State**:
The persisted local state that records which pull request comments and review events the Fast Poll Preflight has already seen.
_Avoid_: GitHub label cursor, Lease annotation state, duplicate review handling

**Autonomy Local State**:
The PVC-backed operational state Hermes keeps that is not the source of truth for work items.
_Avoid_: Hidden local queue, GitHub-state duplication, state needed to understand intent

**Work Item Comment Policy**:
The rule for when Hermes writes GitHub comments on work items.
_Avoid_: Hourly heartbeat comments, silent state transitions, noisy audit trail

**Merge Authority**:
The permission to accept a pull request into the target branch.
_Avoid_: PR creation, implementation authority

**Hermes CLI Work Trigger**:
The scheduled Kubernetes CronJob that runs `hermes chat -q` in a short-lived Hermes pod to ask Hermes to scan and process the GitHub Work Queue.
_Avoid_: Separate long-running controller, kubectl exec into the gateway pod, per-ticket worker swarm

**Fast Poll Trigger**:
The separate Kubernetes CronJob that runs the Fast Poll Preflight for active implementation pull requests.
_Avoid_: Merging fast polling into hourly scan, model-call polling loop, long-running webhook service

**Hermes Trigger Prompt Template**:
The explicit prompt template a CronJob passes to `hermes chat -q`.
_Avoid_: One prompt for every trigger, implicit trigger behavior, fast path starting new tickets

**Prompt Template Delivery**:
The GitOps mechanism that delivers Hermes Trigger Prompt Templates to the CronJob pods.
_Avoid_: Baked image prompt, unreviewed runtime edit, hidden prompt drift

**Kubernetes Permission Boundary**:
The rule for Kubernetes API permissions granted to Hermes autonomous GitHub work.
_Avoid_: Cluster controller role, broad service account, direct production mutation

**Network Access Boundary**:
The network access Hermes autonomous GitHub work needs at runtime.
_Avoid_: Inbound service exposure, unrestricted public service, offline-only worker

**Work Trigger Cadence**:
The schedule for the Hermes CLI Work Trigger.
_Avoid_: Webhook-only trigger, constant polling, manual-only scans

**Work Trigger Deadline**:
The hard runtime limit for one Hermes CLI Work Trigger CronJob run.
_Avoid_: Unbounded run, overlapping hourly work, stuck implementation

**Timed-Out Work Recovery**:
The rule for what Hermes does when a scheduled run reaches its deadline during active work.
_Avoid_: Immediate blocked state, abandoned worktree, duplicate restart from scratch

**Existing Gateway Enablement**:
The constraint that GitHub autonomy reuses the existing Hermes image, ConfigMap, Secret, and PVC rather than introducing a second long-running Hermes service.
_Avoid_: Additional worker deployment, separate controller, duplicate Hermes installation

**Shared Hermes Home**:
The PVC-backed Hermes state directory mounted at `/opt/data` by both the Hermes Gateway and the Hermes CLI Work Trigger.
_Avoid_: Per-job empty state, unmounted workspace, hidden local disk

**Hermes Workspace Root**:
The directory under the Shared Hermes Home where Hermes keeps cloned repositories and active coding workspaces.
_Avoid_: Random temp directories, home-directory sprawl, hidden per-run checkouts

**Workspace Disk Soft Cap**:
The warning and stop-work thresholds for disk usage under the Hermes Workspace Root.
_Avoid_: Filled PVC, silent workspace growth, hard delete without review

**Hermes Repo Workspace**:
The durable per-repository checkout area under the Hermes Workspace Root that Hermes uses as the base for issue-specific work.
_Avoid_: Reclone every run, shared dirty working tree, unrelated repo cache

**Issue Worktree**:
The per-Hermes Work Item git worktree and branch Hermes uses while implementing one issue.
_Avoid_: Editing the base checkout directly, mixing issues on one branch, anonymous scratch directory

**Implementation Pull Request**:
The pull request Hermes opens or updates for one Hermes Work Item.
_Avoid_: Duplicate PR, hidden branch-only work, merge authority

**Pull Request Branch Source**:
The rule for whether Hermes opens pull requests from target-repository branches or forks.
_Avoid_: Fork-first for owned repos, assuming external write access, hidden import workflow

**Hermes Branch Naming**:
The branch naming convention Hermes uses for implementation branches.
_Avoid_: Anonymous branch, reused branch name, untraceable issue link

**Remote Branch Cleanup**:
The rule for deleting Hermes-owned remote implementation branches after pull request merge.
_Avoid_: Stale remote branch, deleting user-owned branch, confusing branch retention with worktree retention

**Hermes Commit Identity**:
The commit authoring and signing policy Hermes uses for implementation commits.
_Avoid_: User impersonation, signing-key rollout in v1, anonymous commit author

**Hermes Commit Hygiene**:
The rule for cleaning up Hermes-authored commits before and after review.
_Avoid_: Noisy WIP history, rewriting reviewed work unasked, squashing meaningful commits blindly

**Pull Request Review Contract**:
The required pull request body structure that maps acceptance criteria to implementation and validation.
_Avoid_: Vague PR summary, hidden validation, untraceable criteria

**Validation Evidence Style**:
The amount of test or command output Hermes includes in pull requests and comments.
_Avoid_: Full log dumps by default, pass/fail without command names, hidden failures

**Work Item Completion**:
The terminal queue state after Hermes observes that an Implementation Pull Request has been merged.
_Avoid_: Closing before merge, stale review label, unlinked completion

**Issue Worktree Retention**:
The cleanup rule for completed issue-specific worktrees under the Hermes Workspace Root.
_Avoid_: Unbounded PVC growth, immediate forensic loss, deleting repo workspace

**Abandoned Worktree Retention**:
The cleanup rule for issue-specific worktrees whose Hermes Work Item was closed without merged completion.
_Avoid_: Immediate loss after close, permanent abandoned worktree, unsafe resume

**Active Implementation**:
The phase where Hermes is modifying a checkout, running commands, committing, and pushing code for one Hermes Work Item.
_Avoid_: Clarification, queue scanning

**Implementation Lock**:
The shared coordination state that prevents the hourly scanner and fast poll trigger from running concurrent implementation work.
_Avoid_: Parallel branch mutation, hidden concurrent work, separate locks per trigger

**Kubernetes Lease Coordination**:
The narrow Kubernetes Lease permission Hermes uses only for the shared Implementation Lock.
_Avoid_: Broad API access, PVC file-lock race, cluster mutation authority

**Stale Lease Recovery**:
The rule for recovering the Implementation Lock when a CronJob pod dies or stops renewing its Kubernetes Lease.
_Avoid_: Manual unlock by default, permanent stuck lock, stealing a live lock

**Training Data Connector**:
A future integration that gives Hermes controlled access to training data from providers such as Strava or Garmin.
_Avoid_: Direct provider token in Hermes, mixed coding workflow

## Relationships

- A **Discord Bot** is one **Remote Chat Surface**.
- A **Hermes Gateway** connects to exactly one v1 **Discord Bot**.
- A **Discord Bot** sends user messages to the **Hermes Gateway**.
- A **Hermes Secret** supplies credentials to the **Hermes Gateway**.
- The **Hermes Runtime Secret Boundary** limits which credentials from the **Hermes Secret** may be used for autonomous GitHub work.
- An **OpenCode Go Account** supplies model access to the **Hermes Gateway**.
- A **GitHub Work Queue** contains many **Hermes Work Items**.
- The **Work Queue Visibility** controls whether the **GitHub Work Queue** is public or private.
- The **Hermes GitHub Organization** owns the **GitHub Work Queue** and v1 GitHub **Code Host** repositories.
- The **Hermes GitHub App** is installed on the **Hermes GitHub Organization**.
- The **Hermes GitHub App** supplies GitHub credentials to Hermes through the **Hermes Secret**.
- **Work Queue Bootstrap** ensures the **GitHub Work Queue** repository exists before maintaining queue metadata.
- A **Hermes Work Item** targets one **Code Host**.
- A **Hermes Work Item** must follow the **Work Item Template** before Hermes can decide whether to implement or clarify.
- The **Acceptance Criteria Gate** is part of deciding whether a **Hermes Work Item** can enter implementation.
- **Work Queue Bootstrap** creates and maintains the **Work Item Template** and **Work State Labels** in the **GitHub Work Queue**.
- **Work Queue Bootstrap** runs before **Queue Reconciliation**.
- A **Hermes Work Item** must pass the **Work Readiness Gate** before Hermes starts implementation or clarification.
- **Work State Labels** mark whether a **Hermes Work Item** is ready, clarifying, working, in review, blocked, or done.
- A **Hermes Work Item** enters the **Clarification Loop** when it lacks enough information for a defensible implementation.
- A **Blocked Work Item** must have a GitHub comment explaining the blocker and the specific user action or answer needed.
- The **Command Safety Boundary** defines when Hermes can continue autonomously and when it must ask for approval.
- The **Container Validation Boundary** is a repo-local validation case inside the **Command Safety Boundary**.
- The **Dependency Change Boundary** limits dependency changes to requested or necessary work.
- The **Dependency Install Boundary** allows repo-local installs needed for validation without authorizing dependency changes.
- The **Dependency Version Default** applies when the **Dependency Change Boundary** permits a dependency choice and the work item does not specify a version.
- **External Documentation Use** is allowed when repository-local context is insufficient or current API, library, or platform behavior matters.
- The **GitOps Change Boundary** is a stricter case of the **Command Safety Boundary** for homelab repository changes that can affect cluster state.
- The **Sealed Secret Change Boundary** is a stricter case of the **GitOps Change Boundary** for encrypted Kubernetes secrets.
- The **External Repository Boundary** applies when a **Hermes Work Item** targets a repository outside the **Hermes GitHub Organization**.
- A **Clarification Loop** records questions and answers on the **Hermes Work Item**.
- A **Review Ping** notifies the user that a **Hermes Work Item** or pull request needs attention.
- The **Attention Ping Threshold** decides whether a meaningful GitHub event should send a **Review Ping**.
- A **Code Host** can be GitHub for remotely reviewable work or Forgejo for LAN-local homelab work.
- A **New Repository Request** must name the repository, visibility, and initial purpose before Hermes creates it.
- **Repository Creation Scope** limits **New Repository Requests** to the **Hermes GitHub Organization**.
- A **New Repository Visibility Default** applies when a **New Repository Request** does not explicitly request public visibility.
- **New Repository Initialization** defines the initial files created for a **New Repository Request**.
- **New Repository CI Default** controls whether CI files are created during **New Repository Initialization**.
- **Target Repository Scope** controls which existing repositories Hermes may modify from a **Hermes Work Item**.
- **Queue Reconciliation** happens before **Queue Ordering** starts a new **Active Implementation**.
- **Queue Ordering** selects one **Hermes Work Item** when multiple items are ready for implementation.
- A **Review Comment Fast Path** reacts to user comments on an existing **Implementation Pull Request** faster than normal hourly queue discovery.
- A **Fast Poll Preflight** decides whether the **Review Comment Fast Path** has new actionable review input before Hermes is invoked.
- A **Fast Poll Cadence** controls how often the **Fast Poll Preflight** checks GitHub.
- **Fast Poll Cursor State** lives in the **Shared Hermes Home**.
- **Autonomy Local State** lives in the **Shared Hermes Home** and supports trigger operation without replacing GitHub as the source of truth.
- A **Work Item Comment Policy** keeps the issue timeline tied to meaningful state changes.
- The user retains **Merge Authority** for pull requests created by Hermes.
- A **Hermes CLI Work Trigger** uses **Existing Gateway Enablement** by running the same Hermes image with the same config, secret, and persistent state as the **Hermes Gateway**.
- A **Fast Poll Trigger** is separate from the hourly **Hermes CLI Work Trigger**.
- A **Hermes Trigger Prompt Template** constrains what each trigger asks Hermes to do.
- **Prompt Template Delivery** mounts **Hermes Trigger Prompt Templates** into trigger pods.
- The **Shared Hermes Home** gives the **Hermes Gateway** and **Hermes CLI Work Trigger** access to the same durable Hermes state, but it does not make a scheduled run part of the same live Discord session.
- A **Hermes Workspace Root** lives inside the **Shared Hermes Home**.
- The **Workspace Disk Soft Cap** applies to the **Hermes Workspace Root**.
- A **Hermes Repo Workspace** lives under the **Hermes Workspace Root** for each GitHub repository Hermes touches.
- An **Issue Worktree** is created from a **Hermes Repo Workspace** for one **Hermes Work Item**.
- An **Implementation Pull Request** is the durable review record for an implemented **Hermes Work Item**.
- **Pull Request Branch Source** determines whether an **Implementation Pull Request** comes from a target-repository branch or fork.
- **Hermes Branch Naming** ties an implementation branch back to its **Hermes Work Item**.
- **Remote Branch Cleanup** applies after **Work Item Completion** for branches matching **Hermes Branch Naming**.
- **Hermes Commit Identity** identifies Hermes-authored implementation commits.
- **Hermes Commit Hygiene** controls when Hermes may rewrite its own implementation branch history.
- A **Pull Request Review Contract** maps the **Acceptance Criteria Gate** to the implementation and validation evidence in an **Implementation Pull Request**.
- **Validation Evidence Style** controls how much command output appears in an **Implementation Pull Request**.
- **Work Item Completion** follows a merged **Implementation Pull Request**.
- **Issue Worktree Retention** starts after **Work Item Completion**.
- **Abandoned Worktree Retention** starts when a **Hermes Work Item** is closed without **Work Item Completion**.
- A **Work Trigger Cadence** controls how often the **Hermes CLI Work Trigger** inspects the **GitHub Work Queue**.
- The **Kubernetes Permission Boundary** limits the **Hermes CLI Work Trigger** to mounted runtime inputs rather than broad Kubernetes API access.
- The **Network Access Boundary** gives the **Hermes CLI Work Trigger** outbound access needed for implementation and validation.
- A **Work Trigger Deadline** bounds each **Hermes CLI Work Trigger** run.
- **Timed-Out Work Recovery** uses the durable **Issue Worktree** and branch from a prior run when work remains safe to continue.
- A **Hermes CLI Work Trigger** permits at most one **Active Implementation** at a time.
- The **Implementation Lock** is shared by the hourly **Hermes CLI Work Trigger** and **Fast Poll Trigger**.
- **Kubernetes Lease Coordination** is the storage mechanism for the **Implementation Lock**.
- **Stale Lease Recovery** applies when **Kubernetes Lease Coordination** is no longer renewed.
- A **Training Data Connector** is outside the v1 GitHub autonomy workflow.

## Current State

- Hermes v1 runs in the `hermes` namespace as a raw Kubernetes Deployment.
- The **Hermes Gateway** uses the **Discord Bot** as the active **Remote Chat Surface**.
- The **Discord Bot** is installed in a private Discord server and is allowlisted by numeric Discord user ID.
- The **Hermes Gateway** is authenticated to the **OpenCode Go Account** through `OPENCODE_GO_API_KEY` in the **Hermes Secret**.
- The **Hermes Runtime Secret Boundary** should allow Hermes to use GitHub App credentials, model-provider credentials, and Discord ping credentials needed to operate.
- The **Hermes Runtime Secret Boundary** should not expose repo-specific application secrets to Hermes unless a later **Hermes Work Item** explicitly adds a scoped secret for that purpose.
- Hermes v1 pins its main model from Kubernetes config to `deepseek-v4-flash`, with the startup wrapper syncing that value into Hermes' persisted `config.yaml`.
- Autonomous coding work should use a **GitHub Work Queue** because Forgejo is only reachable from the home network.
- The **GitHub Work Queue** is `kvarnberg-labs/hermes-work-queue`.
- The **Work Queue Visibility** should be private.
- V1 remotely reviewable repositories should live under the **Hermes GitHub Organization** `kvarnberg-labs`.
- GitHub access should use a **Hermes GitHub App** rather than a personal access token.
- A **Hermes GitHub App** has been created and installed for the dedicated GitHub organization.
- Forgejo remains a valid **Code Host** for LAN-local projects, but it is not the canonical **Remote Work Intake**.
- **Hermes Work Items** should use a structured **Work Item Template** with `Intent`, `Target repository`, `New repository`, `Goal`, `Acceptance criteria`, `Constraints`, and `Notes` sections.
- Hermes should own **Work Queue Bootstrap** for the **GitHub Work Queue**, including creating or updating the issue template and required labels.
- **Work Queue Bootstrap** should create `kvarnberg-labs/hermes-work-queue` if it is missing and the **Hermes GitHub App** has repository-creation permission.
- If **Work Queue Bootstrap** cannot create the **GitHub Work Queue** because of missing permission, Hermes should mark the run blocked and send a **Review Ping**.
- **Work Queue Bootstrap** should be idempotent and run at the start of each hourly **Hermes CLI Work Trigger** scan.
- A **Hermes Work Item** should pass the **Work Readiness Gate** only when it is labeled `hermes-ready`, authored by `JohanMillberg`, and structurally valid against the **Work Item Template**.
- Implementation **Hermes Work Items** must pass the **Acceptance Criteria Gate** before Hermes changes code.
- The **Acceptance Criteria Gate** requires a checklist or a clear expected outcome; missing or vague acceptance criteria enter the **Clarification Loop**.
- **Work State Labels** should be limited to `hermes-ready`, `hermes-clarifying`, `hermes-working`, `hermes-review`, `hermes-blocked`, and `hermes-done`.
- **Work State Labels** should expose queue state; GitHub comments should carry the detailed reasoning, questions, status updates, and handoff notes.
- Passing the **Work Readiness Gate** authorizes Hermes to begin processing, but unclear work must go through the **Clarification Loop** before implementation.
- Hermes should mark a **Blocked Work Item** only for missing required information, unavailable credentials or permissions, failing tests it cannot diagnose after one focused attempt, merge conflicts it cannot resolve confidently, or unsafe/destructive operations that need approval.
- Hermes should not mark routine implementation uncertainty as blocked when it can ask a clarification question, inspect code, run tests, or make a defensible narrow change.
- The **Command Safety Boundary** should allow normal repo-local build, test, lint, format, and inspection commands.
- The **Command Safety Boundary** should require approval before destructive commands, external deploys, cloud billing changes, data deletion, secret rotation, or production service mutation.
- **Headless Command Approval Mode** is allowed only inside the short-lived **Hermes CLI Work Trigger** and **Fast Poll Trigger** subprocesses; it prevents terminal approval prompts from hanging CronJobs, but it does not permit bypassing the **Command Safety Boundary**.
- The **Container Validation Boundary** allows repo-local container builds and short-lived local test services when the repository already defines them.
- The **Container Validation Boundary** does not allow image publishing, registry pushes, external deploys, or long-running services unless the **Hermes Work Item** explicitly asks and the user approves.
- Hermes should change dependencies only when the **Hermes Work Item** asks for it or when the dependency change is the narrowest necessary fix.
- Hermes should not perform opportunistic dependency updates outside the **Dependency Change Boundary**.
- Hermes may install repo-local dependencies needed to build, test, lint, or validate implementation under the **Dependency Install Boundary**.
- The **Dependency Install Boundary** requires Hermes to use the repository's existing package manager and lockfile conventions.
- Hermes must not install global tools or mutate system packages unless the **Hermes Work Item** explicitly asks and the user approves.
- The **Dependency Version Default** is the LTS or stable supported line for each dependency unless the **Hermes Work Item** explicitly requires another version.
- Hermes may use **External Documentation Use** while implementing tickets, preferring official documentation over blogs, examples, or forum answers.
- When **External Documentation Use** materially affects implementation, Hermes should cite or link the documentation in the issue comment or pull request body.
- Hermes may prepare pull requests against this homelab GitOps repository, but the **GitOps Change Boundary** requires user review and merge before changes affect production.
- Under the **GitOps Change Boundary**, Hermes must not directly apply cluster changes, delete resources, rotate secrets, or mutate production services unless explicitly approved.
- Hermes may create or update SealedSecrets only under the **Sealed Secret Change Boundary**.
- The **Sealed Secret Change Boundary** requires an explicit secret-change **Hermes Work Item** and secret material supplied through the approved sealing workflow.
- Hermes must never request or store raw secret material in GitHub issues, pull requests, comments, or committed plaintext files.
- This homelab GitOps repository is outside the **Hermes GitHub Organization**, so the **External Repository Boundary** applies to Hermes-authored pull requests here.
- Under the **External Repository Boundary**, public repositories are read-only context by default unless the **Hermes GitHub App** is installed with write access on the repository owner account or a user-approved fork/import workflow is used.
- The **Clarification Loop** should use GitHub issue comments as the durable source of truth and **Review Pings** in Discord as notifications.
- Hermes should send **Review Pings** only when the **Attention Ping Threshold** is met.
- The **Attention Ping Threshold** should include clarification questions, blocked items, pull requests ready for review, failed runs needing intervention, and completion summaries.
- Hermes should not mirror every meaningful GitHub comment into Discord.
- Hermes may create repositories only for explicit **New Repository Requests**, and v1 should default those repositories to private under a dedicated GitHub owner or organization.
- The **Repository Creation Scope** for v1 is only the **Hermes GitHub Organization** `kvarnberg-labs`.
- The **New Repository Visibility Default** is private unless the **New Repository Request** explicitly asks for public visibility.
- **New Repository Initialization** should create a minimal `README.md` and an appropriate `.gitignore` based on the requested stack.
- **New Repository Initialization** should not add a license unless the **New Repository Request** explicitly asks for one.
- The **New Repository CI Default** is no CI unless the **New Repository Request** specifies a stack and test or validation command clearly enough to justify CI.
- The **Target Repository Scope** for v1 is any repository in the **Hermes GitHub Organization** that the **Hermes GitHub App** can access; there is no separate target-repository allowlist.
- Existing repositories outside the **Hermes GitHub Organization** remain governed by the **External Repository Boundary**.
- Each hourly run should perform **Queue Reconciliation** for non-terminal Hermes Work Items before starting new implementation work.
- **Queue Reconciliation** should inspect `hermes-clarifying`, `hermes-review`, and `hermes-blocked` issues for new answers, review feedback, merged pull requests, or user actions.
- **Queue Ordering** should start the oldest ready **Hermes Work Item** first.
- User comments on an **Implementation Pull Request** should enter the **Review Comment Fast Path** rather than waiting only for the hourly **Work Trigger Cadence**.
- The **Review Comment Fast Path** should respond to review comments by making changes, asking a clarifying question, or explaining why it is blocked, then requesting review again when ready.
- The **Fast Poll Preflight** should poll GitHub metadata only and must not call Hermes or incur model/provider cost when there are no new actionable comments or review updates.
- The **Fast Poll Preflight** should invoke Hermes only after detecting new actionable user input on an active **Implementation Pull Request**.
- The **Fast Poll Cadence** should be every 5 minutes.
- **Fast Poll Cursor State** should be stored under `/opt/data/hermes-autonomy/state/` on the shared Hermes PVC.
- **Fast Poll Cursor State** should not be stored in GitHub labels, GitHub comments, or Kubernetes Lease annotations.
- The hourly **Hermes CLI Work Trigger** should derive **Hermes Work Item** truth from GitHub each run.
- **Autonomy Local State** may store timeout counters, lock-adjacent metadata, workspace cleanup timestamps, and **Fast Poll Cursor State**.
- **Autonomy Local State** must not become a hidden local work queue or replace GitHub labels, issues, comments, and pull requests as the work source of truth.
- The **Fast Poll Trigger** should be a separate Kubernetes CronJob from the hourly **Hermes CLI Work Trigger**.
- The **Fast Poll Trigger** may reuse the same image, ConfigMap, Secret, and PVC as the **Hermes CLI Work Trigger**.
- The hourly **Hermes CLI Work Trigger** should use a queue-scanner **Hermes Trigger Prompt Template** for bootstrap, reconciliation, and starting the oldest ready item.
- The **Fast Poll Trigger** should use a review-response **Hermes Trigger Prompt Template** that handles only the detected review/comment event for an existing **Implementation Pull Request**.
- The review-response **Hermes Trigger Prompt Template** must not start new **Hermes Work Items**.
- **Prompt Template Delivery** should use Kubernetes ConfigMaps managed in this GitOps repository.
- **Hermes Trigger Prompt Templates** should not be baked into the Hermes image for v1.
- Hermes should follow the **Work Item Comment Policy** by commenting only on meaningful state changes, not on every hourly run.
- Meaningful comment events include clarification asked, work started, pull request opened or updated, blocked, review requested, merged/done, or failed run needing attention.
- Hermes may create, update, and respond to pull requests, but it must not exercise **Merge Authority**.
- The **Pull Request Branch Source** should be a branch in the target repository when the **Hermes GitHub App** has write access.
- The **Pull Request Branch Source** should use a fork or import workflow only for external repositories where direct branch push is unavailable and the user explicitly approves that workflow.
- **Hermes Branch Naming** should use `hermes/issue-<number>-<short-slug>` for implementation branches.
- **Hermes Commit Identity** should use normal bot-authored commits for v1, without commit signing.
- **Hermes Commit Identity** should not impersonate the user as commit author.
- **Hermes Commit Hygiene** should squash or rewrite noisy Hermes-authored WIP commits before opening or updating a pull request when that improves reviewability.
- **Hermes Commit Hygiene** should preserve meaningful commit history when the commits are already coherent.
- **Hermes Commit Hygiene** should not rewrite a branch after user review has started unless the review explicitly asks for it.
- Autonomous coding work should use a **Hermes CLI Work Trigger** rather than `kubectl exec` into the live **Hermes Gateway** pod.
- GitHub autonomy should use **Existing Gateway Enablement**; no additional long-running Kubernetes Deployment should be required.
- The **Hermes CLI Work Trigger** should mount the **Shared Hermes Home** so scheduled work can share durable Hermes state and workspaces with the **Hermes Gateway**.
- The **Kubernetes Permission Boundary** for v1 grants no broad Kubernetes API permissions to the **Hermes CLI Work Trigger** or **Fast Poll Trigger**.
- The **Kubernetes Permission Boundary** permits **Kubernetes Lease Coordination** for the shared **Implementation Lock** only.
- The **Hermes CLI Work Trigger** should otherwise rely on mounted ConfigMap, Secret, and PVC data plus GitHub APIs, not cluster mutation permissions.
- The **Network Access Boundary** should allow outbound HTTPS to GitHub, package registries, official documentation sites, and model/provider APIs.
- The **Network Access Boundary** should not expose the **Hermes CLI Work Trigger** as an inbound service.
- Future Kubernetes NetworkPolicies should model the **Network Access Boundary** as controlled egress, not public ingress.
- The **Hermes Workspace Root** should be `/opt/data/workspaces/github/<owner>/<repo>` for GitHub repositories.
- The **Workspace Disk Soft Cap** should warn and send a **Review Ping** when `/opt/data/workspaces` exceeds 7GiB on the current 10GiB Hermes PVC.
- The **Workspace Disk Soft Cap** should block new implementation work when `/opt/data/workspaces` exceeds 9GiB until cleanup or PVC expansion.
- Hermes must not expand the Hermes PVC automatically; PVC expansion requires explicit user approval through the **GitOps Change Boundary**.
- Hermes should keep a durable **Hermes Repo Workspace** per repository and create issue-specific **Issue Worktrees** and branches for implementation.
- When a **Hermes Work Item** already has an open **Implementation Pull Request**, Hermes should update that pull request and its branch rather than opening a duplicate pull request.
- Hermes should leave an **Implementation Pull Request** alone unless the issue, pull request comments, or review comments require changes or user attention.
- Every **Implementation Pull Request** should follow the **Pull Request Review Contract** by listing each acceptance criterion, the corresponding change, and the validation performed.
- The **Validation Evidence Style** should include command names and concise pass/fail summaries by default.
- The **Validation Evidence Style** should include short error excerpts only when validation fails or is skipped; full logs should be attached or commented only when needed.
- Hermes should use `hermes-review` while waiting for user review on an **Implementation Pull Request**.
- Hermes should perform **Work Item Completion** only after observing that the linked **Implementation Pull Request** has been merged.
- **Work Item Completion** should comment with the merged pull request, apply `hermes-done`, and close the **Hermes Work Item**.
- **Remote Branch Cleanup** should delete a remote `hermes/issue-*` branch after merge only when Hermes owns that branch.
- **Issue Worktree Retention** should keep completed **Issue Worktrees** for 7 days after **Work Item Completion**, then prune them.
- **Issue Worktree Retention** should not delete the durable **Hermes Repo Workspace**.
- **Abandoned Worktree Retention** should keep **Issue Worktrees** for 14 days after the **Hermes Work Item** is closed without merge, then prune them.
- If a **Hermes Work Item** reopens during **Abandoned Worktree Retention**, Hermes should resume from the existing **Issue Worktree** only if the workspace state is safe to continue.
- The **Work Trigger Cadence** should be hourly, with overlapping CronJob runs forbidden and each run bounded by an execution deadline.
- The **Work Trigger Deadline** should be 45 minutes for the hourly **Hermes CLI Work Trigger**.
- If the **Work Trigger Deadline** is reached during active work, **Timed-Out Work Recovery** should resume the same **Hermes Work Item** on the next run when the **Issue Worktree** and branch are recoverable.
- **Timed-Out Work Recovery** should mark a **Blocked Work Item** after three consecutive timeouts on the same **Hermes Work Item**, or sooner when the workspace state is unsafe to continue.
- Hermes may scan and clarify multiple **Hermes Work Items**, but it should run only one **Active Implementation** at a time.
- The hourly **Hermes CLI Work Trigger** and **Fast Poll Trigger** must share one **Implementation Lock**.
- The **Implementation Lock** should use **Kubernetes Lease Coordination** in the Hermes namespace rather than a PVC lock file.
- The **Implementation Lock** should be one global Lease for all repositories in v1.
- **Kubernetes Lease Coordination** should renew the Lease periodically while a trigger owns the **Implementation Lock**.
- **Stale Lease Recovery** should allow the next trigger run to acquire the **Implementation Lock** after the Lease expires.
- **Stale Lease Recovery** must not steal an actively renewed Lease.
- If the **Implementation Lock** is held, the **Fast Poll Trigger** should record the actionable review event for the active work item instead of starting concurrent implementation.
- If the **Implementation Lock** is held by the **Fast Poll Trigger**, the hourly **Hermes CLI Work Trigger** must not start a new implementation.
- Strava and Garmin access should be deferred to a separate **Training Data Connector** phase, not mixed into the v1 GitHub autonomy rollout.

## Example Dialogue

> **Dev:** "Should Hermes be exposed through a web UI first?"
> **Domain expert:** "No - v1 needs a **Remote Chat Surface**, and that surface is the **Discord Bot**."
> **Dev:** "Can I put the Telegram token in the ConfigMap while testing?"
> **Domain expert:** "No - credentials belong in the **Hermes Secret**."
> **Dev:** "Can I put the OpenCode Go API key in the ConfigMap while testing?"
> **Domain expert:** "No - model-provider credentials belong in the **Hermes Secret**."
> **Dev:** "Can Hermes use any secret available in the cluster while working on tickets?"
> **Domain expert:** "No - the **Hermes Runtime Secret Boundary** limits Hermes to its operating credentials unless a scoped repo-specific secret is explicitly added later."
> **Dev:** "Can Hermes discover new work from Forgejo issues?"
> **Domain expert:** "Not as the remote intake path - Forgejo is LAN-local, so new autonomous work starts as a **Hermes Work Item** in the **GitHub Work Queue**."
> **Dev:** "Should the GitHub Work Queue be public?"
> **Domain expert:** "No - the **Work Queue Visibility** is private because issues can contain operational context."
> **Dev:** "If a user-authored **Hermes Work Item** is labeled `hermes-ready`, should Hermes fill in missing details itself?"
> **Domain expert:** "No - the **Work Readiness Gate** starts processing; unclear work enters the **Clarification Loop** before code changes."
> **Dev:** "Can Hermes start implementation without acceptance criteria?"
> **Domain expert:** "No - the **Acceptance Criteria Gate** requires clear expected outcomes before code changes."
> **Dev:** "Should Hermes track detailed implementation state in labels?"
> **Domain expert:** "No - **Work State Labels** expose coarse queue state, and comments carry the detail."
> **Dev:** "Should Hermes mark an issue blocked on the first failing test?"
> **Domain expert:** "No - a **Blocked Work Item** requires a real blocker such as missing information, unavailable credentials, unresolved permissions, unsafe operations, or a failure Hermes cannot diagnose after one focused attempt."
> **Dev:** "Can Hermes run arbitrary commands from repository docs?"
> **Domain expert:** "No - the **Command Safety Boundary** allows repo-local build and test commands, but destructive or externally mutating commands require approval."
> **Dev:** "Can Hermes use Docker Compose to validate a repo?"
> **Domain expert:** "Yes - the **Container Validation Boundary** allows repo-defined, short-lived local validation services."
> **Dev:** "Can Hermes publish a Docker image while validating?"
> **Domain expert:** "No - image publishing and registry pushes need an explicit work item and user approval."
> **Dev:** "Can Hermes update dependencies while working on an unrelated ticket?"
> **Domain expert:** "No - the **Dependency Change Boundary** allows dependency changes only when requested or when they are the narrowest necessary fix."
> **Dev:** "Can Hermes install project dependencies to run tests?"
> **Domain expert:** "Yes - the **Dependency Install Boundary** allows repo-local installs using the existing package manager and lockfile conventions."
> **Dev:** "Can Hermes install global tools while implementing?"
> **Domain expert:** "No - global tools and system package changes require an explicit work item and user approval."
> **Dev:** "If Hermes must choose a dependency version, should it use latest?"
> **Domain expert:** "No - the **Dependency Version Default** is the LTS or stable supported line unless the work item explicitly asks otherwise."
> **Dev:** "Can Hermes browse external documentation while implementing?"
> **Domain expert:** "Yes - **External Documentation Use** is allowed when local context is insufficient, with official docs preferred and material sources cited."
> **Dev:** "Can Hermes work on the homelab GitOps repository?"
> **Domain expert:** "Yes - Hermes may prepare pull requests, but the **GitOps Change Boundary** keeps deployment, deletion, secret rotation, and production mutation behind user review or explicit approval."
> **Dev:** "Can Hermes ask for secret values in a GitHub issue?"
> **Domain expert:** "No - the **Sealed Secret Change Boundary** requires the approved sealing workflow and forbids raw secrets in GitHub."
> **Dev:** "Can Hermes push to this homelab repository just because the repository is public?"
> **Domain expert:** "No - the **External Repository Boundary** treats public external repositories as readable by default, not writable."
> **Dev:** "Can the clarification happen only in Discord?"
> **Domain expert:** "No - Hermes may send a **Review Ping** in Discord, but the answer belongs on the **Hermes Work Item**."
> **Dev:** "Should every GitHub status comment be mirrored to Discord?"
> **Domain expert:** "No - the **Attention Ping Threshold** limits **Review Pings** to events that need user attention."
> **Dev:** "Can Hermes create a new repo if it decides one would be useful?"
> **Domain expert:** "No - repo creation requires an explicit **New Repository Request** with name, visibility, and purpose."
> **Dev:** "Can Hermes create a new repo outside `kvarnberg-labs`?"
> **Domain expert:** "No - the v1 **Repository Creation Scope** is only the lab organization."
> **Dev:** "Should Hermes-created repositories be public by default?"
> **Domain expert:** "No - the **New Repository Visibility Default** is private unless the work item explicitly asks for public."
> **Dev:** "Should Hermes create new repositories empty?"
> **Domain expert:** "No - **New Repository Initialization** creates a minimal `README.md` and stack-appropriate `.gitignore`, but no license unless explicitly requested."
> **Dev:** "Should Hermes create CI for every new repository?"
> **Domain expert:** "No - the **New Repository CI Default** is no CI unless the work item clearly specifies the stack and validation command."
> **Dev:** "Do we need a separate allowlist for existing lab repositories?"
> **Domain expert:** "No - the v1 **Target Repository Scope** is the lab organization, constrained by the **Work Readiness Gate** and GitHub App permissions."
> **Dev:** "If several work items are ready, should Hermes pick the newest one?"
> **Domain expert:** "No - **Queue Ordering** starts with the oldest ready work item."
> **Dev:** "Should Hermes ignore existing clarifying or review items when starting new work?"
> **Domain expert:** "No - each hourly run performs **Queue Reconciliation** before starting the oldest ready work item."
> **Dev:** "If the user comments on a pull request, should Hermes wait for the next hourly run?"
> **Domain expert:** "No - **Review Comment Fast Path** handles user PR comments faster than normal hourly queue discovery."
> **Dev:** "Should the fast poller invoke Hermes every time it checks GitHub?"
> **Domain expert:** "No - **Fast Poll Preflight** checks GitHub cheaply and invokes Hermes only when new actionable review input exists."
> **Dev:** "Should the fast poller run every minute?"
> **Domain expert:** "No - the **Fast Poll Cadence** is every 5 minutes."
> **Dev:** "Should last-seen review comment IDs be stored in GitHub labels?"
> **Domain expert:** "No - **Fast Poll Cursor State** lives under `/opt/data/hermes-autonomy/state/` on the shared PVC."
> **Dev:** "Should the hourly scanner use a local queue file as the source of truth?"
> **Domain expert:** "No - GitHub remains source of truth, and **Autonomy Local State** stores only operational counters, cursors, and cleanup timestamps."
> **Dev:** "Should fast polling be folded into the hourly queue scanner?"
> **Domain expert:** "No - the **Fast Poll Trigger** is a separate CronJob with cheap GitHub-only preflight behavior."
> **Dev:** "Can the fast poll trigger use the same prompt as the hourly scanner?"
> **Domain expert:** "No - each trigger uses its own **Hermes Trigger Prompt Template**, and the fast prompt cannot start new work."
> **Dev:** "Should trigger prompts be baked into the Hermes image?"
> **Domain expert:** "No - **Prompt Template Delivery** uses GitOps-managed ConfigMaps."
> **Dev:** "Should Hermes add a heartbeat comment every hour?"
> **Domain expert:** "No - the **Work Item Comment Policy** allows comments only for meaningful state changes."
> **Dev:** "Can Hermes merge the pull request when checks pass?"
> **Domain expert:** "No - Hermes prepares the pull request and sends a **Review Ping**; the user keeps **Merge Authority**."
> **Dev:** "Should Hermes fork every repository before opening a pull request?"
> **Domain expert:** "No - the **Pull Request Branch Source** is a target-repository branch when Hermes has write access, with forks only for approved external workflows."
> **Dev:** "Can Hermes choose arbitrary branch names?"
> **Domain expert:** "No - **Hermes Branch Naming** uses `hermes/issue-<number>-<short-slug>`."
> **Dev:** "Should Hermes sign commits in v1?"
> **Domain expert:** "No - **Hermes Commit Identity** uses normal bot-authored commits without signing for v1."
> **Dev:** "Can Hermes leave noisy WIP commits in a pull request?"
> **Domain expert:** "No - **Hermes Commit Hygiene** lets Hermes clean up its own unreviewed branch before review, but not rewrite after review starts unless requested."
> **Dev:** "Should Hermes launch a separate worker job for each work item?"
> **Domain expert:** "No - v1 uses one scheduled **Hermes CLI Work Trigger** with one **Active Implementation** at a time."
> **Dev:** "Should the CronJob exec into the running gateway pod?"
> **Domain expert:** "No - the **Hermes CLI Work Trigger** should run `hermes chat -q` in a short-lived Hermes pod and mount the **Shared Hermes Home**."
> **Dev:** "Should we deploy a separate GitHub worker for the work queue?"
> **Domain expert:** "No - use **Existing Gateway Enablement** so the existing Hermes Deployment gets the tools and permissions it needs."
> **Dev:** "Should the CronJob get broad Kubernetes API permissions?"
> **Domain expert:** "No - the **Kubernetes Permission Boundary** permits only narrow **Kubernetes Lease Coordination** plus mounted runtime inputs and GitHub APIs."
> **Dev:** "Should the CronJob be exposed as a service so GitHub can call it?"
> **Domain expert:** "No - the **Network Access Boundary** requires outbound HTTPS, not inbound exposure."
> **Dev:** "Can Hermes implement two work items at the same time in the gateway pod?"
> **Domain expert:** "No - the **Hermes CLI Work Trigger** allows one **Active Implementation** at a time, though Hermes can clarify multiple work items."
> **Dev:** "Can the fast poll trigger work on a review while the hourly scanner starts another issue?"
> **Domain expert:** "No - both triggers share the **Implementation Lock**."
> **Dev:** "Should the implementation lock be a PVC file if Kubernetes Lease is cleaner?"
> **Domain expert:** "No - use **Kubernetes Lease Coordination** with narrow Lease permissions only."
> **Dev:** "Should Hermes use one Lease per repository?"
> **Domain expert:** "No - v1 uses one global **Implementation Lock** for all repositories."
> **Dev:** "If a CronJob pod dies while holding the Lease, should the lock require manual cleanup?"
> **Domain expert:** "No - **Stale Lease Recovery** lets the next run acquire the lock after the Lease expires."
> **Dev:** "Will the gateway remember work started by the scheduled trigger?"
> **Domain expert:** "Durable Hermes state can be shared through the **Shared Hermes Home**, but the scheduled run is not automatically part of the same live Discord session."
> **Dev:** "Should Hermes scan GitHub every few minutes?"
> **Domain expert:** "No - the **Work Trigger Cadence** is hourly, with overlapping runs forbidden."
> **Dev:** "Can a scheduled Hermes run continue indefinitely?"
> **Domain expert:** "No - the **Work Trigger Deadline** is 45 minutes for the hourly trigger."
> **Dev:** "If a run times out, should Hermes immediately mark the issue blocked?"
> **Domain expert:** "No - **Timed-Out Work Recovery** resumes recoverable work next hour and blocks only after repeated timeouts or unsafe state."
> **Dev:** "How many repeated timeouts should block a work item?"
> **Domain expert:** "Three consecutive timeouts on the same **Hermes Work Item**."
> **Dev:** "Can Hermes clone repositories wherever the current process happens to start?"
> **Domain expert:** "No - GitHub checkouts belong under the **Hermes Workspace Root** at `/opt/data/workspaces/github/<owner>/<repo>`."
> **Dev:** "Can Hermes keep cloning until the PVC fills?"
> **Domain expert:** "No - the **Workspace Disk Soft Cap** warns at 7GiB and blocks new implementation at 9GiB on the current 10GiB PVC."
> **Dev:** "Can Hermes expand the PVC automatically when the workspace is too large?"
> **Domain expert:** "No - PVC expansion requires explicit approval through the **GitOps Change Boundary**."
> **Dev:** "Should Hermes edit the durable repository checkout directly?"
> **Domain expert:** "No - Hermes keeps a **Hermes Repo Workspace** and implements each issue in an **Issue Worktree** on its own branch."
> **Dev:** "If an issue already has a pull request, should Hermes open another one?"
> **Domain expert:** "No - the existing **Implementation Pull Request** is the active implementation record."
> **Dev:** "Can Hermes summarize a pull request without mapping acceptance criteria?"
> **Domain expert:** "No - the **Pull Request Review Contract** maps each criterion to changes and validation."
> **Dev:** "Should Hermes paste full test logs into every pull request?"
> **Domain expert:** "No - the **Validation Evidence Style** is command names plus concise pass/fail summaries, with excerpts only for failures or skipped validation."
> **Dev:** "Should Hermes close a work item when it opens a pull request?"
> **Domain expert:** "No - **Work Item Completion** happens only after Hermes observes the linked **Implementation Pull Request** merged."
> **Dev:** "Should merged Hermes branches stay on the remote forever?"
> **Domain expert:** "No - **Remote Branch Cleanup** deletes Hermes-owned `hermes/issue-*` branches after merge."
> **Dev:** "Should completed worktrees be deleted immediately?"
> **Domain expert:** "No - **Issue Worktree Retention** keeps completed worktrees for 7 days, then prunes them while keeping the durable repo workspace."
> **Dev:** "Should worktrees for issues closed without merge be pruned immediately?"
> **Domain expert:** "No - **Abandoned Worktree Retention** keeps them for 14 days and resumes only if reopened safely."
> **Dev:** "Can I just give Hermes my personal GitHub token?"
> **Domain expert:** "No - v1 uses a **Hermes GitHub App** installed on the **Hermes GitHub Organization**."
> **Dev:** "Can Hermes infer missing repository details from a free-form issue?"
> **Domain expert:** "No - each **Hermes Work Item** follows the **Work Item Template**, and missing required fields enter the **Clarification Loop**."
> **Dev:** "Should labels and issue templates be created manually?"
> **Domain expert:** "No - Hermes owns **Work Queue Bootstrap** for the GitHub Work Queue."
> **Dev:** "Should queue bootstrap be a separate one-time command?"
> **Domain expert:** "No - **Work Queue Bootstrap** is idempotent and runs at the start of each hourly scan."
> **Dev:** "If the work-queue repository is missing, should Hermes wait for manual setup?"
> **Domain expert:** "No - **Work Queue Bootstrap** creates `kvarnberg-labs/hermes-work-queue` when permissions allow, otherwise Hermes pings for permission changes."
> **Dev:** "Should Strava and Garmin tokens be added while enabling GitHub work?"
> **Domain expert:** "No - training data belongs in a later **Training Data Connector** phase."

## Flagged Ambiguities

- "Access Hermes remotely" could mean a public web UI, an API endpoint, or a messaging integration; resolved for v1 as a **Discord Bot** remote chat surface.
- "Credentials" covers both the Telegram bot token and model-provider keys; resolved as **Hermes Secret** material, not ConfigMap data.
- "Hermes secrets" could mean operating credentials or arbitrary project secrets; resolved as the **Hermes Runtime Secret Boundary**.
- "Model provider" could mean a direct API key provider or a subscription provider; resolved for v1 as **OpenCode Go Account**.
- "Messaging service" was initially resolved as Telegram, then changed because the user decided to delete Telegram; v1 is now Discord.
- "Tickets" could mean Discord messages, Forgejo issues, GitHub issues, Linear issues, or markdown tasks; resolved as **Hermes Work Items** in a **GitHub Work Queue**.
- "Work queue visibility" could mean public for convenience or private for operational safety; resolved as private **Work Queue Visibility**.
- "Repository access" covers both work discovery and code mutation; resolved as separate concepts: **Remote Work Intake** and **Code Host**.
- "GitHub access" could mean a personal token or an app identity; resolved as the **Hermes GitHub App** installed on the **Hermes GitHub Organization**.
- "Issue body" could mean free-form intent or structured input; resolved as the **Work Item Template** with required fields.
- "Queue setup" could mean one-time manual labels and templates or Hermes-owned setup; resolved as **Work Queue Bootstrap**.
- "Ready" could mean ready to implement or ready for Hermes to inspect; resolved as the **Work Readiness Gate**, which requires `hermes-ready`, authorship by `JohanMillberg`, and template validity while still requiring the **Clarification Loop** for ambiguity.
- "Acceptance criteria" could mean optional notes or a true implementation gate; resolved as the **Acceptance Criteria Gate**.
- "Issue state" could mean project-board columns, comments only, or labels; resolved as **Work State Labels** plus detailed GitHub comments.
- "Blocked" could mean any obstacle or only user-actionable blockers; resolved as **Blocked Work Item** for user-actionable blockers after reasonable autonomous effort.
- "Run commands" could mean unrestricted shell authority or only repo-local validation; resolved as the **Command Safety Boundary**.
- "Use containers" could mean local validation or registry/deploy mutation; resolved as the **Container Validation Boundary**.
- "Dependency updates" could mean opportunistic modernization or ticket-scoped necessity; resolved as the **Dependency Change Boundary**.
- "Dependency install" could mean repo-local project install or global/system mutation; resolved as the **Dependency Install Boundary**.
- "Dependency version" could mean latest or LTS; resolved as the **Dependency Version Default**.
- "Use docs" could mean untracked browsing or source-backed implementation; resolved as **External Documentation Use** with official-source preference and citations when material.
- "Work on homelab" could mean preparing GitOps pull requests or directly changing the cluster; resolved as the **GitOps Change Boundary** with user-controlled merge and explicit approval for production mutation.
- "Secret change" could mean raw secret transport through GitHub or sealed-secret workflow; resolved as the **Sealed Secret Change Boundary**.
- "External repo" could mean readable public context or writable target repository; resolved as the **External Repository Boundary**.
- "Ping me" could mean the decision record lives in chat; resolved as a **Review Ping**, with decisions recorded on the **Hermes Work Item** or pull request.
- "Ping me" could also mean every GitHub event is sent to Discord; resolved as the **Attention Ping Threshold** for user-actionable events only.
- "Create a repo" could mean an implementation detail chosen by Hermes or an explicit user request; resolved as a **New Repository Request** only when stated by the user.
- "Where to create repos" could mean any owner Hermes can access; resolved as **Repository Creation Scope** limited to `kvarnberg-labs`.
- "New repo visibility" could mean public by default or private by default; resolved as private **New Repository Visibility Default** unless explicitly public.
- "New repo initialization" could mean empty, fully scaffolded, or minimal files; resolved as **New Repository Initialization** with README and `.gitignore`, but no inferred license.
- "New repo CI" could mean automatic workflows or explicit stack-based workflows; resolved as **New Repository CI Default** with no CI unless justified by the work item.
- "Allowed repos" could mean a separate allowlist or the whole lab organization; resolved as **Target Repository Scope** covering lab repositories the App can access.
- "Scan tickets" could mean only looking for new ready issues or also updating current issue states; resolved as **Queue Reconciliation** before **Queue Ordering**.
- "Which ticket first" could mean newest, oldest, priority labels, or random; resolved as **Queue Ordering** with oldest ready first.
- "Respond to review" could mean waiting for the hourly queue scan or using a faster reaction path; resolved as **Review Comment Fast Path** for user pull request comments.
- "Fast polling" could mean frequent model calls or cheap GitHub checks; resolved as **Fast Poll Preflight** with no Hermes invocation unless actionable updates exist.
- "Fast poll cadence" could mean every minute, 2 minutes, or 5 minutes; resolved as 5-minute **Fast Poll Cadence**.
- "Fast poll state" could mean GitHub labels, Lease annotations, or local state; resolved as **Fast Poll Cursor State** on the shared PVC.
- "Autonomy state" could mean hidden local queue or operational metadata; resolved as **Autonomy Local State** with GitHub as work source of truth.
- "Fast poll deployment" could mean a separate long-running service or a separate CronJob; resolved as **Fast Poll Trigger**.
- "Trigger prompt" could mean one shared prompt or trigger-specific prompts; resolved as **Hermes Trigger Prompt Template** per trigger.
- "Prompt delivery" could mean baked image content or reviewable GitOps ConfigMaps; resolved as **Prompt Template Delivery** through ConfigMaps.
- "Status updates" could mean hourly heartbeats or meaningful audit events; resolved as the **Work Item Comment Policy**.
- "Create PRs" could imply merge permission; resolved separately through **Merge Authority**, which stays with the user.
- "PR branch source" could mean target branches or always-fork; resolved as **Pull Request Branch Source**.
- "Branch naming" could mean arbitrary branch names or issue-linked names; resolved as **Hermes Branch Naming**.
- "Branch cleanup" could mean deleting local worktrees or remote branches; resolved as **Remote Branch Cleanup** plus separate worktree retention.
- "Commit identity" could mean signed commits, user-authored commits, or bot-authored commits; resolved as **Hermes Commit Identity** with unsigned bot-authored commits in v1.
- "Commit cleanup" could mean preserving every WIP commit or rewriting reviewed work; resolved as **Hermes Commit Hygiene** for unreviewed Hermes-authored branches only.
- "Autonomous worker" could mean a separate long-running controller, `kubectl exec` into the gateway pod, or a scheduled CLI run; resolved as the **Hermes CLI Work Trigger** using `hermes chat -q`.
- "Give Hermes access" could mean adding a separate controller; resolved as **Existing Gateway Enablement** against the current Hermes runtime image, config, secret, and PVC.
- "Kubernetes access" could mean mounted runtime inputs, narrow Lease coordination, or broad API permissions; resolved as the **Kubernetes Permission Boundary** with only **Kubernetes Lease Coordination** added in v1.
- "Network access" could mean outbound egress or inbound service exposure; resolved as the **Network Access Boundary** with controlled outbound HTTPS and no inbound CronJob service.
- "Concurrency lock" could mean separate trigger-local locks, per-repo locks, a PVC file, or one Kubernetes Lease; resolved as one global **Implementation Lock** using **Kubernetes Lease Coordination**.
- "Stale lock" could mean manual cleanup or lease expiry; resolved as **Stale Lease Recovery** without stealing active leases.
- "Main agent memory" could mean the same live Discord session or shared durable state; resolved as **Shared Hermes Home** for durable state, not live-session continuity.
- "Discover new tickets" could imply real-time webhooks or frequent polling; resolved as an hourly **Work Trigger Cadence**.
- "Run timeout" could mean unbounded work or a hard execution cap; resolved as a 45-minute **Work Trigger Deadline**.
- "Timeout handling" could mean immediate blocked state or resumable work; resolved as **Timed-Out Work Recovery** with a three-consecutive-timeout threshold.
- "Clone the repos" could mean temporary checkouts or durable workspaces; resolved as the **Hermes Workspace Root** under `/opt/data/workspaces/github/<owner>/<repo>`.
- "Workspace disk limit" could mean no limit, hard deletion, or warning threshold; resolved as the **Workspace Disk Soft Cap**.
- "PVC expansion" could mean automatic storage growth or explicit infrastructure approval; resolved as explicit approval through the **GitOps Change Boundary**.
- "Working copy" could mean editing one shared checkout or using per-issue git worktrees; resolved as a durable **Hermes Repo Workspace** with one **Issue Worktree** per implementation.
- "Create PRs" could mean opening a new pull request every run; resolved as one **Implementation Pull Request** per **Hermes Work Item**.
- "PR body" could mean a loose summary or an acceptance-criteria report; resolved as the **Pull Request Review Contract**.
- "Validation evidence" could mean full logs or concise command summaries; resolved as **Validation Evidence Style**.
- "Done" could mean PR opened or PR merged; resolved as **Work Item Completion** only after merge.
- "Worktree cleanup" could mean immediate deletion or indefinite retention; resolved as 7-day **Issue Worktree Retention** after completion.
- "Abandoned worktree cleanup" could mean immediate deletion or indefinite retention; resolved as 14-day **Abandoned Worktree Retention**.
- "Working on tickets" could mean clarifying, scanning, or editing code; resolved by allowing many pending clarifications but only one **Active Implementation**.
- "Personal resources" includes both code hosts and training providers; resolved by doing GitHub autonomy first and deferring training providers to a **Training Data Connector**.
