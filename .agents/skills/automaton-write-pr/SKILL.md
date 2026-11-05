---
name: automaton-write-pr
description: Use whenever you open a pull request in Automaton or automaton-apps, from any flow, so every PR says what changed, how it was verified, what could break, and never closes its issue before production.
license: Apache-2.0
---

# Writing a pull request

## Title

What the change does for people, in the imperative: "Let app owners delete apps after losing Runner access (#533)".

## Body

```markdown
Refs #<issue>.

- **<behaviour>:** what changed and why, one line each.
- **Docs:** which Admin or architecture docs were updated, or why none were needed.
- **Tests:** what was added, and what ran (project names, pass counts).

**How verified:** end-to-end on the test server (link the proof) / unit tests only (why) / not yet (after release).
**Risk:** what could break and how to roll back, in one line.

**Not in this PR:** anything from the issue left for later, with the follow-up issue number.
```

For Studio or website changes, add a screenshot (push it to the `pr-screenshots` branch and link it).

"How verified" usually says "not yet" when the PR opens, because test-server checks run after merge. Update it in a
comment when the proof is attached.

Never write `Closes #N` or `Fixes #N`: an issue closes only when its change is live on production. Use
`Part of #N` when the PR delivers only part of the issue.

## Documentation

Update documentation in the same PR when behaviour that administrators or users see changes (admin-docs skill), or
when architecture changes (`AGENTS.md`). Keep it brief: say what is true now, in a few sentences, in the existing
page. Do not add new pages, change logs or plans unless the task asks for them.

## Commits

Small commits are fine; the PR is squash-merged. End commit messages and the PR body with the attribution lines
your tool asks for.
