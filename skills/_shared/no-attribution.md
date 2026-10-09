# No AI attribution

**Reference-only.** Not a skill. Read by `init` (writes the setting) and `deliver` (writes
commits and PRs).

## The setting

`init` writes this into the workspace's `.claude/settings.json` (merging if the file exists).
A plugin cannot ship it: plugin-level `settings.json` supports only `agent` and
`subagentStatusLine`, so it has to land in each workspace.

```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

Empty strings turn off the Co-Authored-By trailer and the "Generated with" PR line.
The old `includeCoAuthoredBy` key is deprecated and not written.

## Wording rules for deliver

- Commit messages follow Conventional Commits: `type(scope): summary`, imperative, no period.
- No mention of an AI assistant, model or tool in commit messages, PR titles or PR bodies.
  No trailers of any kind unless the project's conventions require one (such as `Refs:` for a
  ticket).
- Branch names carry the ticket key when `git.ticket_key` is set: `<KEY>-<n>-<slug>`;
  otherwise `<type>/<slug>`: `feat/<slug>` from `start`, `fix/<slug>` from `fix`.
- The PR body describes what changed and how it was verified, in the same voice a colleague
  would use.
