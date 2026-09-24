# Git for deliver

## What `git.authority` allows

| Value | start | implement | deliver |
|---|---|---|---|
| `none` | no branch | no commits | G5 view only; prints what it would do |
| `commit` | branch | commits | G5 view; stops before push |
| `push` | branch | commits | push the branch |
| `draft_pr` | branch | commits | push, then a draft PR or MR |

## Push

`git -C <code_root> push -u origin <branch>` where `branch` comes from `state.yml`. A remote
that rejects the push (no permission, protected name) is reported verbatim and the run stops
at G5 recorded; no retry with different flags.

## Draft PR or MR

| `git.pr_host` | Command |
|---|---|
| `github` | `gh pr create --draft --base <default> --head <branch> --title "<title>" --body-file <rendered body>` |
| `gitlab` | `glab mr create --draft --target-branch <default> --source-branch <branch> --title "<title>" --description-file <rendered body>` (older `glab`: `--description "$(cat file)"`) |
| `none` | print the compare URL only |

Title: Conventional Commit style from the spec's problem line, under 70 characters, ticket
key first when `git.ticket_key` is set (`PROJ-12: add invoice export`).

The rendered body is written to `<spec_root>/<feature>/pr-body.md`, used, then deleted; it
is never committed. The PR URL the CLI prints goes to `state.yml` (`pr`).

## When the CLI is missing

Check with `command -v gh` (or `glab`). Missing: push as under `push`, then print the exact
command from the table with the rendered body's path, and the compare URL
(`<remote_url without .git>/compare/<default>...<branch>?expand=1` on GitHub,
`<remote_url>/-/merge_requests/new?merge_request[source_branch]=<branch>` on GitLab).
`pr` stays empty; the handoff says the PR is the user's to open.

## Branch names, recap

From [`../../_shared/no-attribution.md`](../../_shared/no-attribution.md): `<KEY>-<n>-<slug>`
when the project uses ticket keys, else `feat/<slug>`. Set by `start`; `deliver` never
renames.
