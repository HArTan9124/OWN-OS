# 00 — Phase 1 Execution Plan

This is the single actionable document for Phase 1. It divides Phase 1
into **10 sequential stages**, and each stage fuses the two halves of
the plan that previously lived in separate folders:

- **the gate** — which open questions / requirements
  (`docs/requirements/`) must be resolved before the stage starts, and
- **the work** — which technical steps (`docs/phase1/01`–`09`) actually
  get executed.

Documents `01`–`09` in this folder remain the detailed reference
("why this flag," "why this order"). This document is the "what do I
do, in what order, and how do I know a stage is actually finished"
layer on top of them — **start here when it's time to actually build**.

> **⚠ Target changed to Raspberry Pi 4/5 (`aarch64`) on 2026-10-06 — read
> [11-arm-raspberry-pi-adaptation.md](./11-arm-raspberry-pi-adaptation.md)
> before working any stage below.** This document was originally written
> for an x86_64 build booting via BIOS/GRUB with a native chroot phase.
> Document 11 replaces: the chroot-and-build-natively mechanism (now full
> cross-compilation throughout, no chroot at all), the entire GRUB-based
> boot stage (now Raspberry Pi firmware boot), and the test-suite timing
> (deferred to after first boot on real hardware). Stages **3, 4, and 5**
> below are the most affected — their task lists still describe the
> original x86_64/chroot mechanics and have not yet been rewritten
> line-by-line for the cross-build approach; treat Document 11 as
> overriding them wherever the two disagree, and expect this file's
> per-stage task lists to be revised to match as each stage is actually
> executed.

## How to use this document

- Work stages **in order, top to bottom**. They are drawn as strictly
  sequential because that's how the underlying LFS process actually
  works (Document 01 §1.6) — there is no meaningful way to parallelize
  Stage 3 with Stage 2, for example.
- Every stage has a **Gate** — if any item in it is unresolved, stop and
  resolve it (in `docs/requirements/08` and/or `docs/phase1/10`) before
  doing the stage's tasks. A stage started with an open gate item is the
  single most likely way this project re-does work later.
- Every stage ends with **Acceptance**, tied to the `S1`–`S12` criteria
  in `docs/requirements/07-success-criteria-and-metrics.md`. A stage
  isn't done because the commands ran without error — it's done when
  its acceptance row is true.
- Check boxes as you go. This file is meant to be edited in place as
  the build progresses (unlike `docs/phase1/01-09` and
  `docs/requirements/01-09`, which are reference/planning material and
  change rarely).

## Stage tracker

| Stage | Name | Maps to (phase1 milestone) | Gate clears | Acceptance | Status |
|---|---|---|---|---|---|
| 0 | Decisions & sign-off | — (precondition for everything) | Q1–Q3, Q15/D15, D1–D20 | S1 | ◐ In progress — D1, D12, D18–D20 resolved 2026-10-06; D2–D11, D13–D17 and Q1–Q3/Q15 still open |
| 1 | Host environment & partitioning (image-file, not real partition — Doc 11 §1) | M1 | D1–D4 | S2 | ◐ In progress — host toolchain verified 2026-10-06 |
| 2 | Source acquisition (+ Pi firmware/device-tree, minus GRUB — Doc 11 §4) | M2 | D17 (storage) | S3 | ☐ Not started |
| 3 | Cross-toolchain bootstrap (target `aarch64-unknown-linux-gnu` — Doc 11 §3) | M3 | D4 | S4 | ☐ Not started |
| 4 | **Full cross-build, no chroot** (replaces "temp tools + chroot entry" — Doc 11 §1/§3) | M4 | D17, D20 | S5 | ☐ Not started |
| 5 | System configuration (unaffected by arch — applied directly, no chroot) | M5 | D7–D10 | S7 | ☐ Not started |
| 6 | **Image assembly** (replaces "bootable system"/GRUB — Doc 11 §5) | M6 | D11, D19 | S8 | ☐ Not started |
| 7 | Identity & final review (no chroot-exit step) | M7 | D13 | S10 | ☐ Not started |
| 8 | **First boot on real Pi** (flash SD card, power on) | M8 | D14 | S9, S11 | ☐ Not started |
| 9 | Phase 1 closeout | M9 | D16, D18 | S12 + all of S1–S11 | ☐ Not started |

