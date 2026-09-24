# Decisions

Running log of decisions that change or refine `docs/PLAN.md`. Newest last. Each entry:
context (why it came up), decision, consequences. The plan stays the source for the design;
this file is the diff history of its judgement calls.

## 2026-09-22 M0: specd.yml schema is an annotated YAML reference

Context: the schema needed a home that both skills and humans read, before any init code exists.
Decision: `skills/_shared/specd-yml.md` holds the default file with a comment per key plus
reader and writer rules; no JSON Schema.
Consequences: one source, copied verbatim by `init`; machine validation of user config, if ever
needed, is a later script reading the same file.

## 2026-09-22 M0: principles text lives in `_shared`, the skill is a protocol spine

Context: PLAN 3.9 lists principles among shared blocks; the skill is also the trigger surface.
Decision: `skills/_shared/principles.md` is canonical; `skills/principles/SKILL.md` is a short
protocol that reads it. Other skills link the shared file, not the skill.
Consequences: one extra file read when the skill fires; no duplicated text.

## 2026-09-22 M0: scripts are stdlib-only Python 3

Context: macOS ships Python 3.9 without PyYAML; the validator and path helper must run anywhere.
Decision: `scripts/*` use the standard library only. Config is read by a flat-key parser that
covers the subset specd.yml uses; full YAML parsing is not needed.
Consequences: `specd.yml` stays flat (two levels, scalar values, simple lists) so the parser
keeps working. A key needing deeper nesting is a decision to revisit here.

## 2026-09-22 M0: path resolution is a script, not prose

Context: every skill resolves `workspace_root`, `code_root`, `docs_root`, `spec_root`.
Decision: `scripts/resolve-paths` prints them as JSON; `skills/_shared/paths.md` documents the
rules and tells skills to run it.
Consequences: skills never reason about folder layout; wrapper and embedded are one code path.

## 2026-09-22 M0: no-attribution is a workspace setting written by init

Context: a plugin-level `settings.json` supports only `agent` and `subagentStatusLine`, so the
plugin cannot switch attribution off for its users.
Decision: `skills/_shared/no-attribution.md` holds the `attribution` snippet `init` writes into
each workspace's `.claude/settings.json`; this repo carries the same snippet for itself.
Consequences: attribution is off only after `init`; deliver enforces the wording rules regardless.

## 2026-09-22 M0: `git.docs_repo` dropped from specd.yml

Context: PLAN 2 says wrapper mode chooses at init whether `docs/` lands in the wrapper or the
code repo. A separate key would duplicate what `paths.docs_root` already says.
Decision: `init` writes `paths.docs_root` to the chosen folder; a skill committing a docs change
picks the repo that contains the path.
Consequences: one key fewer; the repo choice is derived, never stated twice.

## 2026-09-22 M0: marketplace and plugin are both named `specd`

Context: the repo serves itself as a marketplace with one plugin.
Decision: marketplace `specd`, plugin `specd`, install as `specd@specd`; `source: "./"`.
Consequences: a second plugin in this repo would need a `plugins/` subfolder and a rename of
`source`; not planned.

## 2026-09-22 M0: layout folders appear when first used

Context: PLAN 4 lists `agents/` and `hooks/`; git does not track empty folders.
Decision: no placeholder files. `agents/` arrives with the first agent (M1), `hooks/` with the
gate hook (M2). The validator already checks both when present.
Consequences: the tree matches PLAN 4 only after M2; README shows the full layout anyway.

## 2026-09-22 M0: principles auto-invocation stays probabilistic until after M2

