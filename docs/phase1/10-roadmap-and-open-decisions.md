# 10 — Master Roadmap & Open Decisions

This document consolidates every "decision needed" flagged across
Documents 1–9 into one place, plus the milestone checklist for Phase 1
execution. **Nothing in Phase 1 should start until the decisions below
are either answered or explicitly deferred with a reason.**

## 10.1 Consolidated decision log

| # | Decision | Where discussed | Recommendation in this plan | Status |
|---|---|---|---|---|
| D1 | Target architecture | Doc 1 §1.2, **Doc 11** | ~~x86_64 only~~ **RESOLVED 2026-10-06: `aarch64`, Raspberry Pi 4 or 5. Build happens via full cross-compilation in this cloud session (x86_64); the Pi is only touched to flash/boot the finished image. See Document 11.** | **Resolved** |
| D2 | Single continuous build session vs. documented multi-session resume | Doc 2 §2.2 | **RESOLVED 2026-10-06: proceed continuously; checkpoint periodically (see D17) so a session loss costs minimal redo.** | Resolved (default applied) |
| D3 | Image size (was: partition layout) | Doc 2 §2.3, Doc 11 §1 | **RESOLVED 2026-10-06: ~6 GB root-partition image + ~256 MB FAT32 boot partition, assembled on the 32 GB SD card confirmed by the owner. Well under the card's capacity (final system is ~3 GB per the book); root partition is resizable to use the full card later if wanted, not required for Phase 1.** | Resolved (default applied) |
| D4 | Parallel build job count (`-j`) | Doc 4 §4.1 | **RESOLVED: `$(nproc)` — this build session has 4 cores.** | Resolved (default applied) |
| D5 | Debug-symbol stripping | Doc 5 §5.4 | **RESOLVED: skip for first build (keep debuggability).** | Resolved (default applied) |
| D6 | Package management strategy | Doc 5 §5.3 | **RESOLVED: none for Phase 1; revisit explicitly in Phase 2.** | Resolved (default applied) |
| D7 | Init system: SysVinit vs. systemd | Doc 6 §6.1 | **RESOLVED: SysVinit (matches base LFS; systemd is a Phase 2+ undertaking).** | Resolved (default applied) |
| D8 | Network interface naming: persistent (`enp5s0`-style) vs. legacy `ethN` | Doc 6 §6.3 | **RESOLVED: keep persistent naming.** | Resolved (default applied) |
| D9 | Networking: static IP vs. DHCP | Doc 6 §6.4 | **RESOLVED: static IP for Phase 1** (DHCP is a BLFS addition). First boot is validated via HDMI+keyboard console, not SSH, specifically so a networking misconfiguration doesn't block seeing the login prompt (S9). | Resolved (default applied) |
| D10 | Default system locale | Doc 6 §6.6 | **RESOLVED: `C.UTF-8`.** | Resolved (default applied) |
| D11 | `/etc/fstab` + boot config: device-path vs. UUID/PARTUUID identification | Doc 7 §7.1, Doc 11 §5 | **RESOLVED: PARTUUID/UUID** — robust against the SD card ever being read as a different device node (e.g. `mmcblk0` vs. a USB-SD adapter's `sda`). | Resolved (default applied) |
| D12 | Boot path: legacy BIOS GRUB vs. UEFI GRUB | Doc 7 §7.2/§7.3, **Doc 11 §5** | ~~Legacy BIOS~~ **RESOLVED 2026-10-06: neither — no GRUB at all. Raspberry Pi firmware boot (EEPROM + `config.txt` + device tree), per Document 11 §5.** | **Resolved** |
| D13 | OS identity in `/etc/os-release` etc. (name, version, codename) | Doc 8 §8.1 | **RESOLVED 2026-10-06 by the project owner: name is "Ravya OS".** `NAME="Ravya OS"`, `ID=ravya`, `PRETTY_NAME="Ravya OS 1.0"` (version/codename scheme to be finalized at Document 08 execution time, but the name itself is locked). | **Resolved** |
| D14 | Post-boot dev workflow | Doc 8 §8.4, Doc 10 §10.2 M9 | **RESOLVED: HDMI+keyboard console for first boot/validation; SSH once static networking is confirmed working.** No chroot-back-in option exists under the cross-build methodology (D20). | Resolved (default applied) |
| D15 | **OWN-OS's actual purpose** — workstation / server / embedded / other | Doc 8 §8.5 | Still not pre-judged — highest-leverage open question, but does not block Phase 1 technical execution | **Needs sign-off — blocks all of Phase 2, not Phase 1** |
| D16 | Ongoing security-advisory monitoring ownership | Doc 8 §8.6 | **RESOLVED: project owner (solo, per Document 06 A1), recurring — mechanism (e.g. a scheduled check-in) to be set up at Stage 9.** | Resolved (default applied) |
| D17 | Checkpoint/backup storage location (ephemeral session risk) | Doc 9 §9.3 | **RESOLVED: periodic compressed checkpoints, sent to the project owner as downloadable files from this session at natural break points (not just kept on the container's local disk), so a session loss doesn't mean a full restart.** | Resolved (default applied) |
| D18 | Test-suite scope (big-three only vs. every package from Ch. 7 on) | Doc 9 §9.1, **Doc 11 §6** | ~~Every package from Ch. 7 onward~~ **RESOLVED 2026-10-06: deferred for the entire build — full cross-compilation (D20) means nothing executes natively on the build host at all. Test suites become meaningful only after first boot on the real Pi. See Document 11 §6.** | **Resolved (deferred)** |
| D19 | Raspberry Pi kernel source: mainline `kernel.org` vs. Raspberry Pi Foundation's downstream fork | **Doc 11 §4** | **RESOLVED 2026-10-06: mainline `kernel.org`, consistent with the existing package list. Accepted trade-off: incomplete camera/GPU-acceleration/some-peripheral support, irrelevant to Phase 1's login-prompt goal.** | **Resolved** |
| D20 | Build methodology for a cross-architecture target: LFS's own "Cross Edition" (boot temp system on real Pi, finish natively there) vs. full cross-compilation throughout (PiCLFS-shaped) | **Doc 11 §1** | **RESOLVED 2026-10-06: full cross-compilation throughout, to match the already-chosen "build it here, you boot the result" model. Explicit trade-off accepted: departs further from the stock LFS book than the original x86_64 plan.** | **Resolved** |

None of these block *writing* the plan (done, in this folder) — they
block *executing* it. Recommend reviewing this table with the project
owner as a single pass before Phase 1 implementation work begins.

## 10.2 Milestone checklist (maps 1:1 to Documents 2–8)

**Updated for the `aarch64`/Raspberry Pi target (Document 11) — M4/M5/M6/M8/M9
below now read differently than the original x86_64-chroot version; see
Document 11 for why.**

- [x] **M0 — Decisions.** D1–D20 above resolved or explicitly deferred. ✅ 2026-10-06.
- [ ] **M1 — Host ready** (Doc 2, adapted by Doc 11 §4). Build-session toolchain verified, a loop-mounted disk image (standing in for a real partition, since this build runs in a cloud session) created and formatted, `$LFS`/umask environment confirmed.
- [ ] **M2 — Sources staged** (Doc 3, + Doc 11 §4 firmware/device-tree acquisition). All ~90 packages (minus GRUB, per Doc 11 §4) + 6 patches + Raspberry Pi firmware/device-tree files downloaded and verified.
- [ ] **M3 — Cross-toolchain built** (Doc 4 §4.1–4.3, target triplet per Doc 11 §3). Binutils→GCC(1)→headers→Glibc→libstdc++ built targeting `aarch64-unknown-linux-gnu`.
- [ ] **M4 — Full system cross-built** (Doc 11 §1/§3, replaces the original M4–M6). Every package from what was Chapter 6 *and* Chapter 8 cross-compiled with `--host=aarch64-unknown-linux-gnu --build=x86_64-pc-linux-gnu` — no chroot phase, no native execution on the build host at all.
- [ ] **M5 — System configured** (Doc 6, unaffected by the arch change). Init/bootscripts, udev, networking, locale, inputrc/shells all in place per D7–D10, applied to the cross-built rootfs tree directly (no chroot needed to write config files).
- [ ] **M6 — Image assembled** (Doc 11 §5). Kernel built for `arm64`/Pi per D19, FAT32 boot partition (firmware blobs, `config.txt`, `cmdline.txt`, device tree, kernel image) + ext4 root partition assembled into one SD-card image.
- [ ] **M7 — Identity & final review** (Doc 8 §8.1–8.2, no chroot-exit step per Doc 11 §6). Identity files rebranded (D13), root password set, final config review done directly on the assembled rootfs.
- [ ] **M8 — First boot** (Doc 11 §6). SD card flashed, inserted into the real Pi, powered on, login prompt reached — the Phase 1 finish line.
- [ ] **M9 — Phase 1 complete.** Login prompt reached on OWN-OS's own kernel/cross-built toolchain, on real Raspberry Pi hardware. Post-boot workflow (D14, now meaning SSH or a serial/HDMI console on the Pi itself — there's no "chroot back in" option without an x86_64 chroot to return to) operational. Maintenance posture (D16) assigned.

**M9 is the Phase 1 finish line.** Everything past it — workstation vs.
server direction (D15), any package manager, any desktop/display stack,
systemd reconsideration — is explicitly Phase 2+ and deliberately not
planned in this folder.

## 10.3 What is explicitly out of scope for Phase 1

- BLFS (Beyond LFS) packages of any kind — servers, desktop environments,
  browsers, drivers, anything not in the ~90-package list in Document 3
  (minus GRUB, per Document 11 §4).
- Multilib / 32-bit ARM compatibility (`armhf`) on the 64-bit (`aarch64`) build.
- GRUB, BIOS boot, UEFI boot — none apply to this target (D12, Document 11 §5).
- Any package manager (per D6 default).
- Any init system other than SysVinit (per D7 default).
- DHCP / dynamic networking (per D9 default).
- Custom compiler optimization flags.
- Any architecture other than `aarch64`/Raspberry Pi 4/5 (D1).
- Raspberry Pi peripherals needing the vendor kernel fork — camera, GPU
  acceleration, etc. (accepted trade-off of D19).
- Native test-suite execution during the build (accepted trade-off of D20) —
  deferred to post-boot, on the real Pi.

If execution later proves any of these necessary *within* Phase 1 after
all, that's a scope change significant enough to revisit this document,
not a silent addition.
