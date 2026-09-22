# Path resolution

**Reference-only.** Not a skill. Every skill locates files through these roots; no skill
hardcodes a path.

## Roots

| Root | Meaning | Embedded default | Wrapper default |
|---|---|---|---|
| `workspace_root` | folder holding `specd.yml` | `<repo>/.specd` | `<wrapper>` |
| `code_root` | the code | `<repo>` (`paths.code_root: ".."`) | `<wrapper>/<project>` |
| `docs_root` | persistent knowledge | `<workspace_root>/docs` | `<workspace_root>/docs`, or inside the code repo if chosen at init |
| `spec_root` | working set | `<workspace_root>/spec` | `<workspace_root>/spec` |

`paths.*` in `specd.yml` are relative to `workspace_root`. Absolute paths work but do not survive
a clone, so `init` never writes them.

## Finding specd.yml

From the current directory upward, at each level: `<dir>/specd.yml` (wrapper), then
`<dir>/.specd/specd.yml` (embedded). The first hit wins. No hit means the workspace is not
initialised: stop and suggest `/specd:init`.

## The script

Run the helper instead of reasoning about paths:

```
"${CLAUDE_PLUGIN_ROOT}/scripts/resolve-paths" [--from DIR]
```

It prints one JSON object: `mode`, `project`, `workspace_root`, `code_root`, `docs_root`,
`spec_root`, `config` (the file it read). All paths are absolute. Exit code 1 with a one-line
message when no `specd.yml` is found.

## Rules

- Name locations by root, never by literal folder: `<docs_root>/project.md`,
  `<spec_root>/<feature>/state.yml`, not `docs/project.md`.
- Code commits run `git -C <code_root>`. Spec and docs commits run in the repo that contains
  them: the wrapper repo in wrapper mode, the same repo in embedded mode. A skill decides by
  asking which repo a path belongs to, not by mode.
- Subagents receive resolved absolute paths in their prompt; they do not resolve on their own.
