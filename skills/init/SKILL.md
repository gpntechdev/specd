---
name: init
description: >
  Invoke whenever the user asks to set up, initialise, install or configure specd (or "the
  workflow") in a repository, or runs `/specd:init`. It does the mechanical setup only: detects
  greenfield or brownfield, git host, ticket convention and project commands with scripts,
  asks a short interview (retention, TDD, gate granularity, PR host, private mode), then
  writes `.specd/specd.yml`, the no-attribution setting, `.mcp.json` with Context7 and a
  pointer block in `CLAUDE.md`, and commits. It writes no project knowledge; that is
  `/specd:onboard`, which it hands off to.
---

# Skill: init

Creates the specd workspace in a repository, embedded mode. Everything it writes is either a
verbatim copy of a reference file or the answer to a question; nothing is generated from a
scan of the code. Wrapper mode arrives with M4: if asked for it, say so and stop.

## Inputs

- The current directory (the repository, or a folder inside it).
- [`../_shared/specd-yml.md`](../_shared/specd-yml.md): the config file the script copies.
- [`../_shared/no-attribution.md`](../_shared/no-attribution.md): the setting the script merges.
- [`../_shared/paths.md`](../_shared/paths.md): what the roots mean.
- `./templates/CLAUDE.md` and `./templates/lessons.md`, rendered by the script.

## Outputs

- `<repo>/.specd/specd.yml`, `<repo>/.specd/docs/` (with `lessons.md`), `<repo>/.specd/spec/`,
  `<repo>/.specd/sources/`.
- `<repo>/.claude/settings.json` (attribution off), `<repo>/.mcp.json` (Context7),
  `<repo>/CLAUDE.md` pointer block between `<!-- specd:begin -->` and `<!-- specd:end -->`.
- Private mode instead: `CLAUDE.local.md`, `.claude/settings.local.json`, three lines in
  `.git/info/exclude`, no `.mcp.json`.
- One commit `chore: initialise specd workspace` (none in private mode).

## Protocol

1. **Refuse a second init.** Run `"${CLAUDE_PLUGIN_ROOT}/scripts/resolve-paths"`. If it finds
   a config, say `already initialised at <config>; use /specd:config to change settings` and
   end with the handoff block.
2. **Detect.** Run `"${CLAUDE_PLUGIN_ROOT}/scripts/detect-repo"` and
   `"${CLAUDE_PLUGIN_ROOT}/scripts/detect-commands"`. If `git` is false, ask whether to run
   `git init`; on no, stop. Keep the two JSON results; they answer most of the interview.
3. **Interview**, one grouped question set with defaults prefilled, nothing that detection
   already answered:
   - retention for full-tier features: `distill` (default) / `clean` / `keep`;
   - TDD: off (default) / on;
   - gate granularity for implementation: `phase` (default) / `task` / `end`;
   - PR host: prefilled from `remote_host`; ask only when it is `none`;
   - ticket key: ask only when `ticket_key` is empty ("none" is a valid answer);
   - private mode: no (default) / yes, for a repo whose team should not see specd files.
4. **Create.** Run `"${CLAUDE_PLUGIN_ROOT}/scripts/init-workspace" --root <root> [--private]`.
   Show its `created`, `merged`, `skipped` and `warnings` lists as they came back.
5. **Configure.** Run `"${CLAUDE_PLUGIN_ROOT}/scripts/config-set" --file <config>` with the
   interview answers and the derived values: `project` (from `project_name`), `git.pr_host`,
   `git.ticket_key`, and each non-empty `commands.*` from `detect-commands`. Show the
   `old -> new` list; mark derived values so the user can correct them with `/specd:config`.
6. **Commit** (skip in private mode): stage exactly the paths the script created or merged
   plus `CLAUDE.md`, commit `chore: initialise specd workspace` on the current branch, no
   attribution line.
7. **Handoff** per [`../_shared/handoff.md`](../_shared/handoff.md). `Next`: `/specd:onboard`.
   Greenfield: add a second line "then `/specd:scaffold` once the architecture section is
   signed off". Last line: "project skills and MCP recommendations come with
   `/specd:toolsmith`, not yet available".

## Anti-patterns

- Regenerating or hand-editing `specd.yml`; `config-set` is the only writer.
- Putting knowledge into the CLAUDE.md block; it points at `<docs_root>/`, nothing more.
- Reading source files to "get a feel" for the repo; `detect-repo` already counted them.
- Asking a question whose answer is in the detection output.
- Committing files the script did not touch.
