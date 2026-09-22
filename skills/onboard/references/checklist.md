# Knowledge checklist

The questions each `docs/` section must be able to answer. `onboard` fills them from what exists
(code, existing docs, pasted sources) and interviews only for the gaps. A section is complete
when every question has an answer or an explicit "not applicable" in the file.

## project (`<docs_root>/project.md`)

- P1 What is this project, in two sentences, for someone who has never seen it?
- P2 Who uses it and what do they get from it?
- P3 What is the stack: language, runtime and framework, with versions?
- P4 How is it run locally, tested, linted, formatted, type-checked and built? (one command each)
- P5 What is the top-level layout and what lives where?
- P6 What does it integrate with: services, APIs, queues, third parties?
- P7 Where do requirements and decisions come from: tracker, wiki, chat, a person?
- P8 (greenfield only) What must, should and will not be in the first version?

## architecture (`<docs_root>/architecture/overview.md` and `<area>.md`)

- A1 What are the main areas or subsystems and what is each responsible for?
- A2 Where does a request or job enter and which areas does it pass through? (key flows)
- A3 What are the boundaries: what may call what, what is forbidden?
- A4 What are the tooling choices: package manager, lint, format, typecheck, test runner, build?
- A5 What is the testing strategy: levels, what is mocked, what runs in CI?
- A6 Which decisions shaped the structure and why? (links to `decisions/`)
- A7 Which existing architecture docs exist and where? (linked, not copied)

## conventions (`<docs_root>/conventions.md`)

- C1 Code style: formatter, linter and the rules that are enforced, not just wished for.
- C2 Patterns in use: error handling, logging, configuration, dependency wiring, module layout.
- C3 Naming: files, modules, functions, tests, branches, commits.
- C4 Testing rules: what every change must cover, where tests live, naming, fixtures.
- C5 Git and PR rules: branch model, review expectations, what blocks a merge.
- C6 What must never be done in this codebase?

## data-models (`<docs_root>/data-models/README.md` and `<domain>.md`)

- D1 Does the project persist data at all? If not, say so with the date checked.
- D2 What are the domains and their main entities?
- D3 How do entities relate; what are the invariants?
- D4 Where is each model defined in code and how are migrations done?

## decisions (`<docs_root>/decisions/README.md` and `NNNN-<title>.md`)

- R1 Which decisions are already recorded (ADRs, wiki pages, comments) and where?
- R2 Which choices visible in the code have no recorded reason? (candidates to record)
- R3 For each recorded decision: context, the decision, consequences.