Context: Claude Code invokes a skill when the model judges its `description` relevant; nothing
forces it. In a headless test the same small coding request skipped the skill with a descriptive
wording and invoked it with an imperative one ("Invoke first, before any tool call, whenever...,
however small the task"). One sample each, so the wording helps but is not a guarantee.
Decision: keep description-driven invocation for ad-hoc coding requests. Inside the flow the
skills that write code (`implement`, `fix`, `scaffold`) link `_shared/principles.md` in their own
protocol, so the principles load deterministically there. No hook.
Consequences: a request outside the flow may occasionally run without the principles. Revisit
after M2, once the flow has shipped a real feature: if ad-hoc requests miss the skill often
enough to matter, add a `UserPromptSubmit` hook that reminds the model to invoke it, and record
that here.

## 2026-09-22 M1: toolsmith stays in M6; init only names it

Context: the M1 request listed `toolsmith` with the init commands. PLAN 3.5 calls it from init's
handoff, but its substance (allowlist search, quarantine, two-stage audit, `skills.lock`) is M6.
Decision: no `toolsmith` skill in M1. `init`, `onboard` and `scaffold` end their handoff with one
line naming `/specd:toolsmith` as a later step.
Consequences: a fresh workspace has Context7 and nothing else until M6; the handoff line is the
only place to update when the command lands.

## 2026-09-22 M1: secrets guardrail hook ships now, file tools only

Context: PLAN 3.5 lists the guardrail under `init`; the M0 entry above expected `hooks/` to
arrive with the gate hook in M2. The hook is plugin-level, so `init` has nothing to write for it.
Decision: `hooks/hooks.json` registers a `PreToolUse` hook on `Read|Edit|Write|MultiEdit|
NotebookEdit`; `hooks/secrets-guard` denies (exit 2) a fixed list of secret file patterns
(`.env*` minus example files, keys, certificates, credential stores). Bash inspection (`cat
.env`) is out of scope.
Consequences: `hooks/` exists before M2, amending the M0 entry. A leak through Bash is the
trigger to revisit; parsing shell is not worth it before one is observed.

## 2026-09-22 M1: onboard resumes from a draft marker, not a state file

Context: `onboard` must resume after `/clear` from the first unfinished section; files are the
state.
Decision: every onboard template starts with `<!-- specd:draft -->`. A section is unfinished when
its index file is missing or still carries the marker; sign-off removes the marker and commits.
No `onboard.yml`, no key in `specd.yml`.
Consequences: resume is a file check. Project kind is not persisted either: `detect-repo` runs
at the start of `onboard` and `scaffold`, and after scaffold the repo correctly reads as
brownfield.

## 2026-09-22 M1: scaffold builds inline; the reviewer pass arrives with M2

Context: PLAN 3.5 says scaffold runs as a quick-tier feature with a gate and a review; the quick
flow and the `reviewer` agent are M2/M3.
Decision: `scaffold` writes `<spec_root>/scaffold/{tasks.md,state.yml}` (tier `quick`,
`retention: clean`), passes G3 on the task list, builds in the main thread applying
`_shared/principles.md`, proves build/test/lint with at most three repairs per command, passes
G5 on the diff, records `commands.*` and commits. No reviewer in M1.
Consequences: the skeleton gets both gates but no fresh-context review until M2, when
`spec/scaffold/` is a closed quick feature that `close` or `spec-clean` remove like any other.

## 2026-09-22 M1: commit policy for init and onboard; private mode

Context: PLAN fixes git authority for features (up to a draft PR) but not for setup commands.
Decision: `init` commits once (`chore: initialise specd workspace`), `onboard` once per
signed-off section (`docs: onboard <section>`), `scaffold` once at G5. Private mode
(`init-workspace --private`) is the `.git/info/exclude` flag from PLAN 3.1 made concrete:
`.specd/`, `CLAUDE.local.md` and `.claude/settings.local.json` are excluded, `.mcp.json` is
skipped with a `claude mcp add` hint, and `init` does not commit.
Consequences: a normal workspace has a clean tree after each command; a private one never shows
specd files to the team and relies on the user's own MCP setup.

## 2026-09-22 M1: gates, handoff and state.yml are shared blocks from now

Context: AUTHORING 3 names the handoff format and gate protocol as shared blocks; `scaffold` is
the first step with gates and a `state.yml`.
Decision: `_shared/handoff.md` (the closing block), `_shared/gates.md` (show, ask, record in
`state.yml`, refuse when unapproved) and `_shared/state-yml.md` (flat two-level default file:
feature, tier, step, retention, gates G1..G5, tasks, timestamps) exist now. M2 extends
`state.yml` with triage signals and review rounds and records the keys here.
Consequences: every later step links these three files and states only its delta.

## 2026-09-22 M1: init-workspace owns every init file operation; config-set is the only writer

Context: init touches eight artifacts, all mechanical (copy, merge, marker insert), and
`specd.yml` must be patched key by key by both `init` and `config`.
Decision: `scripts/init-workspace` creates or merges all of them idempotently, including the
CLAUDE.md pointer block, and reports created/merged/skipped. `scripts/config-set` patches
`specd.yml` values in place, all-or-nothing, refusing list keys. `init` writes the detected
`commands.*` (marked derived) rather than leaving them empty.
Consequences: skills never edit config or settings files by hand; a bug in either script is
fixed once. Detected commands are visible and correctable in `specd.yml`.

## 2026-09-22 M1: explorer has no Bash; the caller passes the model role

Context: AUTHORING 10 keeps read-only agents without `Write`, `Edit` or `Bash`; AUTHORING 8
forbids model names outside `specd-yml.md`, which rules out `model:` in agent frontmatter.
Decision: `explorer` runs with `Read, Grep, Glob`; every git fact comes from `detect-repo` in
the main thread. `researcher` (`WebSearch`, `WebFetch`, Context7) is included minimal for the
greenfield architecture interview. The calling skill reads `models.<role>` from `specd.yml` and
passes it as the agent's model parameter.
Consequences: agents are portable across projects with different model mappings. If the
runtime ignores the per-call model parameter, the fallback is a validator allowlist for
`model:` lines in `agents/`, to be recorded here.

## 2026-09-22 M1: external pages snapshot to `<workspace_root>/sources/`

Context: onboard lets the user paste or point at wiki pages; PLAN 3.7 gives the snapshot and
refresh util to M5.
Decision: snapshots go to `<workspace_root>/sources/<slug>.md` with a provenance header and
redaction counted; `docs/` links and condenses, never copies. Redaction is a documented rule in
`skills/onboard/references/sources.md` until M5 provides the util.
Consequences: raw material is separated from curated knowledge and is never auto-loaded; M5's
refresh util has a folder to own.

## 2026-09-24 M1: config skill added; scaffolded repos are brownfield by state, not by count

Context: a review of M1 found two gaps. `init`, the CLAUDE.md template and the handoffs sent
users to `/specd:config`, which no milestone had built. And the entry above claiming that a
scaffolded repo "correctly reads as brownfield" was false: a skeleton has two or three source
files, under the `detect-repo` threshold of five, so onboard sections run after scaffold
(conventions, data-models, decisions, which its gate does not require) would have run as
interviews.
Decision: `skills/config` exists as a thin wrapper over `config-set`: show, ask, patch, say
when the value applies, no commit. `detect-repo` reports `scaffolded: true` when
`.specd/spec/scaffold/state.yml` has `step: closed` and then reports `kind: brownfield`
whatever the file count; the count stays the fallback for repos specd did not scaffold.
Consequences: every `/specd:config` reference resolves. Lowering the threshold was rejected
because a tiny real repo would then read as greenfield; the state file is the signal specd
itself wrote. A scaffold still in progress leaves the kind untouched.

## 2026-09-24 M2: branch at start, commit per step; `git.authority` caps deliver

Context: PLAN 3.3 gave `deliver` "branch, atomic commits, push, draft PR", which leaves every
spec file and all code uncommitted until the last step; M1 already commits per step.
Decision: `start` creates the feature branch; `specify`, `design` and `tasks` commit their
artifact after the gate; `implement` commits per task with the message fixed at G3; `deliver`
re-runs checks, passes G5, pushes and opens the draft PR. `git.authority` means: `none`, specd
runs no git write command at all (no branch, no commit); `commit`, branch and local commits,
deliver stops after G5; `push`, plus push; `draft_pr`, plus the PR.
Consequences: resume after `/clear` never loses work and two features never share a tree.
The PLAN 3.3 deliver row is read with this amendment.

## 2026-09-24 M2: tiers are recorded, every feature runs the full flow

Context: triage proposes trivial/quick/full, but the light flows and escalation are M3.
Decision: `start` writes `tier` and `triage.proposed`; the tier sets the default retention
and is shown by `status`; every step runs regardless of tier.
Consequences: a quick feature costs a full flow until M3; the state is already in place for
M3 to branch on.

## 2026-09-24 M2: `scripts/state` owns state.yml; new keys; feature resolution; no gate hook

Context: every step reads and patches `state.yml`, must know whether it may run, and must
find its feature after `/clear` without an argument. PLAN 3.9 and the M0 entry expected a
gate-enforcement hook.
Decision: `scripts/state` (init, get, set, check, find, list) is the only writer; `set` may
add `tasks.<id>` keys and takes `now` as a timestamp. New keys: `branch`, `pr`,
`triage.proposed`, `triage.signals`, `review.round`, `review.open`. Resolution order in
`state find`: argument, then the feature whose `branch` is the current git branch, then the
only open feature, else ask. `state check` encodes the step order and gate prerequisites; a
step may re-run an earlier artifact, which clears every later gate. No hook: a hook cannot
know which feature is active without a "current feature" file, which parallel features (M4)
would break. Amends the M0 entry a second time.
Consequences: skills never reason about order or edit state by hand. Revisit the hook with
worktrees in M4.

## 2026-09-24 M2: sources rules move to `_shared`; the brief is written in the main thread

Context: `start` snapshots files and pastes the same way `onboard` does; PLAN 3.3 says the
brief is written by an agent that treats fetched text as data.
Decision: `skills/_shared/sources.md` and `_shared/templates/source.md` replace the onboard
copies. For local files and pastes the main thread writes the brief: a paste is already in
context, and a file the user named is theirs. The isolating agent arrives with URL and MCP
sources in M5.
Consequences: one set of redaction rules; M5 adds the agent without touching the brief format.

## 2026-09-24 M2: critic hook points are marked comments in the spine

Context: the critic agent is M3, but `specify` and `design` must already have its place.
Decision: each spine carries one protocol step marked `<!-- specd:critic-hook: M3 -->` that
states the M3 behaviour and, until then, prints `critic: not available until M3`. M3 replaces
the marker line, nothing else moves.
Consequences: the user sees where the critic will run; the gate summary is unchanged.

## 2026-09-24 M2: the architect writes `design.md` itself; sections are selected before it runs

Context: AUTHORING 5 says subagents return bounded summaries, never file dumps; a design
draft is a whole file. PLAN 3.3 wants only the sections the feature needs.
Decision: `architect` gets `Write` for exactly one path, `<spec_root>/<feature>/design.md`,
writes the draft there and returns a summary (sections, decisions, open questions,
pointers) of at most 40 lines. The main thread selects the sections from a signal table in
`skills/design/references/sections.md` before dispatching, and the user can veto the
selection. The tier does not change the selection in this milestone.
Consequences: the draft never passes through the conversation; the validator keeps
`architect` out of the read-only set. A second writable path is a new decision.

## 2026-09-24 M2: `run-checks` runs the checks; implement continues after a G4 approval

Context: `implement`, `verify` and `deliver` all run the project checks and must not fill
the context with output. A G4 stop could end the run or let it continue.
Decision: `scripts/run-checks` calls `detect-commands`, runs each detected command in the
repo root and prints JSON with the exit code and the last 40 lines per check. After a G4
approval `implement` continues in the same context with the next task; the implementer's
summaries are small, so the context grows slowly. Every stop is resumable from disk.
Consequences: check output never exceeds a bounded tail; a user who wants a fresh context
rejects the gate (tasks stay `done`) and re-runs the command after `/clear`.

## 2026-09-24 M2: the reviewer reads a diff file; fixes become tasks; three rounds at most

Context: the `reviewer` is read-only (no Bash, M1 validator rule), yet it must see what
changed, including removed lines. Findings the user wants fixed must reach `implement`.
Decision: `review` writes `git diff <default>...HEAD` to `<spec_root>/<feature>/review.diff`,
the agent reads it, and the file is deleted before the commit. Each finding the user marks
`fix` becomes a task in a `Review fixes (round n)` phase of `tasks.md` with a new id and a
`commit:` line; `step` goes back to `implement`. A `missing` AC cannot be accepted. After
three rounds `review` refuses and the user decides by hand.
Consequences: fixes get the same gate, commit and evidence path as any task; the diff is
never committed; the validator keeps `reviewer` read-only.

## 2026-09-24 M2: verify fills the matrix in `tasks.md`; deliver degrades without a forge CLI

Context: PLAN 3.3 says `verify` fills the evidence column and lists no `verify.md`; the PR
step depends on `gh` or `glab`, which a machine may lack.
Decision: evidence lives in the coverage matrix of `tasks.md` (a passing named test, a
green check, or a dated manual note the user gave). `deliver` pushes and, when the host CLI
is missing or `git.pr_host` is `none`, prints the exact command and the compare URL and
stops at `push`; the PR URL, when obtained, is written to `state.yml` as `pr`. `close`
pushes its commit when authority ≥ `push`, so the PR carries the distilled docs.
Consequences: no extra artifact; a manual PR is a documented degradation, not a failure.

## 2026-09-24 M2: decision template moves to `_shared`; no `docs` step in M2

Context: `close` writes decision files with the same template `onboard` uses; the brief
listed no `doc-writer` or `docs` step.
Decision: `skills/_shared/templates/decision.md` is the one template, linked by both.
`close` writes `<docs_root>/features/<feature>.md` and the doc deltas; user-facing docs
beyond that are a later milestone.
Consequences: `onboard` and `close` cannot drift on the decision format.
