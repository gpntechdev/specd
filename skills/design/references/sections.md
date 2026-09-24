# Section selection

The main thread reads `spec.md` and picks sections before the architect runs. Approach and
Decisions are always written. Each optional section is selected when a signal appears in the
spec's scope or acceptance criteria; name the triggering AC or sentence when showing the
selection so the user can veto it.

| Section | Select when the spec mentions | Must contain, to be useful to `tasks` |
|---|---|---|
| Data model | an entity, field, table, schema, migration, persisted state, cache shape, file format that changes | the entities and fields with types and constraints; migration steps in order; what happens to existing data |
| API and contracts | an endpoint, route, event, message, webhook, CLI flag, public function or module interface that is new or changes | each contract with input, output and error cases; changed contracts as before → after |
| Sequences | two or more actors or services, an async hop (queue, job, webhook, retry), a timeout or ordering concern | one Mermaid `sequenceDiagram` per flow, happy path plus the error branch that matters |
| UX flows | a screen, page, form, dialog, wizard, multi-step interaction or a state machine the user sees | the screens or states, transitions, and which AC each serves |

Tier rule for this milestone: full and quick both run `design`; the tier does not change the
selection. Trivial runs it too (light flows are M3). A feature with no signal at all still
gets Approach and Decisions; if the architect finds no real decision, `## Decisions` says
"none: the change follows <pattern at path:line>" and G2 is quick.

The signals overlap with triage (`schema`, `api` in `state.yml`); a triage signal that fired
selects its section unless the spec's scope explicitly put it out of scope.
