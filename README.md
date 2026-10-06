# specd

Spec-driven development workflow for AI coding assistants, packaged as a Claude Code plugin.
Read "spec'd". Stack-agnostic; lives inside a code repo (`.specd/`) or wraps around it.

Status: M2 (the full feature flow in embedded mode) is built and awaiting its first real
feature. See `docs/PLAN.md` for the plan of record and milestones.

## Install

From GitHub, inside Claude Code:

```
/plugin marketplace add gpntechdev/specd
/plugin install specd@specd
```

From a local checkout, for development:

```
claude --plugin-dir /path/to/specd
```

Then `/reload-plugins` after edits. Skills appear as `/specd:<name>`.

## Getting started

Set up once per repository, then run the flow per feature. Every command ends with a handoff
block naming the next one; `/clear` between commands is the norm, and `/specd:status` shows
where each feature stands and what to run next.

```
brew install gh && gh auth login          # once per machine (glab for GitLab)
cd <repo> && claude --plugin-dir /path/to/specd

/specd:init                               # mechanical setup, one short interview
/specd:onboard                            # one docs section per run: project, architecture,
                                          # conventions, data-models, decisions
/specd:scaffold                           # greenfield only

/specd:start <feature> --from <file>      # or paste the ticket text; brief + triage, branch
/specd:specify <feature>                  # spec.md, gate G1
/specd:design <feature>                   # design.md by the architect agent, gate G2
/specd:tasks <feature>                    # tasks.md with the coverage matrix, gate G3
/specd:implement <feature> [T1-T3]        # implementer agent per task, WIP commit per task,
                                          # gate G4 at the configured granularity
/specd:review <feature>                   # reviewer agent in a fresh context; fixes become
                                          # tasks and send you back to implement
/specd:verify <feature>                   # evidence for every acceptance criterion
/specd:docs <feature>                     # feature record, decisions, doc deltas, before the PR
/specd:deliver <feature>                  # checks, gate G5, optional squash; you choose push / PR
/specd:feedback <feature>                 # PR comments -> fix tasks (back to implement) or replies
/specd:close <feature>                    # retention: remove or keep the spec folder, then merge
```

`/specd:config key=value` changes a setting later (gate granularity, TDD, retention, PR
host, model roles). `git.authority` in `specd.yml` caps what the flow may do with git:
`none`, `commit`, `push` or `draft_pr` (default).

## Layout

```
.claude-plugin/   plugin.json, marketplace.json
skills/<name>/    SKILL.md spine + references/ + templates/
skills/_shared/   blocks referenced by many skills: gates, handoff, state, paths, sources
agents/           explorer, researcher, architect, implementer, reviewer
hooks/            secrets guardrail
scripts/          deterministic helpers: validate, resolve-paths, detect-repo,
                  detect-commands, config-set, init-workspace, state, run-checks, squash-wip,
                  pr-comments
docs/             PLAN.md, DECISIONS.md, AUTHORING.md
```

## Contributing

Read `docs/AUTHORING.md` before adding or changing a skill. Run `scripts/validate` and
`claude plugin validate .` before committing. Decisions go to `docs/DECISIONS.md`.
