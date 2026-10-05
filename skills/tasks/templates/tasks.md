<!-- specd:draft -->
<!-- Written by /specd:tasks to <spec_root>/<feature>/tasks.md; read by implement, review,
     verify and deliver. Ids T1.. never change after G3; review appends a
     "Review fixes (round n)" phase with new ids. Two layouts, never mixed: INLINE, each task
     is the block below; SPLIT (used when the inline file would pass 100 lines), each task is
     one index line `- T<n> · <goal> · risk <low|high> · covers <ACs> · tasks/T<n>.md` and its
     body lives in tasks/T<n>.md. Fields per task: goal, files, done-when (a command or an
     observable), depends-on, risk (low|high), commit (Conventional Commit message implement
     uses verbatim), covers (AC ids), tests (TDD only: case names). The matrix has one row per
     AC; verify fills Evidence. state.yml mirrors the ids. Remove the first line on G3. -->
# Tasks: {{feature}}

Spec: `spec.md` · Design: `design.md` · TDD: {{on|off}} · Gate granularity: {{task|phase|end}} · Layout: {{inline|split}}

## Phase 1: {{name}}

- T1
  - goal:
  - files:
  - done-when:
  - depends-on: none
  - risk: low
  - commit: {{type}}({{scope}}): {{summary}}
  - covers: AC1
  - tests:
    - {{test case name: what it asserts}}

## Phase {{n}}: Verify

- T{{n}}
  - goal: every project check passes on the finished feature
  - files: none
  - done-when: `run-checks` reports ok
  - depends-on: {{all previous}}
  - risk: low
  - commit: none
  - covers: all

## Coverage matrix

| AC | Task | Test | Evidence |
|---|---|---|---|
| AC1 | T1 | {{test name or check or manual}} | |
