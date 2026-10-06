# 11 — ARM / Raspberry Pi 4/5 Adaptation

**Status:** Supersedes the x86_64/BIOS/GRUB assumptions in Documents
01–10 wherever they conflict with this document. Written after the
target was confirmed as a **Raspberry Pi 4 or 5** with the build
happening in this cloud session (not on the Pi itself, not in a
desktop VM). Documents 01–10 remain valid for everything that's
architecture-agnostic (package rationale, LFS methodology theory,
system-configuration content, risk-register structure, testing
philosophy) — this document states exactly what changes and why.

## 1. What changed and why

The original plan assumed x86_64 building for x86_64, booting via
BIOS/GRUB — stock LFS's native case. A Raspberry Pi is a genuinely
different target architecture (ARM64/`aarch64`) from this build
session (x86_64), with a completely different boot mechanism (no
BIOS, no GRUB). Research into how to handle this properly (see below)
ruled out two candidates and landed on a third:

- **CLFS ("Cross Linux From Scratch")**, LFS's historical sibling
  project for exactly this scenario — **dead since 2017**, not usable.
- **LFS's own current "Cross Edition"** (actively maintained by the
  LFS project, at `linuxfromscratch.org/~xry111/lfs/view/clfs-ng-systemd/`) —
  technically excellent and the most "book-authentic" option, but its
  method is "cross-compile a minimal temp system, then **boot it
  directly on the real target hardware and finish the rest of the
  build natively there**" instead of chrooting. That means a large
  fraction of the build (everything past the temporary-tools stage)
  would run on the Pi's own CPU, not in this cloud session — in
  direct tension with the explicit choice already made ("build it
  here, I just boot the result").
- **PiCLFS** (`github.com/LeeKyuHyuk/PiCLFS`) — a Raspberry Pi 4
  port of classic CLFS. Stale (2020, GCC 9.2, no Pi 5 support,
  targets an old LFS release) so not usable directly, but its overall
  *shape* — fully cross-compile everything on an x86_64 build host,
  touch the Pi only once to flash the finished SD card — is exactly
  what matches the chosen build model, and is the same approach
  real embedded-Linux build systems (Buildroot, Yocto) use for ARM
  boards.

**Decision (supersedes nothing prior, is new): build methodology is
full cross-compilation**, PiCLFS-shaped, using the current LFS 12.4
package set already catalogued in `03-sources-and-packages.md` rather
than PiCLFS's outdated pinned versions. Nothing is ever natively
executed on the build host; the Pi is touched exactly once, to flash
and boot the finished image. This is a deliberate trade-off,
documented here rather than silently assumed: it departs further from
the stock LFS book than the original x86_64 plan did. If a more
"book-authentic" build (part of it running natively on the Pi) is ever
preferred instead, that's the LFS Cross Edition approach above, and
would need its own revision of this document.

## 2. Resolved decisions (updates to Document 10)

| ID | Was | Now |
|---|---|---|
| **D1** (architecture) | x86_64 only, no multilib | **`aarch64` (ARM64), target hardware: Raspberry Pi 4 or 5** |
| **D12** (boot path) | Legacy BIOS GRUB vs. UEFI GRUB | **Neither — Raspberry Pi firmware boot (EEPROM + `config.txt`), no GRUB at all** |
| **D18** (test-suite scope) | Every package with a test suite, from Ch. 7 onward | **Deferred for the whole build** — nothing can execute natively on the x86_64 build host under full cross-compilation; test suites become meaningful only after the finished image boots on the real Pi (see §6) |
| **D19** (new — kernel source) | — | **Mainline `kernel.org`** (matches the existing package list's Linux version), not the Raspberry Pi Foundation's downstream fork. Trade-off accepted: camera/GPU-acceleration/some-peripheral support may be incomplete; irrelevant to Phase 1's bare "reach a login prompt" goal. Revisit if a later phase needs those peripherals. |
| **D20** (new — build methodology) | — | **Full cross-compilation throughout** (not just Ch. 5–6) — `--host=aarch64-unknown-linux-gnu --build=x86_64-pc-linux-gnu` style flags apply to every package, including what was Chapter 8. No chroot-and-execute-natively phase exists in this plan. |

## 3. Toolchain target

`LFS_TGT` (Document 04 §4.1) becomes **`aarch64-unknown-linux-gnu`**
instead of the host's own `x86_64-...` triplet — this was already
true for stock LFS's *simulated* cross-compilation in Ch. 5–6 (just
with a renamed vendor field on the same architecture); here it's a
*real* cross-compilation to a genuinely different architecture, so
`--build=x86_64-pc-linux-gnu --host=aarch64-unknown-linux-gnu
--target=aarch64-unknown-linux-gnu` (as appropriate per package) is
used consistently, not just in Ch. 5.

**Everything in `04-toolchain-bootstrap.md` §4.2–4.3 (binutils-first
rule, the libgcc/glibc/libstdc++ bootstrapping order, the
`--with-sysroot` mechanism) still applies unchanged** — that theory is
exactly what makes a real cross-build possible in the first place.
What does **not** carry over: §4.1's chroot-entry steps and anything
in §4.4–4.6 that assumes executing a just-built binary on the build
host to test or use it. Those steps are replaced by continuing to
cross-compile, package by package, through what was Chapter 8
(`05-base-system-build.md`), using the same `--host`/`--build` pattern
for every one of the ~90 packages.

## 4. Package list changes

The package inventory in `03-sources-and-packages.md` mostly still
applies (Glibc, GCC, Binutils, Bash, Coreutils, etc. are all
architecture-agnostic source packages). Concrete deltas:

- **GRUB is dropped entirely** — not needed, not used, don't build it.
- **New: Raspberry Pi firmware/boot files** are not a "package" in the
  LFS sense but a required acquisition: the GPU firmware blobs
  (`start4.elf`/`fixup4.dat`-family files, confirmed still used on Pi
  5 despite the "4" in the name — verify exact current filenames
  against `raspberrypi.com/documentation` at build time, naming has
  shifted historically across Pi generations) and the device-tree
  blob for the specific board (e.g. `bcm2712-rpi-5-b.dtb` for Pi 5).
  These come from Raspberry Pi's own firmware repository
  (`github.com/raspberrypi/firmware` or the `rpi-eeprom`/
  `firmware-nonfree` packaging), not from GNU/kernel.org mirrors.
- **Linux kernel configuration** needs Pi-specific options (device
  tree support, the Pi's SD/MMC host controller, USB controller,
  and BCM2712 SoC support for Pi 5) in addition to the mandatory
  settings already listed in `07-bootable-system.md` §7.2 — exact
  config fragment to be pulled from current upstream kernel
  `arch/arm64/configs/` defconfig for the relevant SoC at
  implementation time, not guessed here.

## 5. Boot mechanism (replaces `07-bootable-system.md`'s GRUB section
for this target)

