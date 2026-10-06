# Phase 1 — Building Our OS from Scratch (LFS-based)

This folder is the complete planning package for Phase 1 of the OWN-OS
project: **building a working, bootable, custom Linux system from source**,
following the Linux From Scratch (LFS) methodology
(https://www.linuxfromscratch.org/lfs/view/stable/, LFS 12.4, published
2025-09-01) as the technical foundation.

No build work starts until these documents are reviewed. They translate the
entire LFS stable book into an actionable, OWN-OS-specific plan: what we
build, in what order, on what host, with which decisions explicitly called
out for sign-off before we touch a compiler.

**Ready to actually build?** Start at
[00-execution-plan.md](./00-execution-plan.md) instead of reading
top to bottom — it's the single sequenced, checkbox-driven plan that
combines this folder with `docs/requirements/` (gates, requirement IDs,
success criteria) into 10 ordered stages. Documents `01`–`10` below are
its reference material, not a separate path to follow.

**⚠ Target is now Raspberry Pi 4/5 (`aarch64`), not x86_64** — decided
2026-10-06. Read
[11-arm-raspberry-pi-adaptation.md](./11-arm-raspberry-pi-adaptation.md)
first; it states exactly what that changes in Documents 01–10 (mainly:
no GRUB, no chroot phase, full cross-compilation throughout).

## Reading order

| # | Document | Covers |
|---|----------|--------|
| 0 | [00-execution-plan.md](./00-execution-plan.md) | **The actionable plan** — 10 sequential stages combining this folder with `docs/requirements/`, with gates, task checklists, and acceptance criteria |
| 1 | [01-overview-and-methodology.md](./01-overview-and-methodology.md) | Why LFS, what "from scratch" means, the 4-stage build model, how this maps to OWN-OS phases |
| 2 | [02-host-environment-and-partitioning.md](./02-host-environment-and-partitioning.md) | Host system requirements, disk partitioning, filesystem, `$LFS`/umask, staged-build rules |
| 3 | [03-sources-and-packages.md](./03-sources-and-packages.md) | Full package & patch inventory, versions, acquisition/verification plan |
| 4 | [04-toolchain-bootstrap.md](./04-toolchain-bootstrap.md) | Cross-toolchain theory, Chapters 5–6 (temporary toolchain + tools), Chapter 7 (chroot) |
| 5 | [05-base-system-build.md](./05-base-system-build.md) | Chapter 8: the ~90-package final system build, library/static-lib policy, package management decision |
| 6 | [06-system-configuration.md](./06-system-configuration.md) | Chapter 9: bootscripts, udev/device management, networking, locale, console |
| 7 | [07-bootable-system.md](./07-bootable-system.md) | Chapter 10: fstab, kernel configuration/build, GRUB |
| 8 | [08-finalization-and-post-lfs.md](./08-finalization-and-post-lfs.md) | Chapter 11: first boot, sanity checks, BLFS roadmap (workstation vs. server) |
| 11 | [11-arm-raspberry-pi-adaptation.md](./11-arm-raspberry-pi-adaptation.md) | **Read this one too** — the `aarch64`/Raspberry Pi 4/5 target change: cross-build methodology, toolchain triplet, dropped GRUB, Pi firmware boot, kernel source choice |
| 9 | [09-testing-validation-and-risk.md](./09-testing-validation-and-risk.md) | Test-suite policy, SBU time budgeting, backup/restore checkpoints, troubleshooting, risk register |
| 10 | [10-roadmap-and-open-decisions.md](./10-roadmap-and-open-decisions.md) | Master milestone checklist + the decisions only the project owner can make before Phase 1 execution starts |

## Scope note on sourcing

Every narrative/architectural page of the LFS stable book (Preface, Parts
I–V introductions, all of Chapters 1–4, the toolchain technical notes and
general compilation instructions, all of Chapters 7, 9, 10, 11, and the
package-management/stripping/cleanup sections of Chapter 8) was read in
full and is reflected below. Chapters 5, 6, and the ~90 individual
per-package pages in Chapter 8 are one-page-per-package build recipes
(`./configure && make && make install` plus package-specific flags); those
are catalogued here by package/version/purpose/order (Document 3 and 5)
rather than transcribed verbatim — the exact flags will be pulled from the
book page-by-page during actual implementation in Phase 2, package by
package, since reproducing ~130 near-identical recipe pages adds no
planning value and these pages change with every LFS point release.

## Key takeaway up front

LFS is **not** a distro installer — it is a fully manual, source-compiled
build process in four structural stages:

1. **Host prep** — verify/supplement the host Linux system's toolchain, partition and mount the target disk.
2. **Cross-toolchain + temporary tools** (Ch. 5–6) — build an isolated, host-independent compiler/libc/binutils under `$LFS/tools`, then cross-compile a minimal temporary userland with it.
3. **Chroot + final temporary tools** (Ch. 7) — enter `chroot $LFS`, build the last few tools needed to be fully self-hosted.
4. **Final system build** (Ch. 8–10) — rebuild ~90 packages natively and permanently, configure the system, build the kernel, install GRUB.

Everything after a successful boot (Ch. 11 and beyond) is BLFS
("Beyond LFS") territory — a separate, much larger catalogue of optional
packages (desktop environments, servers, drivers) that is explicitly
**out of scope for Phase 1** and belongs to a later phase.
