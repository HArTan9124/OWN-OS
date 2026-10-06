# 07 — Success Criteria & Metrics

Success criteria at two levels: Phase 1 (concrete, measurable, already
achievable with the current plan) and product-level (gated on
`02-target-users-and-use-cases.md`).

## Phase 1 success criteria (binary, checkable today)

Directly maps to `docs/phase1/10-roadmap-and-open-decisions.md` §10.2,
M0–M10. Phase 1 is **done** when every one of these is independently
verifiable, not just "probably fine":

| Criterion | How verified |
|---|---|
| S1 | All 18 decisions (D1–D18) in `docs/phase1` Document 10 are resolved and recorded | Table in that document shows no "Needs sign-off" rows |
| S2 | Host toolchain passes `version-check.sh` | Script output, zero `ERROR:` lines |
| S3 | All ~90 packages + 6 patches downloaded and MD5-verified | Checksums match the book's published values |
| S4 | Cross-toolchain (Ch. 5) builds cleanly, as `lfs`, under `$LFS/tools` | Binutils/GCC/Glibc/libstdc++ present, correct triplet |
| S5 | Temporary tools (Ch. 6) build and the chroot is entered successfully | `chroot` prompt reached, `/etc/passwd`-based prompt resolves |
| S6 | Full base system (Ch. 8) builds with test suites run per the agreed policy (`docs/phase1` D18) | Build log reviewed, unexpected failures (not matching published LFS build logs) are zero |
| S7 | System configuration (Ch. 9) complete | Bootscripts, udev, network config, locale, inputrc/shells all present and reviewed |
| S8 | Kernel builds with every mandatory config option set (`docs/phase1/07-bootable-system.md` §7.2) and GRUB is installed/configured | `.config` reviewed against the mandatory list; `grub.cfg` reviewed |
| S9 | **First successful reboot into OWN-OS's own kernel, reaching a login prompt, with no dependency on the build host at that point** | Physical/observed boot — this is the actual Phase 1 finish line |
| S10 | Root password set, system identity files (`/etc/os-release` etc.) rebranded to OWN-OS | Manual check |
| S11 | Post-boot dev workflow operational (chroot-back-in or SSH, per `docs/phase1` D14) | Demonstrated by actually using it once |
| S12 | Security-advisory monitoring responsibility assigned and a recurring check scheduled | A named owner + an actual scheduled mechanism exists, not just an intention |

**Phase 1 is complete when S1–S12 are all true.** This is intentionally
a hard, binary bar — there is no "mostly done."

## Product-level success criteria (gated, to be filled in per persona)

Cannot be meaningfully written until `02-target-users-and-use-cases.md`
§4 is resolved. Placeholder structure for whichever persona(s) are
confirmed:

- **If P1/P5 (builder / reference artifact):** success = Phase 1's S1–S12
  above, plus this documentation set (`docs/requirements/` +
  `docs/phase1/`) being complete and accurate enough that a third party
  could rebuild OWN-OS from zero using only these docs. No further
  metric needed — the build process *is* the deliverable.
- **If P2 (workstation):** success metrics to be added once FR-16/FR-17
  (`03-functional-requirements.md`) are scoped — likely: boots to a
  usable graphical session, can browse the web, can edit/save a
  document, survives a normal day of interactive use without manual
  intervention.
- **If P3 (server/appliance):** success metrics to be added once the
  specific service (FR-18) is named — likely: service starts on boot
  unattended, responds correctly to its expected workload, survives a
  reboot without manual reconfiguration.
- **If P4 (embedded/minimal):** success metrics to be added once NFR-02/
  NFR-03 get concrete targets — likely: boots within a stated time
  budget, fits within a stated size budget, on the actual target
  hardware/image format.

## Metric ownership

Every metric above with "to be added" needs an owner and a measurement
method recorded in
[08-open-questions-and-decision-log.md](./08-open-questions-and-decision-log.md)
at the same time it's written — a success criterion with no way to
check it is not a success criterion.
