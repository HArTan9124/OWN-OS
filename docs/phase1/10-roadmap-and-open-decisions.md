# 10 — Master Roadmap & Open Decisions

This document consolidates every "decision needed" flagged across
Documents 1–9 into one place, plus the milestone checklist for Phase 1
execution. **Nothing in Phase 1 should start until the decisions below
are either answered or explicitly deferred with a reason.**

## 10.1 Consolidated decision log

| # | Decision | Where discussed | Recommendation in this plan | Status |
|---|---|---|---|---|
| D1 | Target architecture | Doc 1 §1.2 | x86_64 only, no multilib | **Needs sign-off** |
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
| D12 | Boot path: legacy BIOS GRUB vs. UEFI GRUB | Doc 7 §7.2/§7.3 | Legacy BIOS for first build | **Needs sign-off** |
| D13 | OS identity in `/etc/os-release` etc. (name, version, codename) | Doc 8 §8.1 | Rebrand to OWN-OS identity, not "Linux From Scratch" | **Needs sign-off** |
| D14 | Post-boot dev workflow (chroot-back-in vs. SSH vs. native console) | Doc 8 §8.4 | Chroot-back-in first, SSH once available | **Needs sign-off** |
| D15 | **OWN-OS's actual purpose** — workstation / server / embedded / other | Doc 8 §8.5 | Not pre-judged by this plan — highest-leverage open question | **Needs sign-off — blocks all of Phase 2** |
| D16 | Ongoing security-advisory monitoring ownership | Doc 8 §8.6 | Assign explicitly, make recurring | **Needs sign-off** |
| D17 | Checkpoint/backup storage location (ephemeral session risk) | Doc 9 §9.3 | Export outside the build container, not just local disk | **Needs sign-off** |
| D18 | Test-suite scope (big-three only vs. every package from Ch. 7 on) | Doc 9 §9.1 | Every package with a test suite, from Ch. 7 onward | **Needs sign-off** |

None of these block *writing* the plan (done, in this folder) — they
block *executing* it. Recommend reviewing this table with the project
owner as a single pass before Phase 1 implementation work begins.

## 10.2 Milestone checklist (maps 1:1 to Documents 2–8)

- [ ] **M0 — Decisions.** D1–D18 above resolved or explicitly deferred.
- [ ] **M1 — Host ready** (Doc 2). Host toolchain verified, partition created, formatted, mounted, `$LFS`/umask environment confirmed.
- [ ] **M2 — Sources staged** (Doc 3). All ~90 packages + 6 patches downloaded and MD5-verified into `$LFS/sources`.
- [ ] **M3 — Cross-toolchain built** (Doc 4 §4.1–4.3). Binutils→GCC(1)→headers→Glibc→libstdc++ built under `$LFS/tools`, as `lfs`.
- [ ] **M4 — Temporary tools built** (Doc 4 §4.4). M4 through GCC pass 2 built, as `lfs`, natively cross-compiled against the Ch.5 toolchain.
- [ ] **M5 — Chroot entered & finalized** (Doc 4 §4.5). Ownership flipped, virtual filesystems mounted, chroot entered, FHS tree + essential files created, final temporary tools (Gettext..Util-linux) built, Ch.7 backup checkpoint exported.
- [ ] **M6 — Base system built** (Doc 5). All ~90 final packages built natively with test suites (per D18), stripping decision applied (per D5), cleanup done, end-of-Ch.8 checkpoint exported.
- [ ] **M7 — System configured** (Doc 6). Init/bootscripts, udev, networking, locale, inputrc/shells all in place per D7–D10.
- [ ] **M8 — System made bootable** (Doc 7). fstab, kernel built with all mandatory config options, GRUB installed and configured, per D11–D12.
- [ ] **M9 — First boot** (Doc 8 §8.1–8.3). Identity files rebranded (D13), pre-reboot checklist done, chroot exited cleanly, system rebooted to a login prompt on its own kernel.
- [ ] **M10 — Phase 1 complete.** Login prompt reached on OWN-OS's own kernel/toolchain/bootloader. Post-boot workflow (D14) operational. Maintenance posture (D16) assigned.

**M10 is the Phase 1 finish line.** Everything past it — workstation vs.
server direction (D15), any package manager, any desktop/display stack,
systemd reconsideration — is explicitly Phase 2+ and deliberately not
planned in this folder.

## 10.3 What is explicitly out of scope for Phase 1

- BLFS (Beyond LFS) packages of any kind — servers, desktop environments,
  browsers, drivers, anything not in the ~90-package list in Document 3.
- Multilib / 32-bit compatibility on a 64-bit build.
- UEFI boot (unless D12 is decided otherwise before execution starts).
- Any package manager (per D6 default).
- Any init system other than SysVinit (per D7 default).
- DHCP / dynamic networking (per D9 default).
- Custom compiler optimization flags.
- Multi-architecture support beyond the single confirmed target (D1).

If execution later proves any of these necessary *within* Phase 1 after
all, that's a scope change significant enough to revisit this document,
not a silent addition.