No GRUB, no `grub-install`, no `/etc/fstab` GRUB-designator concerns.
Raspberry Pi boot (confirmed against official Raspberry Pi
documentation, `raspberrypi.com/documentation/computers/`):

1. On power-up, the Pi 4/5's **on-board EEPROM** bootloader runs first
   (not SD-card-resident `bootcode.bin` — that's only the older,
   pre-2020-firmware-update Pi 4 behavior; Pi 5 is EEPROM-only,
   always). It reads its own EEPROM config (`BOOT_ORDER`) to decide
   where to look for the next stage — default order tries the SD card
   first, which is what Phase 1 needs; no EEPROM reconfiguration
   required for a straightforward SD-card boot.
2. It loads the GPU firmware (`start4.elf` + `fixup4.dat`, or the
   current Pi-5-appropriate equivalents) from the **boot partition** —
   a plain **FAT32** partition, conventionally mounted at
   `/boot/firmware` on modern Pi OS layouts (not bare `/boot`).
3. That firmware reads `config.txt` (hardware/boot options) and
   `cmdline.txt` (kernel command line — this is where `root=...`
   equivalents go, there's no GRUB menu entry to edit instead), then
   loads the device-tree blob for the exact board and the kernel image
   (conventionally `kernel8.img` for a 64-bit kernel) directly — no
   bootloader menu, no intermediate boot sector.

**Partition layout for the SD card** (replaces the GRUB-BIOS-partition
/ `/boot` guidance in `02-host-environment-and-partitioning.md` §2.3
for this target):
- Partition 1: FAT32, boot partition — firmware blobs, `config.txt`,
  `cmdline.txt`, device-tree blob(s), kernel image. Small (a few
  hundred MB is generous).
- Partition 2: ext4, root filesystem — the full cross-built OWN-OS
  userland, same content/structure Document 05 already specifies.

**`/etc/fstab`** (`07-bootable-system.md` §7.1) still gets written,
still needs a root-filesystem entry — just no GRUB-side consistency
concern, since there's no GRUB config to keep in sync with it. The
kernel command line's `root=` value in `cmdline.txt` is this plan's
equivalent of GRUB's `linux ... root=...` line.

## 6. What "boot it and validate" means now

Document 08's reboot/validation steps (`08-finalization-and-post-lfs.md`
§8.3) change in one respect: there is no "exit chroot" step, because
there was never a chroot — the whole build is cross-compiled from
outside. The equivalent checkpoint is: assemble the two-partition SD
card image from the cross-built rootfs + kernel + firmware files,
flash it (the project owner already has a card and a way to write it),
insert it into the real Pi, and power on. First successful boot to a
login prompt is still the Phase 1 finish line (`S9` in
`docs/requirements/07-success-criteria-and-metrics.md`) — just reached
by flashing an SD card instead of rebooting a build machine.

Test suites (D18, deferred per §2 above) become meaningful **after**
this first boot, running natively on the Pi itself, which is the first
point in this entire plan anything actually executes on the target
architecture. This is a real limitation worth being honest about: a
cross-compiled package that silently "builds fine" but has a runtime
bug specific to the target arch will not be caught until this point —
not ever, during Ch. 5-era cross-compilation, in stock LFS either, so
this isn't a new category of risk, just a larger dose of the same one
stock LFS already accepts for its own Ch. 5–6 cross-compiled packages
(`09-testing-validation-and-risk.md` §9.1).

## 7. What stays exactly the same

- The entire package **rationale** (why each package is included) —
  `01-overview-and-methodology.md` §1.5.
- `06-system-configuration.md` in full — SysVinit, udev, bootscripts,
  networking, locale, console, `/etc/inputrc`/`/etc/shells` are all
  architecture-agnostic.
- The risk-register *structure* and backup/checkpoint discipline in
  `09-testing-validation-and-risk.md` (the specific PTY-exhaustion
  risk item is moot — there's no native test execution during the
  build at all now, let alone a host `devpts` issue).
- Every requirement in `docs/requirements/` except the parts explicitly
  called out as architecture-dependent — this is a target/methodology
  change, not a product-purpose change.

## 8. Open follow-up (not yet resolved)

- Exact current Pi 5 firmware filenames and kernel `defconfig`
  fragment — flagged in §4 as "verify at implementation time," not
  guessed here.
- Whether EEPROM `BOOT_ORDER` needs any adjustment at all for this
  specific Pi (depends on its current EEPROM config, unknown until
  checked on the actual device) — default order already tries SD
  first, so likely a non-issue, but worth a quick check
  (`rpi-eeprom-config`) before the first flash attempt if the owner
  has access to boot the Pi into any existing OS first to check.
