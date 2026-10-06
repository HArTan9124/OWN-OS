# 06 — Assumptions, Constraints & Dependencies

## Assumptions (believed true, not yet confirmed)

| ID | Assumption | Where it's used | Risk if wrong |
|---|---|---|---|
| A1 | Single project owner, no team, no external users yet | `01-prd-own-os.md` §8 | Solo-oriented decisions (workflow, maintenance ownership) would need rework for a team |
| A2 | Problem is educational/ownership-driven, not a market or technical gap | `01-prd-own-os.md` §2 | Changes almost every prioritization downstream |
| A3 | Trusted local/dev threat model (not Internet-facing) | `04-non-functional-requirements.md` NFR-07 | Security posture would need a full rework if wrong |
| A4 | x86_64, single architecture, no multilib | `docs/phase1` D1, NFR-15 | Toolchain bootstrap (`docs/phase1/04-toolchain-bootstrap.md`) assumes this throughout |
| A5 | Build happens in an ephemeral cloud session/container, not persistent local hardware | `docs/phase1/09-testing-validation-and-risk.md` §9.3 | Backup/checkpoint strategy exists specifically because of this — if wrong (persistent host), checkpoints are still good practice but less urgent |
| A6 | No fixed boot-time or disk-footprint budget needed yet | NFR-02, NFR-03 | Only matters if an embedded/appliance persona (P4) is confirmed |

## Constraints

- **C1 — LFS version pin.** Built against LFS 12.4 (stable as of
  2025-09-01). A future LFS point release is a deliberate decision, not
  an automatic upgrade — re-validate `docs/phase1/03-sources-and-packages.md`
  against the new release's errata before adopting it.
- **C2 — No distro inheritance.** Nothing may be installed as a
  pre-built binary package from a third-party distro repository; this
  is the one constraint that most directly defines what "OWN-OS" means
  as opposed to "a regular Linux install" — see `01-prd-own-os.md` §2–3.
- **C3 — Host toolchain floor/ceiling.** The build host's own toolchain
  versions must satisfy the floor and must not exceed the untested
  ceiling documented in `docs/phase1/02-host-environment-and-partitioning.md`
  §2.1 — this constrains *where* Phase 1 can even be attempted.
- **C4 — Single continuous or explicitly-resumable build.** Given A5,
  the build must either fit in one session or have a working
  resume/checkpoint procedure in active use — not an assumption that a
  multi-day uninterrupted session is available (`docs/phase1` D2).

## Dependencies

- **External: linuxfromscratch.org.** The entire Phase 1 technical plan,
  package list, and patch set depend on the LFS project's own published
  book, source mirrors, and security-advisory page continuing to be
  reachable and accurate. No fallback plan currently exists if a
  specific upstream source URL goes dead outside of the book's own
  mirror guidance (`docs/phase1/03-sources-and-packages.md` §3.4).
- **External: upstream package maintainers.** Every one of the ~90
  packages is a dependency in the literal sense — a security issue or
  build-breaking change in any one of them is not under OWN-OS's
  control and must be tracked (`docs/phase1` D16, NFR-04).
- **Internal: `docs/phase1/` itself.** This requirements folder assumes
  the Phase 1 technical plan is accurate and current. If `docs/phase1/`
  changes (e.g. a Document-10 decision flips), check whether any
  requirement here that cites it needs updating too.
- **Internal: this session's research.** Both `docs/phase1/` and this
  folder were produced from reading the LFS book directly plus this
  conversation — no independent second source was cross-checked. Treat
  any single factual claim that materially affects a go/no-go decision
  (version floors/ceilings, mandatory kernel config options) as worth a
  quick re-check against the live book before relying on it at execution
  time, since the book itself updates between point releases.

## Review trigger

Re-read this document whenever any assumption A1–A6 is confirmed,
denied, or becomes newly uncertain — each one gates specific
requirements elsewhere in this folder, listed in the table above.