Stage numbering above (0–9) is kept stable as a reference frame; the
*content* of Stages 4–8 changed substantially from what's written in
their detailed sections further down (still the original x86_64/chroot/
GRUB task lists — see the warning at the top of this document). Treat
the table above, not the prose below it, as current until those
sections are rewritten.

Update the **Status** column (☐ Not started / ◐ In progress / ☑ Done)
as the single source of truth for "where are we" — don't track progress
anywhere else.

---

## Stage 0 — Decisions & sign-off

**Goal:** Make sure nothing downstream gets built on an assumption that
later turns out wrong. This stage produces no OS artifact at all — its
only deliverable is a fully resolved decision log.

**Gate:** None (this is the gate for everything else).

**Tasks:**
- [ ] Resolve **Q15 / D15** first — OWN-OS's actual purpose/persona
  (`docs/requirements/02-target-users-and-use-cases.md` §4). Minimum
  acceptable answer for now: explicitly confirm the default
  ("P1 builder + P5 reference artifact only; Phase 2 personas
  deferred") rather than leaving it silently assumed.
- [ ] Resolve **Q1–Q3** (team size, motivation, threat model —
  `docs/requirements/08-open-questions-and-decision-log.md`).
- [ ] Resolve **D1–D18** (`docs/phase1/10-roadmap-and-open-decisions.md`
  §10.1) — every row currently marked "Needs sign-off." Pay particular
  attention to **D13** (OWN-OS's actual name/version/codename — needed
  concretely, not just "rebrand," before Stage 8 can write real files).
- [ ] Record every resolution with a date in
  `docs/requirements/08-open-questions-and-decision-log.md`'s Decision
  Log section, and update the corresponding row in
  `docs/phase1/10-roadmap-and-open-decisions.md` §10.1 to match — the
  two tables must never disagree.
- [ ] Re-read `docs/requirements/06-assumptions-constraints-dependencies.md`
  and confirm/correct A1–A6 in light of the above.

**Acceptance (S1):** Both decision tables show zero "Needs
sign-off"/"Open" rows — every item is either answered or explicitly,
dately deferred with a stated reason.

---

## Stage 1 — Host environment & partitioning

**Reference:** `docs/phase1/02-host-environment-and-partitioning.md`

**Gate:** D1 (architecture — ✅ resolved, `aarch64`/Pi 4/5), D2 (session
strategy), D3 (partition layout, superseded by Doc 11 §1 — image file,
not a real partition), D4 (parallelism) — D1 resolved in Stage 0,
D2/D3/D4 still open.

**Tasks:**
- [x] Run the book's `version-check.sh`-equivalent checks on the actual
  build host (this cloud session); resolve every gap. **Done
  2026-10-06**: Ubuntu 24.04 base, Bash 5.2.21, Binutils 2.42 (ld),
  Bison 3.8.2, GCC/G++ 13.3.0, Coreutils present, Diffutils 3.10,
  Findutils 4.9.0, Grep 3.11, Gzip 1.12, M4 1.4.19, Make 4.3, Patch
  2.7.6, Perl present, Python 3.13.16, Sed 4.9, Tar 1.35, Xz 5.4.5,
  kernel 6.18.44 — all within LFS's floor/ceiling. Two gaps found and
  fixed: `gawk` and `texinfo` were missing, installed via `apt-get
  install gawk texinfo`; confirmed `/usr/bin/awk` resolves to
  `/usr/bin/gawk` afterward. 4 cores / 15 GiB RAM available (meets the
  "≥4 cores, ≥8 GB" recommendation).
- [ ] Confirm the host kernel supports UNIX 98 PTY — moot for this
  build: Document 11 §6 (D20) defers all native test execution to
  after first boot on the real Pi, so host PTY support is not a build
  blocker here. Revisit only if the build methodology changes.
- [ ] ~~Partition the target disk~~ **Superseded by Doc 11 §1**: this
  session has no real disk to partition. Instead: create a loop-mounted
  disk image file (verified working in this container — root access,
  `losetup`, `mkfs.ext4`, and loop-mount all confirmed functional
  2026-10-06) sized per D3's eventual decision (10–30 GB; this
  container currently has ~30 GB free, so the lower end of that range
  is the realistic ceiling here).
