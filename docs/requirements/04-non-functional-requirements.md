# 04 — Non-Functional Requirements

Each requirement has a stable ID (`NFR-NN`). These describe *qualities*
of the system (how well it does things), as opposed to
`03-functional-requirements.md` (what it does).

## Performance

- **NFR-01** Build time must be measured and budgeted, not open-ended —
  using the SBU methodology already defined in
  `docs/phase1/09-testing-validation-and-risk.md` §9.2. A total
  wall-clock estimate must exist before a build session is committed to.
- **NFR-02 [ASSUMED — confirm]** No hard boot-time target is currently
  set. Upstream LFS's own benchmark (~8–12 seconds kernel-to-login on
  SysVinit on medium hardware, `docs/phase1/06-system-configuration.md`
  §6.1) is the implicit baseline until a real target is confirmed. If
  OWN-OS is ever aimed at a boot-time-sensitive use case (P4,
  embedded/appliance in Document 2), this needs an explicit target
  (e.g. "boots to login in under N seconds on reference hardware").
- **NFR-03 [ASSUMED — confirm]** No disk-footprint target is currently
  set beyond the Phase 1 partition sizing guidance (10 GB minimum, 20–30
  GB recommended, `docs/phase1/02-host-environment-and-partitioning.md`
  §2.3). If a minimal/embedded target is ever confirmed, a concrete
  image-size budget needs to be added here.

## Security

- **NFR-04** Every package's known security advisories must be checked
  before pinning its version (`docs/phase1/03-sources-and-packages.md`
  §3.4) and monitored on an ongoing basis after boot
  (`docs/phase1/08-finalization-and-post-lfs.md` §8.6).
- **NFR-05** No static linking of libraries except where structurally
  required (Glibc, GCC — `docs/phase1/05-base-system-build.md` §5.1),
  specifically so a library security fix only requires rebuilding/
  relinking the library itself, not every consumer.
- **NFR-06** A root password must be set before first boot and the
  system must never ship or boot with a known/default credential
  (`docs/phase1/08-finalization-and-post-lfs.md` §8.2).
- **NFR-07 [ASSUMED — confirm]** No formal threat model exists yet.
  Given the current confirmed persona is P1 (builder, local/dev use —
  `02-target-users-and-use-cases.md` §4), the assumed exposure is
  "trusted local/dev environment, not Internet-facing," which is why
  static IP + no hardened network-facing services is an acceptable
  Phase 1 default (`docs/phase1` D9). **This assumption must be revisited
  the moment any server/appliance persona (P3) is confirmed** — an
  Internet-facing OWN-OS needs its own security requirements document,
  not an inherited Phase-1-era assumption.

## Reliability & recoverability

- **NFR-08** The build must have documented checkpoints that can be
  restored without re-running already-completed stages
  (`docs/phase1/09-testing-validation-and-risk.md` §9.3), stored outside
  the (ephemeral) build environment itself.
- **NFR-09** Every known, named failure mode in the risk register
  (`docs/phase1/09-testing-validation-and-risk.md` §9.4) must have a
  mitigation already in place before the stage it threatens begins —
  not discovered live during execution.
- **NFR-10 [ASSUMED — confirm]** No uptime/availability target exists
  (not meaningful yet for a system that doesn't run a persistent service
  — this becomes relevant only if P3, server/appliance, is confirmed).

## Maintainability

- **NFR-11** Every requirement, decision, and assumption must be
  written down in this folder or `docs/phase1/`, with an ID where
  applicable — no undocumented tribal knowledge. This is the operating
  principle behind the entire `docs/` structure, not just a nice-to-have.
- **NFR-12** Every file installed by a package must be attributable to
  that package (even without a formal package manager, Phase 1 — see
  `docs/phase1` D6) well enough that "what installed this file and why"
  is always answerable from the package list in
  `docs/phase1/03-sources-and-packages.md` plus the build logs.
- **NFR-13** Kernel source and configuration must be retained (not
  deleted after build) specifically to support future reconfiguration
  without re-deriving settings from scratch (`docs/phase1/07-bootable-system.md`
  §7.2 step 7).

## Portability / compatibility

- **NFR-14** The system follows POSIX.1-2008, FHS 3.0, and the
  LSB-Core/LSB-Languages package subset (not full LSB certification)
  as its compatibility baseline (`docs/phase1/01-overview-and-methodology.md`
  §1.4) — chosen so third-party source software has a predictable
  environment to build against, even though OWN-OS itself ships no
  binary compatibility guarantee.
- **NFR-15** Single target architecture only for now
  (`docs/phase1` D1) — no multi-arch requirement exists, and none
  should be assumed into any design without a corresponding entry here.

## Auditability

- **NFR-16** Every package in the base system must have a stated reason
  for inclusion (already true for the full Phase 1 set —
  `docs/phase1/01-overview-and-methodology.md` §1.5) — this bar carries
  forward to every future addition past Phase 1 as well (ties to
  `FR-14`).

## Open items from this document

Flagged **[ASSUMED — confirm]** above (`NFR-02`, `NFR-03`, `NFR-07`,
`NFR-10`) are tracked in
[08-open-questions-and-decision-log.md](./08-open-questions-and-decision-log.md).
