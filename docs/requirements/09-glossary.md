# 09 — Glossary

Shared vocabulary for this folder and `docs/phase1/`. Add a term here
the first time its meaning could be ambiguous between two readers,
rather than letting each document define it slightly differently.

| Term | Meaning in this project |
|---|---|
| **OWN-OS** | This project's operating system — the end product, distinct from "an LFS build" (see `01-prd-own-os.md` §1, D13 in `docs/phase1`). |
| **LFS** | Linux From Scratch — the upstream book/methodology (version 12.4 pinned, `docs/phase1/06-assumptions...` C1) used to build Phase 1. Not the product name. |
| **BLFS** | "Beyond LFS" — the companion upstream project/book covering everything past a bootable base system (desktop environments, servers, etc.). Everything in `docs/phase1` §10.3's out-of-scope list is BLFS territory. |
| **Phase 1** | The milestone defined entirely in `docs/phase1/`: a minimal, bootable, self-hosting LFS-based base system. Ends at M10 in `docs/phase1/10-roadmap-and-open-decisions.md`. |
| **Phase 2+** | Everything after Phase 1, entirely gated on the persona question (Q15/D15) and not yet planned in detail anywhere. |
| **Host (build host)** | The existing Linux system used to bootstrap Phase 1 — see `docs/phase1/02-host-environment-and-partitioning.md`. Discarded from the dependency chain once Phase 1's chroot stage begins; not part of the shipped OWN-OS. |
| **`$LFS`** | The environment variable (and, by extension, the mount point / target partition it points to) used throughout `docs/phase1/` to mean "where OWN-OS is being built." See `docs/phase1/02-host-environment-and-partitioning.md` §2.5. |
| **SBU (Standard Build Unit)** | A hardware-independent build-time unit defined as however long Binutils pass 1 takes to build on one core, on the actual build machine. See `docs/phase1/09-testing-validation-and-risk.md` §9.2. |
| **FR-xx / NFR-xx** | Stable requirement IDs defined in `03-functional-requirements.md` / `04-non-functional-requirements.md`. Reference these, not a paraphrase, from code/commits/PRs. |
| **D1–D18** | The 18 technical/build-engineering decisions tracked in `docs/phase1/10-roadmap-and-open-decisions.md`. |
| **Q1–Q15** | The product-level open questions tracked in `08-open-questions-and-decision-log.md`. Q15 and D15 are the same underlying question, cross-listed deliberately. |
| **Persona (P1–P5)** | The candidate "who is this for" answers defined in `02-target-users-and-use-cases.md` §1 — none are confirmed yet except P1. |
| **[ASSUMED — confirm]** | An inline marker in this folder meaning: this statement was inferred, not told to us directly, and needs explicit confirmation before anything depends on it being true. Tracked centrally in Document 8. |
| **[CANDIDATE]** | An inline marker meaning: this is one possible direction under consideration, not a decision. |
| **[GENERIC] / [DEPENDS ON D15]** | Tags used in `03-functional-requirements.md` to mark whether a requirement applies regardless of persona, or only if a specific persona is confirmed. |
| **"Done" (for Phase 1)** | Specifically: S1–S12 in `07-success-criteria-and-metrics.md` are all true. Not "mostly works" — a binary bar. |
| **Checkpoint** | A saved, restorable snapshot of build state at a defined point (end of Ch. 7, end of Ch. 8), stored outside the ephemeral build environment. See `docs/phase1/09-testing-validation-and-risk.md` §9.3. |
