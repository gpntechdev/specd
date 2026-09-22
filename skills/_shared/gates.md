# Gate protocol

**Reference-only.** Not a skill. Read by every gated step (`specify` G1, `design` G2, `tasks` G3,
`implement` G4, `deliver` and `scaffold` G5); written by nobody. Approval lives in the feature's
`state.yml` (schema: [`state-yml.md`](./state-yml.md)), never only in the conversation.

## Passing a gate

1. Show the artifact's path and a summary of at most ten lines: what it contains, the
   assumptions it makes, the open questions it leaves. Never paste the file.
2. Ask one question with three options: **approve**, **edit** (the user says what to change),
   **reject** (stop here). Use the assistant's question tool when available, otherwise ask in
   prose and wait.
3. On **approve**: write the current UTC time in ISO-8601 (`2026-09-22T14:03:00Z`) to
   `gates.G<n>` in `state.yml`, update `updated`, then continue.
4. On **edit**: apply the change to the artifact on disk, then go back to step 1.
5. On **reject**: leave `gates.G<n>` empty, set nothing else, and end with the handoff block
   (`Next` names this same command).

## Entering a gated step

- Read `state.yml` first. If the gate the step depends on is empty, refuse with one line:
  `G<n> not approved; run /specd:<previous command> first.` Then end with the handoff block.
- A gate approved once stays approved. Re-running the earlier step and changing its artifact
  clears the gate (the earlier step does that when it writes).

## Rules

- Chat approval alone is not approval. If the user says "looks good" in prose, still record it
  in `state.yml` before continuing.
- Risky tasks (marked `risk: high` in `tasks.md`) stop at G4 regardless of
  `flow.gate_granularity`.
- Never approve a gate on the user's behalf, not even for a trivial artifact.