- [ ] Format the image file `ext4`; no swap needed (cross-build, no
  native compilation of the heaviest packages happens on constrained
  target hardware).
- [ ] `export LFS=...`, `umask 022`; make both durable in the relevant
  shell profile(s) (§2.5).
- [ ] Mount the partition, fix ownership/mode, verify no `nosuid`/
  `nodev` on the mount (§2.6).
- [ ] If D2 = multi-session build: write down the actual resume
  checklist per chapter-range now, before it's needed (§2.2).

**Acceptance (S2):** `version-check.sh` output has zero errors;
`$LFS` is mounted, owned `root:root`, mode `755`, with correct options.

---

## Stage 2 — Source acquisition

**Reference:** `docs/phase1/03-sources-and-packages.md`

**Gate:** D17 (checkpoint/archive storage location) resolved — decide
where downloaded sources and later backups live before downloading,
since Stage 4's backup includes this directory.

**Tasks:**
- [ ] Create `$LFS/sources` with sticky-bit, world-writable permissions
  (§3.1).
- [ ] Check the LFS security advisories page before finalizing any
  version (§3.1, §3.4).
- [ ] Download all ~90 package tarballs + 6 patches (§3.2–3.3).
- [ ] Verify every MD5 sum against the book's published values — do not
  skip this (§3.4).
- [ ] Confirm total disk budget still holds given actual downloaded +
  expected build sizes (ties back to Stage 1 partition sizing).

**Acceptance (S3):** Every required package/patch present in
`$LFS/sources` with a verified-matching MD5.

---

## Stage 3 — Cross-toolchain bootstrap

**Reference:** `docs/phase1/04-toolchain-bootstrap.md` §4.1–4.3

**Gate:** D4 (parallel job count) resolved (feeds `MAKEFLAGS`).

**Tasks:**
- [ ] Minimal FHS layout (`etc`, `var`, `usr/{bin,lib,sbin}`,
  `lib64` symlink logic, `tools`); confirm `/usr/lib64` does **not**
  exist (§4.1 step 1).
- [ ] Create the `lfs` user/group, set ownership of `$LFS` subtrees
  (§4.1 step 2).
- [ ] Write `~/.bash_profile` and `~/.bashrc` for `lfs` exactly as
  specified (clean-room env, `LFS_TGT`, `PATH` ordering,
  `CONFIG_SITE`, `MAKEFLAGS=-j<D4 value>`); neutralize host
  `/etc/bash.bashrc` if present (§4.1 step 3).
- [ ] **As `lfs`, never `root`:** build, in order — Binutils pass 1 →
  GCC pass 1 → Linux API headers → Glibc → libstdc++ (§4.3).
- [ ] Time the Binutils pass-1 build specifically (wrap in `time { }`)
  to establish the project's SBU baseline
  (`docs/phase1/09-testing-validation-and-risk.md` §9.2) — this feeds
  every later time estimate.

**Acceptance (S4):** `$LFS/tools` contains a working cross-toolchain
targeting `$LFS_TGT`; SBU baseline recorded.

---

## Stage 4 — Temporary tools + chroot entry

**Reference:** `docs/phase1/04-toolchain-bootstrap.md` §4.4–4.6

**Gate:** D17 (checkpoint storage) — confirm the backup target is ready
before the end-of-stage checkpoint.

**Tasks:**
- [ ] **As `lfs`:** cross-compile, in order — M4 → Ncurses → Bash →
  Coreutils → Diffutils → File → Findutils → Gawk → Grep → Gzip → Make
  → Patch → Sed → Tar → Xz → Binutils pass 2 → GCC pass 2 (§4.4).
- [ ] **As `root`:** `chown` the whole `$LFS` tree back to `root:root`
  (§4.5 step 1).
- [ ] Mount virtual kernel filesystems (`dev`, `devpts`, `proc`, `sys`,
  `run`, `dev/shm`) into `$LFS` (§4.5 step 2).
- [ ] Enter chroot with the exact clean environment specified (§4.5
  step 3).
- [ ] Build the full FHS directory tree; re-confirm `/usr/lib64`
  absent (§4.5 step 4).
