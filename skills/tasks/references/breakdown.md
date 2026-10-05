# Breaking a feature into tasks

## Size and order

- A task is at most a day of work and touches one concern: one module, one migration, one
  endpoint, one screen. If listing its `files` takes more than five lines, split it.
- Order by dependency, not by design section: what must exist first comes first. Phases
  group tasks that can be reviewed together at a G4 stop: `Foundations` (schema, contracts,
  helpers), `Behaviour` (the feature itself), `Edges` (error paths, limits, permissions),
  `Verify`. Use the names the feature needs; keep `Verify` last.
- Between two and twelve tasks is the norm. One task means the tier should have been
  trivial; more than twelve means the spec is two features.

## Layout: inline or split

- Inline: every task is a block in `tasks.md`. Right for a handful of tasks without tests
  blocks; the file stays readable in one screen.
- Split: `tasks.md` holds the header, each phase with one index line per task
  (`- T<n> · <goal> · risk <low|high> · covers <ACs> · tasks/T<n>.md`) and the matrix; each
  task body is `tasks/T<n>.md` from `templates/task.md`, at most 60 lines. Use it when the
  inline file would pass 100 lines; with tests blocks that is about six tasks.
- Decide before writing, never convert mid-way; the `Layout:` field in the header says
  which. Readers test for the `tasks/` folder: present means split.
- Review-fix tasks follow the layout in force: an index line plus a file in split, a block
  inline.

## done-when forms

- A command: `<test command> -k <name> passes`, `<lint command> exits 0`, `curl … returns 201`.
- An observable: "`GET /items/1` returns the record created by T2", "the migration applies
  and rolls back on an empty database", "the screen renders the empty state from AC4".
- Never "code is written", "works as expected" or "reviewed".

## risk

Mark `risk: high` when the task: changes or deletes persisted data or a migration; touches
auth, permissions, payments or secrets; changes a public contract other code depends on;
removes code; or runs anything against an external system. High-risk tasks always stop for
G4 (`gates.md`), whatever the granularity.

## commit

One Conventional Commit message per task, fixed here so `implement` never invents one:
`<type>(<scope>): <summary>`, imperative, no period, type from `feat`, `fix`, `test`,
`refactor`, `chore`, `docs`; scope per `conventions.md` when it names scopes, else the
module. `commit: none` only for the Verify task.

## covers and the matrix

- `covers` lists the AC ids the task makes true. A prerequisite task with no AC of its own
  lists the AC of the task that needs it, with `(via T<n>)`.
- The matrix has exactly one row per AC in `spec.md`; every row names at least one task.
  The Test column names the test case (TDD on), the check that proves it (`lint`, `typecheck`,
  a command) or `manual` for ACs written as `manual:` in the spec.
- `verify` writes Evidence; leave it empty here.

## tests block (TDD on)

- One case per behaviour in the covered ACs, named the way the project names tests
  (`conventions.md`, else the test framework's convention): what it asserts, not how.
- Error paths from the ACs get their own case. The Verify task has no tests block.
- `implement` writes these cases first and watches them fail before writing code.
