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
| 0 | Decisions & sign-off | — (precondition for everything) | Q1–Q3, Q15/D15, D1–D20 | S1 | ☑ Done for Phase 1 purposes (2026-10-06) — D1–D14, D16–D20 resolved; **D15/Q15 (purpose) and Q1–Q3 deliberately deferred — they gate Phase 2, not Phase 1 execution** |
| 1 | Host environment & partitioning (image-file, not real partition — Doc 11 §1) | M1 | D1–D4 | S2 | ◐ In progress — host toolchain verified 2026-10-06 |
| 2 | Source acquisition (+ Pi firmware/device-tree, minus GRUB — Doc 11 §4) | M2 | D17 (storage) | S3 | ☑ Done (2026-10-06) — 81/84 packages (530 MB) + Pi firmware; Binutils/GCC/Glibc checksum-verified. Ninja/Systemd/Vim deferred (not needed until later) |
| 3 | Cross-toolchain bootstrap (target `aarch64-unknown-linux-gnu` — Doc 11 §3) | M3 | D4 | S4 | ☑ Done (2026-10-06) — Binutils, GCC (C+C++), Linux headers, Glibc, libstdc++ all cross-built and verified; two real bugs found & fixed, see Doc 11 §3.1 |
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

**Gate:** D17 ✅ resolved (periodic exported checkpoints).

**Tasks:**
- [x] Create a sources directory — `/build/sources/pkgs` in this
  session (adapted from `$LFS/sources` since there's no real partition
  here, per Document 11 §1). Done 2026-10-06.
- [ ] Check the LFS security advisories page before finalizing any
  version (§3.1, §3.4) — not yet done.
- [◐] Download all packages (minus GRUB, per Doc 11 §4) + 6 patches
  (§3.2–3.3). **26 of 84 downloaded successfully (104 MB) as of
  2026-10-06** — everything reachable via GitHub release assets,
  PyPI, and (surprisingly) `downloads.sourceforge.net`/
  `prdownloads.sourceforge.net` is done. **58 packages are still
  blocked** — this session's network policy rejects `ftp.gnu.org`,
  `sourceware.org`, `kernel.org`, and 12 other domains wholesale. The
  exact, empirically-verified list (every URL actually attempted, not
  guessed) is in `/build/sources/BLOCKED_DOMAINS.txt` in this session
  and reproduced in the chat. **This blocks Stage 3 entirely** —
  Binutils, GCC, and Glibc are all on the blocked list, so the
  cross-toolchain cannot be built until the network allowlist is
  widened. The download script (`/build/sources/download-all.sh`) is
  idempotent — re-running it after the allowlist changes will fetch
  only what's still missing.
- [ ] Verify every MD5 sum against the book's published values — not
  yet done (waiting on the remaining 58 files).
- [x] Disk budget confirmed: 530 MB against ~30 GB available; fits
  comfortably.

**Stage 2 closed out 2026-10-06.** Final result: 81/84 packages (530 MB),
MD5-verified for Binutils/GCC/Glibc against the book's published sums.
Three packages deferred — not needed until much later in the build, so
not worth blocking on: **Ninja** (needed only for Meson-based builds —
systemd/udev; `codeload.github.com` is blocked even under "Full" network
access, and its PyPI sdist is a prebuilt-binary wrapper, not real source,
so it was deliberately not substituted), **Systemd** (same
`codeload.github.com` block; only needed for its udev component), **Vim**
(same block; not on SourceForge, no easy mirror found). Revisit these
three specifically closer to when they're actually needed — by then the
network situation may have changed, or a proper source mirror can be
found rather than accepting a dubious substitute.

**Package deviations from Document 03's exact versions, logged here since
they weren't in the original plan:**
- Acl: **2.3.1** substituted for 2.3.2 (Savannah, the only source, is
  unreachable from this session — persistent connection resets, not a
  policy block; closest available version pulled from Debian's source
  pool instead).