- [ ] Create essential files: `/etc/mtab`, `/etc/hosts`,
  **first** `/etc/passwd`/`/etc/group` (fixed GIDs, incl. `tty`=5), the
  temporary `tester` account, initialized log files (§4.5 step 5).
- [ ] **Natively, inside chroot:** build Gettext → Bison → Perl →
  Python → Texinfo → Util-linux (§4.5 step 6).
- [ ] Clean up (`/usr/share/{info,man,doc}`, stray `.la` files, delete
  `/tools`) and **take the Chapter-7 backup checkpoint**, exported to
  the D17 location, not just local disk (§4.5 step 7,
  `docs/phase1/09` §9.3).

**Acceptance (S5):** Chroot entered successfully (prompt resolves once
`/etc/passwd` exists); all final temporary tools built; checkpoint
archive verified to exist outside the ephemeral build environment.

---

## Stage 5 — Base system build

**Reference:** `docs/phase1/05-base-system-build.md`

**Gate:** D5 (stripping policy), D6 (package-management strategy — in
practice "none" for Phase 1), D18 (test-suite scope) resolved.

**Tasks:**
- [ ] Confirm build-wide policy before starting: no custom optimization
  flags, `--disable-static` except Glibc/GCC (§5.1).
- [ ] Build the full ~90-package sequence in book order (§5.2) — do not
  reorder; the sequence is dependency-derived.
- [ ] Per D18: run test suites for every package that offers one
  (not just Binutils/GCC/Glibc), from this stage onward
  (`docs/phase1/09` §9.1). Cross-check any unexpected failure against
  the LFS published build logs before treating it as a real defect.
- [ ] Rebuild Glibc's extra upgrade-safety notes are *not* needed for
  this from-scratch build but worth a read now for future-upgrade
  context (§5.2.1).
- [ ] Apply the D5 stripping decision (§5.4 step 1) or explicitly skip
  it, consistently with what Stage 0 decided.
- [ ] Clean up: purge `/tmp`, remove `.la` files, remove
  `$LFS_TGT`-prefixed cross-compiler remnants, `userdel -r tester`
  (§5.4 step 2).
