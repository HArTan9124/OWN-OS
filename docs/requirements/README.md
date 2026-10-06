# Requirements — OWN-OS

This folder holds every Product Requirements Document (PRD) and
requirement specification for the OWN-OS project. The rule going
forward: **no implementation work starts on anything until the relevant
requirement here is written down and its open questions are resolved.**

This folder answers *what we are building and why, for whom, and how we
will know it worked*. `docs/phase1/` (already written) answers *how we
technically build the first milestone (an LFS-based bootable base
system)*. Read both — this folder's decisions feed directly into the
open-decision table in `docs/phase1/10-roadmap-and-open-decisions.md`,
and that table's answers (especially D15, OWN-OS's purpose) feed back
into the PRD here. They are two halves of one plan, not two separate
projects.

## Contents

| # | Document | Answers |
|---|----------|---------|
| 1 | [01-prd-own-os.md](./01-prd-own-os.md) | The master PRD: vision, problem statement, goals/non-goals, high-level requirements |
| 2 | [02-target-users-and-use-cases.md](./02-target-users-and-use-cases.md) | Who this is for, and the concrete scenarios it must support |
| 3 | [03-functional-requirements.md](./03-functional-requirements.md) | What the system must *do*, broken down by area, each with an ID |
| 4 | [04-non-functional-requirements.md](./04-non-functional-requirements.md) | Performance, security, reliability, maintainability, portability constraints |
| 5 | [05-scope-and-out-of-scope.md](./05-scope-and-out-of-scope.md) | What is explicitly in and out, per phase |
| 6 | [06-assumptions-constraints-dependencies.md](./06-assumptions-constraints-dependencies.md) | What we're taking for granted, what limits us, what we depend on |
| 7 | [07-success-criteria-and-metrics.md](./07-success-criteria-and-metrics.md) | How we know each phase is actually done |
| 8 | [08-open-questions-and-decision-log.md](./08-open-questions-and-decision-log.md) | Every unanswered product question, who answers it, and the running decision record once answered |
| 9 | [09-glossary.md](./09-glossary.md) | Shared vocabulary, so "done," "stable," "supported," etc. mean one thing project-wide |

## How to use this folder

- **Every requirement gets an ID** (e.g. `FR-01`, `NFR-03`) so it can be
  referenced from code, commits, PR descriptions, and `docs/phase1/`
  without ambiguity.
- **Nothing here is final.** Documents 1–7 are a first draft built from
  the conversation and research so far, written so there is something
  concrete to react to rather than a blank page. Every place something
  was inferred or assumed rather than explicitly confirmed is marked
  **[ASSUMED — confirm]** inline. Document 8 is the single place every
  open question collects, including every `[ASSUMED]` marker from the
  other documents — resolve items there, not by silently editing the
  assumption in place.
- **Changing a requirement after work has started on it** is a real
  change, not a typo fix — note it in Document 8's decision record with
  a date, not just overwrite the original text.