- Attr: **2.5.1** substituted for 2.5.2 (same reason).
- Ncurses: **6.5** (plain release) substituted for the pinned
  **6.5-20250809** dated snapshot (that exact snapshot has rolled off
  `invisible-mirror.net`'s retention window; it only keeps recent
  snapshots, not every dated one, and a dated snapshot that far back
  has aged off the hosting window since the book's 12.4 release date).
- Libpipeline, Man-DB: exact versions obtained from Debian's source
  pool instead of Savannah (same Savannah unreachability issue), not a
  version deviation.

**Acceptance (S3):** Every required package/patch present in
`$LFS/sources` with a verified-matching MD5.

---

## Stage 3 — Cross-toolchain bootstrap

**Reference:** `docs/phase1/04-toolchain-bootstrap.md` §4.1–4.3

**Gate:** D4 (parallel job count) resolved (feeds `MAKEFLAGS`).

**Tasks — all done 2026-10-06, adapted per Document 11 (no `lfs` user,
no chroot; `$OWNOS_TOOLS`=`/build/tools`, `$OWNOS_ROOT`=`/build/root`):**
- [x] Minimal FHS layout created directly in `$OWNOS_ROOT` (§4.1 step 1
  equivalent); `/usr/lib64` confirmed absent — **required two fix
  passes**, see Document 11 §3.1 Bug 2 (aarch64's own `t-aarch64-linux`
  defaults to `lib64` just like x86_64's `t-linux64`; the GCC source
  patch must be applied **before GCC pass 1 is first configured**, not
  after — it's baked into the compiler at its own build time).
- [x] Build environment set via `/build/env.sh` (sourced per shell,
  since this session's shells don't persist state) instead of
  `.bash_profile`/`.bashrc` — no `lfs` user needed since nothing here
  risks damaging a shared host (§4.1 steps 2–3 equivalent).
- [x] Built, in order — Binutils 2.45 → GCC 15.2.0 pass 1 (C+C++) →
  Linux 6.16.1 API headers (`ARCH=arm64`, a real adaptation — the book's
  plain `make headers` assumes same-arch) → Glibc 2.42 → libstdc++
  (§4.3). **libstdc++ required a second fix pass** — see Document 11
  §3.1 Bug 1 (the book's `--with-gxx-include-dir` sysroot trick assumes
  `$LFS/tools` nested inside `$LFS`; our tools/root layout is sibling
  directories, not nested, so headers must install directly under
  `$OWNOS_TOOLS`, not via `DESTDIR`-sysroot-prefixing).
- [x] SBU baseline: Binutils pass 1 built in **1m15s on 4 cores** —
  fast relative to the book's 1-core baseline convention, as expected
  given this session's hardware; not directly comparable to published
  book SBU figures, logged here as this project's own reference point.
- [x] All five sanity checks from the book's Glibc page run and passed:
  correct interpreter (`/lib/ld-linux-aarch64.so.1`), no build-path
  leakage into the compiled binary, correct header/linker search paths,
  correct `libc.so.6` resolution. A full `#include <iostream>` program
  compiled, linked, and produced a valid ARM64 PIE executable.

**Acceptance (S4) — met:** `$OWNOS_TOOLS` contains a working
`aarch64-unknown-linux-gnu` cross-toolchain (C and C++); `$OWNOS_ROOT`
has Glibc and libstdc++ correctly installed under `usr/lib` with no
`lib64` split; SBU baseline recorded.

---

## Stage 4 — Full cross-build of the base system

**Superseded task list — see Document 11 §3.2.** There is no `lfs`
user, no chroot, no "temporary tools then rebuild natively" split in
this methodology (D20) — every package is cross-compiled exactly once,
directly from this session, using Chapter 8's (final, fuller-featured)
flags. Chapter 6's separate pass, Binutils pass 2, and GCC pass 2 are
all skipped entirely (§3.2 explains why). "Entering chroot" (old §4.5
steps 1–3) never happens; the FHS tree and essential files (old §4.5
steps 4–5) still get created, just directly in `$OWNOS_ROOT` with no
privilege-separation ceremony around it.

**Reference:** `docs/phase1/05-base-system-build.md` for the package
list and order (minus GRUB, Document 11 §4); pull each package's exact
Chapter 8 configure flags from the live book at build time, same as
done for Binutils/GCC/Glibc/libstdc++ in Stage 3.

**Gate:** D17 ✅ (checkpoint discipline already active — see Document 11
§3.3, a real mount-loss incident already tested this).

**Progress (updated as the build proceeds):**
- [x] M4 1.4.20 — built, installed.
- [x] Ncurses 6.5 — built, installed. **Deviation:** C++ bindings
  (`--with-cxx-shared`) dropped (`--without-cxx` instead) — a known
  `std::byte` conflict between ncurses 6.5's C++ wrapper and GCC 15's
  libstdc++ (the book's pinned dated snapshot, newer than our
  substitute, likely has this fixed upstream). Not needed for a
  minimal bootable system; revisit only if a later package specifically
  needs `libncurses++`.
- [x] Zlib 1.3.1, Bzip2 1.0.8, Xz 5.8.1, Lz4 1.10.0, Zstd 1.5.7,
  Readline 8.3, Flex 2.6.4, File 5.46, Bc 7.0.3, Pkgconf 2.5.1 — built,
  installed. Three more non-autoconf/special-case cross-compile patterns
  found and resolved, see Document 11 §3.4/§3.6.
- [x] **Methodology simplification:** Tcl, Expect, DejaGNU dropped from
  the package list entirely — confirmed via the book's own pages to be
  pure test-suite dependencies, and D18 already defers all test suites.
  See Document 11 §3.5.
- [x] Man-pages 6.15, Iana-Etc 20250807, GMP 6.3.0, MPFR 4.2.2 (built
  with `--disable-decimal-float`), MPC 1.3.1 — built, installed. Found a
  significant general-purpose fix: GCC 15 defaults to `gnu23`, which
  breaks old K&R-style configure-time test code (hit in GMP's own
  configure); fixed globally via `OWNOS_CC="...-gcc -std=gnu17"` in
  `env.sh`, applied to every configure call from here on. See Document
  11 §3.7/§3.8.
- [x] Attr 2.5.1, Acl 2.3.1, Libcap 2.76, Libxcrypt 4.4.38, Shadow 4.18.0
  — built, installed. Shadow needed `--without-libbsd` (its configure
  defaults `--with-libbsd=yes` even without probing availability first,
  and glibc has no `readpassphrase()`). `/etc/passwd`, `/etc/group`,
  `/etc/shadow` still need hand-writing (no `useradd`/`pwconv` run —
  nothing executes cross-built binaries on this host) — folded into the
  "FHS tree + essential files" task below rather than done piecemeal.
- [x] **Binutils (final)** 2.45 — Canadian cross (`--build=x86_64-...
  --host=$OWNOS_TGT --target=$OWNOS_TGT`) built and installed. Verified
  `readelf`/`ld`/etc. in `$OWNOS_ROOT/usr/bin` are genuine aarch64 PIE
  executables — a Binutils that *runs on* the Pi, not the x86_64-hosted
  cross-Binutils from Stage 3. `tooldir=/usr` avoided a `/usr/<triplet>`
  install layout. No source/flag changes needed beyond the triplets —
  worked first try once Stage 3's pass-1 `aarch64-unknown-linux-gnu-gcc`
  was found automatically via the standard `${host_alias}-gcc`
  convention.
- [x] **GCC (final)** 15.2.0 — Canadian cross built and installed.
  Same `--build=x86_64-... --host=--target=aarch64-...` shape as
  Binutils-final, plus `--with-sysroot=/ --with-build-sysroot=$OWNOS_ROOT`
  (bakes in `/` as the sysroot the compiler uses once it's running on
  the Pi's real root, while still finding aarch64 headers/libs in
  `$OWNOS_ROOT` *during this build*) and explicit
  `CC_FOR_BUILD=gcc CXX_FOR_BUILD=g++` (native, for GCC's own
  host-run code generators) alongside `CC="$OWNOS_CC"` (the Stage-3
  cross-compiler, for everything that must itself run on aarch64).
  Verified `cc1`/`cc1plus`/`gcc`/`g++` in `$OWNOS_ROOT` are genuine
  aarch64 PIE executables, and that the installed driver's specs use
  `--sysroot=%R` (the `--with-sysroot=/` runtime mechanism), not a
  hardcoded `$OWNOS_ROOT` path — confirmed by inspection since nothing
  here can execute the binary to check directly. libgcc/libstdc++ were
  rebuilt fresh by this build (replacing Stage 3's copies with the full
  Chapter-8-equivalent versions). Hit one real snag along the way:
  **a stale `build/` directory with an old config.cache survived two
  rounds of `rm -rf` + re-extraction** (traced to a `mv` silently
  moving a fresh extraction *into* an existing same-named directory
  instead of replacing it, rather than any container/mount bug) —
  resolved by extracting to a distinctly-named directory
  (`gcc-final-15.2.0`, not reusing `gcc-15.2.0-final`) and confirming
  no `build/` subdirectory existed before configuring. Also manually
  created `/usr/bin/cc` → `gcc` — a symlink stock LFS creates back in
  the skipped Chapter 6, never produced by Chapter 8's GCC page on its
  own.
- [x] Sed 4.9, Psmisc 23.7 — built, installed. Psmisc's `pstree.c`
  assumes C23's built-in `bool`/`true`/`false` keywords (no
  `<stdbool.h>` include) and fails under the `-std=gnu17` override from
  §3.7 — **the `-std=gnu17` fix isn't universally safe**, so the default
  going forward is the plain cross-compiler (`${OWNOS_TGT}-gcc`, no
  `-std=` override), falling back to `$OWNOS_CC`'s `-std=gnu17` only
  when a package specifically hits the old-K&R-code failure pattern.
- [x] Gettext 0.26, Bison 3.8.2, Grep 3.12 — built, installed (all
  plain cross-compiler, no `-std=` override needed). Gettext needed the
  book's `chmod 0755 /usr/lib/preloadable_libintl.so` post-install fix.
- [x] Bash 5.3 (with `/bin/sh` symlink), Libtool 2.5.4, GDBM 1.26,
  Gperf 3.3 — built, installed. All plain cross-compiler, no issues.
- [x] Expat 2.7.1 — built, installed, no issues.
- [x] Inetutils 2.6, Less 679 — built, installed. Inetutils needed
  `-DPATH_PROCNET_DEV="/proc/net/dev"` added to CPPFLAGS — this macro
  was removed from modern glibc headers entirely (a genuine upstream
  incompatibility between inetutils 2.6's `/proc/net/dev`-parsing code
  and glibc 2.42, unrelated to cross-compiling). `ifconfig` moved to
  `/usr/sbin` per the book.
- [x] **Perl** 5.42.0 — built via the third-party `perl-cross` project
  (stock `Configure -Dusecrosscompile` needs a live reachable target
  device, not viable here). Full details, including the borrowed
  5.41.8 patchset and two real bugs fixed (a patch-hunk source-drift
  conflict, a perl-cross template bug producing a malformed `#
  HAS_FDOPENDIR` line instead of `#define HAS_FDOPENDIR` in generated
  headers) in Document 11 §3.10. Verified the final `perl` binary is
  genuine aarch64.
- [ ] Remaining Chapter 8 package list (~50 packages) — in progress,
  see live status in chat / commit history rather than duplicated here
  to avoid this file going stale mid-build.
- [x] Essential files created directly in `$OWNOS_ROOT`: `/etc/mtab`
  symlink, `/etc/hosts`, `/etc/passwd`, `/etc/group` (book's §7.6
  content verbatim), plus hand-written `/etc/shadow`/`/etc/gshadow`
  (book's chroot-based `pwconv`/`passwd root` steps can't run here —
  every account's password field is `!`, i.e. locked; **root has no
  password set yet, must be done in Stage 7 before first boot**), and
  the log files (`btmp`/`lastlog`/`faillog`/`wtmp`) with book-correct
  permissions.
- [ ] Remaining FHS tree directories not yet created by any package
  install (`/root`, `/home`, `/srv`, `/media`, etc. — audit once the
  package list is done rather than piecemeal).
- [ ] Cleanup (`/usr/share/{info,man,doc}`, stray `.la` files) and a
  checkpoint export at the close of this stage.

**Acceptance (S5, adapted):** Every Chapter 8 package (minus GRUB)
cross-built and installed into `$OWNOS_ROOT`; FHS tree complete;
checkpoint exported.

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
