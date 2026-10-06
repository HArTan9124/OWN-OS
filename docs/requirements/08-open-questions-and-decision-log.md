# 08 — Open Questions & Decision Log

Two things live in this one document on purpose: the **open-questions
table** (everything still unresolved, pulled from every `[ASSUMED —
confirm]` / `[CANDIDATE]` tag elsewhere in this folder) and the
**decision log** (a dated, append-only record of what got decided,
once it's decided). When a question is answered, move its row's
essence into the log below and mark the table row "Resolved" rather
than deleting it — the history of *why* is as valuable as the answer.

This is the product-level counterpart to
`docs/phase1/10-roadmap-and-open-decisions.md`, which tracks the
technical/build-engineering decisions (D1–D18) separately. **Q15
below is the same question as `docs/phase1` D15** — it's listed here
too because it's the one question both folders depend on.

## Open questions

| ID | Question | Raised in | Current default (if any) | Status |
|---|---|---|---|---|
| Q1 | Is this a solo project or a team? | `06-assumptions...md` A1 | Solo | Open |
| Q2 | Is the motivation educational/ownership, or a specific technical/market gap? | `01-prd-own-os.md` §2 | Educational/ownership | Open |
| Q3 | What threat model applies — trusted local/dev only, or ever Internet-facing? | `04-non-functional-requirements.md` NFR-07 | Trusted local/dev only | Open |
| Q4 | Any boot-time budget? | `04-non-functional-requirements.md` NFR-02 | None set | Open |
| Q5 | Any disk-footprint budget? | `04-non-functional-requirements.md` NFR-03 | None set (10–30 GB partition guidance only) | Open |
| Q6 | Any uptime/availability target? | `04-non-functional-requirements.md` NFR-10 | Not applicable yet | Open |
| **Q15** | **Who/what is OWN-OS actually for — P1 builder / P2 workstation / P3 server / P4 embedded / P5 reference artifact (any combination)?** | `02-target-users-and-use-cases.md` §4; mirrors `docs/phase1` **D15** | P1 + P5 only, for now | **Open — highest priority, blocks Phase 2 entirely** |

Every `docs/phase1` decision D1–D18 is also, implicitly, open from this
folder's point of view until resolved — see that document's own table
rather than duplicating all 18 rows here. Q15/D15 is cross-listed
because it's the one question that changes requirements in *both*
folders simultaneously.

## How to resolve a question

1. Get the actual answer from whoever owns the decision (currently: the
   project owner, per A1).
2. Add a dated entry to the Decision Log below.
3. Update the row's **Status** to `Resolved (see log YYYY-MM-DD)`.
4. Go back to every document that cited the question (the "Raised in"
   column, plus anywhere else a grep turns up) and replace the
   `[ASSUMED — confirm]` / `[CANDIDATE]` marker with the confirmed
   text — don't leave stale assumption language sitting next to a
   resolved answer.
5. If the answer changes a `docs/phase1` decision too, update that
   document's table in the same pass.

## Decision log

**2026-10-06 — OS name confirmed: "Ravya OS".** Resolves `docs/phase1`
**D13**. Will appear in `/etc/os-release` (`NAME="Ravya OS"`,
`ID=ravya`) and the login banner once Stage 7 (identity & final
review) runs.

**2026-10-06 — Remaining low-stakes Stage-0 decisions (D2–D11, D14,
D16, D17) resolved by applying this plan's own recommended defaults**,
reviewed and accepted rather than individually deliberated: continuous
build with periodic exported checkpoints (D2, D17), ~6 GB image on the
confirmed 32 GB SD card (D3), `-j4` (D4), no stripping (D5), no package
manager for Phase 1 (D6), SysVinit (D7), persistent network naming
(D8), static IP with HDMI-console-first validation (D9), `C.UTF-8`
(D10), PARTUUID-based disk identification (D11), HDMI-then-SSH
post-boot workflow (D14), project owner as the recurring
security-monitoring owner (D16). Full detail and rationale in
`docs/phase1/10-roadmap-and-open-decisions.md` §10.1. **D15/Q15
(OWN-OS's purpose) and Q1–Q3 remain open — explicitly, since they gate
Phase 2 decisions, not anything Phase 1 execution needs.**

**2026-10-06 — Target hardware confirmed: Raspberry Pi 4 or 5
(`aarch64`), not x86_64.** Resolves `docs/phase1` **D1**. Build happens
via full cross-compilation in the cloud build session; the Pi is
touched only once, to flash and boot the finished SD-card image
(resolves **D20** — chosen over LFS's own "Cross Edition" approach,
which would have required part of the build to run natively on the
Pi). Consequences resolved in the same pass: **D12** (no GRUB —
Raspberry Pi firmware boot instead), **D19** (kernel source: mainline
`kernel.org`, not the Raspberry Pi Foundation's fork — accepted
trade-off of incomplete camera/GPU-acceleration support), **D18**
(test-suite execution deferred to after first boot on real hardware,
since full cross-compilation means nothing runs natively on the build
host at all). Full technical detail in
`docs/phase1/11-arm-raspberry-pi-adaptation.md`. Still open from this
same decision round: who runs the build day-to-day (solo, per A1,
unchanged), and Q15/D15 (OWN-OS's purpose) — the architecture choice
doesn't answer that, it only fixes what hardware any answer runs on.

---

*(Append new entries above this line, newest first, each dated.)*

---

_The first entries above resolve the architecture/build-methodology
fork. Still needed: resolutions to Q15/D15 and Q1–Q3, since almost
everything else cascades from those three._