- [ ] Take the **second checkpoint** (end of Chapter 8 — recommended in
  `docs/phase1/09` §9.3 beyond the book's single official one), exported
  per D17.

**Acceptance (S6):** All ~90 packages built; test-suite results
reviewed against D18 scope with zero unexplained failures; cleanup
done; second checkpoint exported.

---

## Stage 6 — System configuration

**Reference:** `docs/phase1/06-system-configuration.md`

**Gate:** D7 (init system), D8 (network naming), D9 (static/DHCP), D10
(locale) resolved.

**Tasks:**
- [ ] Install LFS-Bootscripts; confirm D7 = SysVinit is what's actually
  being configured below (§6.2).
- [ ] Review udev rules for the actual target hardware/VM; pre-empt the
  known gotchas (module not auto-loading, unwanted module loading,
  naming-order instability) rather than discovering them at boot
  (§6.3).
- [ ] Apply D8 (persistent vs. legacy interface naming).
- [ ] Apply D9: write `/etc/sysconfig/ifconfig.<iface>`,
  `/etc/resolv.conf`, `/etc/hostname`, `/etc/hosts` (§6.4).
- [ ] Write `/etc/inittab` (confirm default run level **3**),
  `/etc/sysconfig/clock` (confirm UTC vs. local hardware clock
  correctly — easy to silently get backwards), `/etc/sysconfig/console`,
  `/etc/sysconfig/rc.site` (§6.5).
- [ ] Apply D10: write `/etc/profile` with the decided default locale
  (§6.6).
- [ ] Write `/etc/inputrc`, `/etc/shells` (§6.7).

**Acceptance (S7):** Every file above present and internally
consistent with the Stage 0 decisions that gated it; no leftover
book-example placeholder values (e.g. example IPs/hostnames) left
unedited.

---

## Stage 7 — Bootable system

**Reference:** `docs/phase1/07-bootable-system.md`

**Gate:** D11 (device-path vs. UUID/PARTUUID scheme), D12 (BIOS vs.
UEFI) resolved — **and D11/D12 must agree with each other and with
what Stage 1 actually partitioned.**

**Tasks:**
- [ ] Write `/etc/fstab` using the D11-decided identification scheme
  consistently for every entry (§7.1).
- [ ] `make mrproper`, then configure the kernel from `defconfig` +
  **every mandatory setting** listed in §7.2 (devtmpfs, cgroups,
  KASLR, stack protector, `WERROR` off, DRM/framebuffer chain,
  x2APIC chain if x86_64, **NVMe support if the target disk is NVMe —
  non-negotiable**).
- [ ] Build kernel + modules; install `vmlinuz`/`System.map`/`config`/
  `Documentation` into `/boot` (§7.2 steps 4–5).
- [ ] Apply the USB load-order modprobe fix (§7.2 step 6).
- [ ] Decide and apply kernel-source retention (`chown -R 0:0` if
  keeping it; never symlink `/usr/src/linux`) (§7.2 step 7).
- [ ] **Have rescue media ready before running `grub-install`** — this
  is a precondition, not a nice-to-have (§7.3 safety note).
- [ ] `grub-install` to the correct disk per D12; hand-write
  `grub.cfg` using the D11-consistent identification scheme (§7.3).

**Acceptance (S8):** `/etc/fstab` and `grub.cfg` use the same, correct
disk-identification scheme; kernel `.config` reviewed line-by-line
against the mandatory list; rescue media confirmed working *before*
the next stage's reboot.

---

## Stage 8 — First boot & validation

**Reference:** `docs/phase1/08-finalization-and-post-lfs.md`

**Gate:** D13 (OWN-OS's actual name/version/codename — must be a real,
decided value by now, not a placeholder), D14 (post-boot workflow)
resolved.

**Tasks:**
- [ ] Write `/etc/lfs-release` (or OWN-OS equivalent), `/etc/lsb-release`,
  `/etc/os-release` with the **actual** D13 identity (§8.1).
- [ ] Set the root password (§8.2 — do not skip).
- [ ] Final review pass: `/etc/fstab`, `/etc/hosts`, `/etc/inputrc`,
  `/etc/profile`, `/etc/resolv.conf`, network config file(s) (§8.2).
- [ ] Exit chroot, unmount every virtual filesystem and the LFS
  partition itself, in the documented order (§8.3).
- [ ] **Reboot.** This is the Phase 1 finish line.
- [ ] If it fails: consult the LFS troubleshooting index first, then
  the risk register (`docs/phase1/09` §9.4) and this project's own
  Chapter-7/Chapter-8 checkpoints before considering a from-scratch
  restart.
- [ ] Once booted: stand up the D14-decided post-boot workflow
  (chroot-back-in and/or SSH) and actually use it once, successfully
  (§8.4).

**Acceptance (S9, S10, S11):** Reboot reaches a login prompt on OWN-OS's
own kernel/toolchain/bootloader with no host dependency; identity files
correctly rebranded; root password set; post-boot workflow demonstrated
working at least once.

---

## Stage 9 — Phase 1 closeout

**Reference:** `docs/phase1/08-finalization-and-post-lfs.md` §8.5–8.7,
`docs/requirements/07-success-criteria-and-metrics.md`

**Gate:** D16 (security-advisory monitoring ownership) resolved.

**Tasks:**
- [ ] Assign D16 concretely — a named owner and an actual scheduled
  mechanism (not just "someone should check sometime") (§8.6).
- [ ] Walk `docs/requirements/07-success-criteria-and-metrics.md` S1–S12
  top to bottom; confirm every one is true with evidence (not from
  memory).
- [ ] Update the Stage tracker table at the top of this document — every
  row should now read ☑ Done.
- [ ] Record the Phase 1 completion date in
  `docs/requirements/08-open-questions-and-decision-log.md`'s decision
  log.
- [ ] **Do not start Phase 2 work yet.** Phase 2 scope is entirely gated
  on Q15/D15 (persona) being resolved with a *real* answer, not just
  the Stage-0 minimum default — revisit
  `docs/requirements/05-scope-and-out-of-scope.md` §2 before committing
  to anything past this point.

**Acceptance (S12 + all prior):** Every S1–S12 criterion independently
verified true. Phase 1 is complete.
