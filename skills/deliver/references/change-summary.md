# The change summary

What the branch built, in the words a reviewer needs. Drafted by `deliver` at every run,
shown with the G5 view, and used twice: as the body of the squash commit and as the
`What changed` section of the PR body. One text, so `git log` and the PR tell the same
story.

## Sources

The summary describes things, not commits. It is drafted from the artefacts that planned the
work and confirmed against the diff:

| Tier | Draft from |
|---|---|
| full | `design.md` `Changes by area` and `Changed contracts`; the `Review fixes` phases of `tasks.md` |
| quick | the task goals and files in `tasks.md` |
| trivial | `brief.md` "Your words" |
| fix (`fix.symptom` set) | the symptom, the cause the reproduction showed, the change: one `Fixed:` bullet |

Then `git diff --stat <default>...HEAD`: a bullet names only paths that are in the diff; a
path in the diff that no bullet covers joins the bullet of the thing it belongs to, or gets
its own; a planned change the diff does not contain is dropped. The diff is the truth, the
design is the vocabulary.

## Form

Plain label lines, each followed by its bullets, in this order and only when non-empty:

```
Added:
- Coin detail page at `/coins/:id` (lazy `CoinDetailView`: `CoinHeader`, `CoinFigures`,
  `PriceChart`, `RangeSelector`) — `src/features/coin-detail/`
- Coin API: `getCoin`, `getCoinMarketChart`, `coinsKeys`, MSW handlers and fixtures — `src/shared/api/`
Changed:
- Coin names in `MarketsTable` link to the detail page
Dependencies:
- `dompurify`, to sanitise the API's description HTML
Tests:
- Unit and component specs beside each file; `e2e/coin-detail.spec.ts` on desktop and mobile
```

- Labels are `Added:`, `Changed:`, `Fixed:`, `Removed:`, `Dependencies:`, `Tests:`. Plain
  words, not headings or bold: the same text goes into a git commit body.
- A bullet names the thing and its kind (route, view, component, composable, service,
  endpoint, migration, formatter, hook, job), gives the path in backticks, and says in one
  clause what it does or how it changed. At most two lines: the behaviour belongs to the
  PR's Summary and the spec, the bullet says what the thing is. Build, CI and config
  changes are `Changed:`.
- `Tests:` names the test levels and where they live, plus test-config changes; it never
  lists test cases.
- At most twelve bullets. Past that, one bullet per area with its folder path.

## Folding

- A review-fix or feedback-fix task folds into the bullet of the thing it changed; it is not
  a bullet of its own. A fix to code that existed before the branch is a `Fixed:` bullet.
- Spec-folder commits, `WIP:` noise, renames done on the way, flake tweaks: not bullets. A
  flake fix a reviewer should know about goes to the PR's `Notes for reviewers`.

## Never

Commit messages, task ids, finding ids, AC ids, review rounds, gate names, tool or command
names. The summary reads as if a colleague wrote it after the fact.

## Handling

Written to `<spec_root>/<feature>/change-summary.md`, passed to `squash-wip --body-file`
and rendered into the PR body, then deleted with `pr-body.md`; never committed. It is
re-drafted from the sources on every re-deliver, so the fixes of a feedback cycle are
absorbed into the bullets they belong to.
