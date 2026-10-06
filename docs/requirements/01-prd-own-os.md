# 01 — Product Requirements Document: OWN-OS

**Status:** Draft v0.1 — built from project conversation + the LFS-based
technical research in `docs/phase1/`. Every **[ASSUMED — confirm]** tag
marks a place this PRD guessed rather than being told, and is tracked in
[08-open-questions-and-decision-log.md](./08-open-questions-and-decision-log.md).

## 1. Summary

OWN-OS is a custom, source-built Linux operating system, constructed
from the ground up rather than derived from or repackaging an existing
distribution. Phase 1 (fully planned in `docs/phase1/`) produces a
minimal, bootable, self-hosting base system by following the Linux From
Scratch (LFS 12.4) methodology. Everything after that — what the system
is actually *for* — is defined by this requirements folder.

## 2. Problem statement

**[ASSUMED — confirm]** The problem this project solves is primarily
**educational and ownership-driven**, not a gap in the existing Linux
distro market: understanding exactly how a Linux system is assembled,
having a system with no unexplained binary or unexplained configuration
choice, and being able to shape every layer intentionally rather than
inheriting a distro maintainer's defaults. This mirrors the LFS
project's own stated rationale (`docs/phase1/01-overview-and-methodology.md`
§1.1), which we're adopting as this project's problem statement until
told otherwise.

If the actual motivation is different (e.g. a specific technical
requirement no existing distro meets, a commercial product, an
embedded/appliance target with licensing or size constraints a stock
distro can't satisfy), **this changes almost every downstream
requirement** and should be corrected here first.

## 3. Vision

A Linux-based operating system that:
1. Boots and runs entirely on software we built and understand, with no
   unaudited component.
2. Starts from the minimal LFS base (Phase 1) and grows deliberately,
   one explicitly-justified package/capability at a time — never by
   inheriting a distro's package set wholesale.
3. Has a name, identity, and versioning scheme of its own (not "a
   Linux From Scratch build") — see `docs/phase1/08-finalization-and-post-lfs.md`
   §8.1, D13.

## 4. Goals

- **G1.** Produce a working, bootable base system via the Phase 1 plan
  (tracked entirely in `docs/phase1/`, not duplicated here).
- **G2.** Define, in this folder, what OWN-OS needs to *do* beyond
  booting — its actual use case(s) — before any Phase 2 (BLFS-era) work
  starts.
- **G3.** Keep every design decision traceable to a written requirement
  or an explicit, dated decision record (Document 8), never an implicit
  assumption baked silently into code or config.
- **G4.** Keep the system auditable and minimal by default — a
  capability is added because a requirement in Document 3 calls for it,
  not because it's commonly bundled elsewhere.

## 5. Non-goals (for now)

- **[ASSUMED — confirm]** Commercial distribution, certification (e.g.
  LSB certification proper — see `docs/phase1/01-overview-and-methodology.md`
  §1.4), or supporting hardware/architectures beyond the single target
  confirmed in `docs/phase1` D1 are **not** goals unless stated
  otherwise.
- Replacing or competing with an existing mainstream distro on
  features/package breadth is explicitly not the goal — see §2.
- Anything catalogued as out-of-scope in `docs/phase1/10-roadmap-and-open-decisions.md`
  §10.3 remains out of scope here too unless this PRD overrides it
  explicitly (it currently does not).

## 6. Primary use case(s) — needs confirmation

See [02-target-users-and-use-cases.md](./02-target-users-and-use-cases.md)
for the full breakdown. This is the single highest-leverage open
question for the whole project (also flagged as D15 in
`docs/phase1/10-roadmap-and-open-decisions.md`): is OWN-OS meant to be a
**personal workstation**, a **server/appliance**, an **embedded/minimal
target**, or primarily a **learning vehicle** with no fixed end-use
beyond the build process itself? This PRD does not pre-decide it, but
every other requirement in this folder is written to degrade gracefully
regardless of which answer comes back — concrete, use-case-specific
requirements only appear in Document 3 where the use case is already
clear from context (e.g. "must boot," "must have a shell") and are
marked **[GENERIC]**; anything use-case-dependent is marked
**[DEPENDS ON D15]**.

## 7. High-level requirements summary

Full detail lives in Documents 3 (functional) and 4 (non-functional).
At the PRD level, OWN-OS must, regardless of eventual use case:

- Boot reliably on the confirmed target architecture (`docs/phase1` D1).
- Provide a working shell, core userland, and networking stack (this is
  just Phase 1 / LFS's own base package set — already fully specified
  in `docs/phase1/03-sources-and-packages.md`).
- Be maintainable by the project owner alone **[ASSUMED — confirm team
  size]** without requiring upstream distro tooling.
- Carry its own identity (name/version/branding) distinct from "LFS."
- Have a documented, repeatable build process (this requirements folder
  + `docs/phase1/` together constitute that documentation — the goal is
  that a rebuild from zero only ever needs these docs, not tribal
  knowledge).

## 8. Stakeholders

**[ASSUMED — confirm]** Single project owner, decision-maker, and
primary (initially only) user/operator, identified in this session as
`tandonharshit757@gmail.com`. No other stakeholders (team members,
external users, customers) are currently known. If this is actually a
team project or has an intended external user base, every requirement
in this folder that assumes a solo audience needs revisiting.

## 9. Relationship to `docs/phase1/`

| | This folder (`docs/requirements/`) | `docs/phase1/` |
|---|---|---|
| Answers | What, for whom, why, how we measure success | How, in exactly what technical order |
| Level | Product | Build engineering |
| Changes when | The project's purpose/audience/priorities change | The LFS version, package set, or build mechanics change |
| Owns D15 (purpose)? | Yes — this is where it gets answered | No — only flags it as blocking |

Phase 1 execution should not begin in earnest until this PRD's open
questions (Document 8) are answered, specifically D15/the use-case
question in §6 — not because Phase 1's *technical* steps depend on the
answer (they mostly don't; LFS's base system is use-case-agnostic by
design), but because several concrete Phase 1 decisions already flagged
in `docs/phase1/10-roadmap-and-open-decisions.md` (networking mode,
locale, init system) are cheaper to lock in once correctly, in light of
the real target use case, than to revisit after the fact.
