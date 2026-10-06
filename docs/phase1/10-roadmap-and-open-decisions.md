# 10 — Master Roadmap & Open Decisions

This document consolidates every "decision needed" flagged across
Documents 1–9 into one place, plus the milestone checklist for Phase 1
execution. **Nothing in Phase 1 should start until the decisions below
are either answered or explicitly deferred with a reason.**

## 10.1 Consolidated decision log

| # | Decision | Where discussed | Recommendation in this plan | Status |
|---|---|---|---|---|
| D1 | Target architecture | Doc 1 §1.2, **Doc 11** | ~~x86_64 only~~ **RESOLVED 2026-10-06: `aarch64`, Raspberry Pi 4 or 5. Build happens via full cross-compilation in this cloud session (x86_64); the Pi is only touched to flash/boot the finished image. See Document 11.** | **Resolved** |
| D2 | Single continuous build session vs. documented multi-session resume | Doc 2 §2.2 | Decide based on actual available session length | **Needs sign-off** |
| D3 | Partition layout: root-only vs. +`/boot` (+`/boot/efi` if UEFI) | Doc 2 §2.3 | Root-only for first build | **Needs sign-off** |
| D4 | Parallel build job count (`-j`) | Doc 4 §4.1 | `$(nproc)` | **Needs sign-off** |
| D5 | Debug-symbol stripping | Doc 5 §5.4 | Skip for first build (keep debuggability) | **Needs sign-off** |
| D6 | Package management strategy | Doc 5 §5.3 | None for Phase 1; revisit explicitly in Phase 2 | **Needs sign-off** |
| D7 | Init system: SysVinit vs. systemd | Doc 6 §6.1 | SysVinit (matches base LFS; systemd is a Phase 2+ undertaking) | **Needs sign-off** |
| D8 | Network interface naming: persistent (`enp5s0`-style) vs. legacy `ethN` | Doc 6 §6.3 | Keep persistent naming | **Needs sign-off** |
| D9 | Networking: static IP vs. DHCP | Doc 6 §6.4 | Static IP for Phase 1 | **Needs sign-off** |
| D10 | Default system locale | Doc 6 §6.6 | `C.UTF-8` | **Needs sign-off** |
| D11 | `/etc/fstab` + GRUB: device-path vs. UUID/PARTUUID identification | Doc 7 §7.1/§7.3 | Decide together, consistently; UUID if disk layout may change | **Needs sign-off** |
| D12 | Boot path: legacy BIOS GRUB vs. UEFI GRUB | Doc 7 §7.2/§7.3, **Doc 11 §5** | ~~Legacy BIOS~~ **RESOLVED 2026-10-06: neither — no GRUB at all. Raspberry Pi firmware boot (EEPROM + `config.txt` + device tree), per Document 11 §5.** | **Resolved** |
| D13 | OS identity in `/etc/os-release` etc. (name, version, codename) | Doc 8 §8.1 | Rebrand to OWN-OS identity, not "Linux From Scratch" | **Needs sign-off** |
| D14 | Post-boot dev workflow (chroot-back-in vs. SSH vs. native console) | Doc 8 §8.4 | Chroot-back-in first, SSH once available | **Needs sign-off** |
| D15 | **OWN-OS's actual purpose** — workstation / server / embedded / other | Doc 8 §8.5 | Not pre-judged by this plan — highest-leverage open question | **Needs sign-off — blocks all of Phase 2** |
| D16 | Ongoing security-advisory monitoring ownership | Doc 8 §8.6 | Assign explicitly, make recurring | **Needs sign-off** |
| D17 | Checkpoint/backup storage location (ephemeral session risk) | Doc 9 §9.3 | Export outside the build container, not just local disk | **Needs sign-off** |
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
