# specd

Spec-driven development workflow for AI coding assistants, packaged as a Claude Code plugin.
Read "spec'd". Stack-agnostic; lives inside a code repo (`.specd/`) or wraps around it.

Status: M1 (init, onboard, scaffold in embedded mode). See `docs/PLAN.md` for the plan of record and milestones.

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

In a repository: `/specd:init` (mechanical setup, one short interview), then `/specd:onboard`
once per docs section with `/clear` in between, then `/specd:scaffold` on a greenfield repo.
Each command ends with a handoff block naming the next one.

## Layout

```
.claude-plugin/   plugin.json, marketplace.json
skills/<name>/    SKILL.md spine + references/ + templates/
skills/_shared/   blocks referenced by many skills
agents/           subagent definitions
hooks/            gate enforcement, secrets guardrail
scripts/          deterministic helpers: validate, resolve-paths, detect-repo,
                  detect-commands, config-set, init-workspace
docs/             PLAN.md, DECISIONS.md, AUTHORING.md
```

## Contributing

Read `docs/AUTHORING.md` before adding or changing a skill. Run `scripts/validate` and
`claude plugin validate .` before committing. Decisions go to `docs/DECISIONS.md`.
