# Git for deliver

## What `git.authority` allows

| Value | start | implement | deliver |
|---|---|---|---|
| `none` | no branch | no commits | G5 view only; prints what it would do |
| `commit` | branch | commits | G5 view; stops before push |
| `push` | branch | commits | push the branch |
| `draft_pr` | branch | commits | push, then a draft PR or MR |

## Squash

`implement` commits every task as `WIP: <its commit line>` until a PR is recorded. Before
the first push, step 4
offers to collapse the branch into one commit with
`"${CLAUDE_PLUGIN_ROOT}/scripts/squash-wip" --from <code_root> --base <default> --title "<title>"`:
a soft reset to the merge base and one commit, so the tree is unchanged and `before` in its
output restores the old history (`git reset --hard <before>`). In embedded mode the
`spec(<feature>): …` commits on the branch collapse into it too; its body lists every
squashed subject. Title: Conventional Commit from the spec's problem line, `type(scope):
summary`, type and scope as the tasks' commit lines use them, under 70 characters, no ticket
key (that goes in the PR title and `Refs:`). The script refuses, and the step keeps the
commits, when the branch already tracks a remote, the tree is dirty, or no `WIP:` commit
exists; `git.authority: none` never asks.

## Push

`git -C <code_root> push -u origin <branch>` where `branch` comes from `state.yml`. A remote
that rejects the push (no permission, protected name) is reported verbatim and the run stops
at G5 recorded; no retry with different flags.

## Re-deliver

After a feedback cycle the branch already tracks the remote: no squash is offered, the
fix commits carry their plain Conventional Commit messages (`implement` drops the `WIP:`
prefix once `pr` is set) and stay in the history, the push is
a plain `git push`, and the PR in `state.yml` `pr` picks the commits up by itself. The G5
view's "since the last push" log is what the reviewers will see as new.

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
