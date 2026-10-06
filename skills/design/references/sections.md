# Section selection

The main thread reads `spec.md` and picks sections before the architect runs. Approach and
Decisions are always written. Each optional section is selected when a signal appears in the
spec's scope or acceptance criteria; name the triggering AC or sentence when showing the
selection so the user can veto it.

| Section | Select when the spec mentions | Must contain, to be useful to `tasks` |
|---|---|---|
| Data model | an entity, field, table, schema, migration, persisted state, cache shape, file format that changes | `### Entities` table (entity, change, fields and constraints, pointer); `### Migration` steps in order and what happens to existing data |
| API and contracts | an endpoint, route, event, message, webhook, CLI flag, public function or module interface that is new or changes | the kinds present, each under its fixed sub-heading: `### External calls` (table), `### Types` (one code fence), `### Module interfaces` (a fenced signature plus one line of behaviour each), `### Components` (table), `### Changed contracts` (before → after table) |
| Sequences | two or more actors or services, an async hop (queue, job, webhook, retry), a timeout or ordering concern | one Mermaid `sequenceDiagram` per flow, happy path plus the error branch that matters |
| UX flows | a screen, page, form, dialog, wizard, multi-step interaction or a state machine the user sees | the screens or states, transitions, and which AC each serves |

Tier rule: only full runs `design`. `specify` applies this same table to a quick spec and
proposes escalation when a section would be selected; a trivial feature has no spec. A full
feature with no signal at all still gets Approach and Decisions; if the architect finds no
real decision, `## Decisions` says "none: the change follows <pattern at path:line>" and G2
is quick.

Approach always has three sub-headings: `### Summary` (three sentences, no pointers),
`### Builds on` (existing code reused, `path:line` each), `### Changes by area` (a table
Area | Files | Change that `tasks` turns into the task list). Prose that explains how
something works belongs in a decision's Context or in a sequence, not in the summary.

The signals overlap with triage (`schema`, `api` in `state.yml`); a triage signal that fired
selects its section unless the spec's scope explicitly put it out of scope.
