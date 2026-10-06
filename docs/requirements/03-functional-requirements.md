# 03 — Functional Requirements

Each requirement has a stable ID (`FR-NN`) for reference from code,
commits, and PRs. **[GENERIC]** = applies regardless of how
`02-target-users-and-use-cases.md` resolves. **[DEPENDS ON D15]** = only
applies to certain personas; the requirement says which.

Every `FR` here that is already fully specified at the engineering level
is cross-referenced to its `docs/phase1/` section — this document does
not re-derive those details, only states the requirement and points at
where it's satisfied.

## Boot & kernel

- **FR-01 [GENERIC]** The system must boot, unattended, directly into a
  usable login prompt on its own kernel and bootloader, with no
  dependency on the original build host at boot time.
  *Satisfied by:* `docs/phase1/07-bootable-system.md`, M9 in
  `docs/phase1/10-roadmap-and-open-decisions.md`.
- **FR-02 [GENERIC]** The kernel must support the confirmed target
  architecture and the confirmed boot path (BIOS or UEFI,
  `docs/phase1` D12), including the storage controller actually in use
  (e.g. NVMe — `docs/phase1/07-bootable-system.md` §7.2).
- **FR-03 [GENERIC]** A rescue/recovery path must exist before the
  bootloader is installed to a disk that matters (`docs/phase1/07-bootable-system.md`
  §7.3 safety note) — not a "nice to have," a precondition for FR-01.

## Toolchain & build reproducibility

- **FR-04 [GENERIC]** The entire system must be buildable from source
  using a documented, repeatable procedure with no undocumented manual
  step — this requirements folder + `docs/phase1/` together are that
  procedure.
- **FR-05 [GENERIC]** Every package version in the base system must be
  pinned and recorded (`docs/phase1/03-sources-and-packages.md`), with
  version drift from upstream LFS only happening deliberately, after
  checking advisories (`docs/phase1/03-sources-and-packages.md` §3.4).
- **FR-06 [GENERIC]** The build must be resumable/restartable from a
  known checkpoint without re-doing already-completed stages
  (`docs/phase1/09-testing-validation-and-risk.md` §9.3).

## Userland & shell

- **FR-07 [GENERIC]** A POSIX-compatible interactive shell (Bash) and
  the standard core utilities must be present and be the first things
  usable after boot. *Satisfied by:* Phase 1 base package set.
- **FR-08 [GENERIC]** The system must support at least one non-graphical
  way to edit text and transfer files onto itself post-boot
  (`docs/phase1/08-finalization-and-post-lfs.md` §8.4) — this is what
  makes the system usable for its own further development without
  depending permanently on the build host.

## Networking

- **FR-09 [GENERIC]** The system must be able to bring up at least one
  network interface with a usable IP configuration, DNS resolution, and
  a stable hostname (`docs/phase1/06-system-configuration.md` §6.4).
- **FR-10 [DEPENDS ON D15]** If a persona requiring dynamic/roaming
  networking is confirmed (P2 workstation, mobile use), DHCP support
  must be added — currently deferred per `docs/phase1` D9 (static IP
  default).

## System configuration & identity

- **FR-11 [GENERIC]** The system must identify itself as OWN-OS (not as
  "Linux From Scratch") via `/etc/os-release` and equivalents
  (`docs/phase1/08-finalization-and-post-lfs.md` §8.1, D13).
- **FR-12 [GENERIC]** The system must have a defined, intentional init
  system — currently SysVinit (`docs/phase1` D7) — with documented
  run-level/service behavior (`docs/phase1/06-system-configuration.md`).
- **FR-13 [GENERIC]** The system must have a defined default locale
  (currently `C.UTF-8`, `docs/phase1` D10) applied consistently across
  console and shell.

## Package & capability management

- **FR-14 [GENERIC]** Adding any capability beyond the Phase 1 base set
  must be a deliberate, documented decision (a new or updated
  requirement in this folder), never an ad hoc install with no record.
- **FR-15 [DEPENDS ON D15]** If any persona needing ongoing package
  upgrades/removals at scale is confirmed, a package-management
  approach must be formally chosen (`docs/phase1/05-base-system-build.md`
  §5.3 catalogues the candidate techniques) — currently explicitly
  deferred (`docs/phase1` D6).

## Graphical stack (entirely conditional)

- **FR-16 [DEPENDS ON D15 — P2 only]** If workstation use is confirmed:
  a display server, a window manager or desktop environment, and basic
  end-user applications (browser at minimum) must be specified in a
  follow-on requirements document before any BLFS work starts on this
  axis.
- **FR-17 [DEPENDS ON D15 — P2 only]** If workstation use is confirmed:
  input device handling (keyboard layout beyond console defaults,
  pointer devices) must be specified.

## Service stack (entirely conditional)

- **FR-18 [DEPENDS ON D15 — P3 only]** If server/appliance use is
  confirmed: the specific service(s) to run must be named, each as its
  own requirement with its own success criteria, before any BLFS work
  starts on this axis.

## Documentation & maintainability

- **FR-19 [GENERIC]** Every requirement in this folder that is marked
  **[ASSUMED — confirm]** anywhere must be resolved (confirmed or
  corrected) and logged in `08-open-questions-and-decision-log.md`
  before the capability it gates is implemented.
- **FR-20 [GENERIC]** Security-advisory monitoring for every shipped
  package must be an assigned, ongoing responsibility, not a one-time
  Phase 1 task (`docs/phase1/08-finalization-and-post-lfs.md` §8.6,
  `docs/phase1` D16).

## Traceability note

`FR-01` through `FR-13` and `FR-19`–`FR-20` are all already fully
addressed by the existing `docs/phase1/` plan — they're restated here
so this folder is a complete, standalone statement of *what* the system
does, with `docs/phase1/` as the *how*. `FR-14`–`FR-18` are the ones
genuinely gated on the open persona question in Document 2 and should
not be started until that's resolved.
