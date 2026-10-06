# 02 — Host Environment & Partitioning

Source: LFS Chapter 2 (Preparing the Host System) — Introduction, Host
System Requirements, Building LFS in Stages, Creating a New Partition,
Creating a File System, Setting the `$LFS` Variable and Umask, Mounting
the New Partition.

## 2.1 Host system requirements

The **build host** is any existing Linux install (a prior LFS build,
or a mainstream distro like Debian/Fedora/openSUSE with its "development"
package group installed). It only needs to be good enough to bootstrap
Stage 1 — it is discarded from the dependency chain almost immediately.

**Hardware (recommended):** ≥4 CPU cores, ≥8 GB RAM. Slower machines work,
just take proportionally longer (GCC alone can take minutes on fast
hardware or *days* on very old hardware).

**Software — minimum versions** (LFS 2.2.2; an official `version-check.sh`
script is provided in the book to verify all of these automatically):

| Tool | Min version | Note |
|---|---|---|
| Bash | 3.2 | `/bin/sh` must symlink to bash |
| Binutils | 2.13.1 | upper bound 2.45 (untested beyond) |
| Bison | 2.7 | `/usr/bin/yacc` → bison |
| Coreutils | 8.1 | needed first — `--version-sort` requires it |
| Diffutils | 2.8.1 | |
| Findutils | 4.2.31 | |
| Gawk | 4.0.1 | `/usr/bin/awk` → gawk |
| GCC (+ g++) | 5.4 | upper bound 15.2.0 (untested beyond); C/C++ std libs+headers required |
| Grep | 2.5.1a | |
| Gzip | 1.3.12 | |
| Linux kernel | 5.4 | must support UNIX 98 PTY (`CONFIG_UNIX98_PTYS=y`) |
| M4 | 1.4.10 | |
| Make | 4.0 | |
| Patch | 2.5.4 | |
| Perl | 5.8.8 | |
| Python | 3.4 | |
| Sed | 4.1.5 | |
| Tar | 1.22 | |
| Texinfo | 5.0 | |
| Xz | 5.0.0 | |

Required symlinks: `sh → bash`, `/usr/bin/awk → gawk`,
`/usr/bin/yacc → bison` (or a script wrapping it).

**Action item:** before any build work, run the book's
`version-check.sh` on the actual build host/container and resolve every
`ERROR:` line. This is a hard gate — an incorrectly configured GCC or
Glibc on the host is described by upstream as the single most common
cause of "subtly broken toolchain" failures that surface much later in
the build.

## 2.2 Building in stages (if the host reboots mid-build)

LFS explicitly supports pausing across reboots, with rules per chapter
range:

- **Ch. 1–4** (host-side prep): after any reboot, re-verify `$LFS` is set
  **for the root user specifically** if root-level steps resume.
- **Ch. 5–6**: the `$LFS` partition must be (re-)mounted; *all* work here
  must be done as the unprivileged `lfs` user (`su - lfs`) — doing it as
  root risks damaging the host.
- **Ch. 7–10**: `$LFS` must be mounted; entering chroot requires `$LFS`
  set for root; virtual kernel filesystems (`/dev`, `/proc`, `/sys`,
  `/run`) must be (re-)mounted before re-entering chroot (exact commands
  in Document 4 §4.3).

**Decision needed:** since this build will run inside an ephemeral cloud
container/session (per this repo's environment), decide up front whether
the whole Phase 1 build happens in one continuous session or whether we
need a documented resume procedure + backup/restore checkpoint strategy
(see Document 9 §9.3) baked into the execution plan from day one.

## 2.3 Partitioning

- **Minimum partition size:** ~10 GB (source tarballs + build artifacts).
- **Recommended:** 20–30 GB if this is meant to grow beyond bare LFS later
  (room for BLFS packages, user data, build leftovers).
- **Swap:** reuse the host's swap if present; otherwise size ~2x RAM,
  more only if hibernation (suspend-to-disk) is wanted. Avoid SSD swap if
  avoidable.
- **GRUB BIOS partition:** if the boot disk uses a GPT partition table, a
  small (~1 MB) unformatted "BIOS Boot" partition is required for GRUB's
  installer (code `EF02` in `gdisk`).
- **Optional convenience partitions** (FHS-guided, not required for a
  minimal build): `/boot` (~200 MB, recommended — isolates kernel/bootloader),
  `/boot/efi` (if UEFI), `/home`, `/usr/src`, `/opt`, `/tmp`. A separate
  `/usr` is explicitly **not** recommended for LFS without extra initramfs
  work (out of scope for Phase 1).

**Decision needed:** single root partition vs. separate `/boot` (and
possibly `/home`). Recommendation: single root + no separate `/boot` for
the first Phase 1 build (simplicity), revisit for later iterations.

Tools: `cfdisk` or `fdisk` on the chosen block device (e.g. `/dev/sda`).

## 2.4 Filesystem

LFS assumes the root filesystem is **ext4**:

```sh
mkfs -v -t ext4 /dev/<LFS_PARTITION>
mkswap /dev/<SWAP_PARTITION>   # only if a new swap partition was made
```

Other filesystems (ext2/ext3, XFS, JFS, Btrfs...) are possible since LFS
only needs whatever the Linux kernel supports, but ext4 is the
book-verified default and what this plan assumes for Phase 1.

## 2.5 The `$LFS` variable and umask

Every subsequent command in the entire book is written assuming an
environment variable `LFS` points at the mount point of the target
partition (book convention: `/mnt/lfs`) and `umask` is `022`.

```sh
export LFS=/mnt/lfs
umask 022
```

This must be true **for every user context** used throughout the build
(the initial root shell, the `lfs` build user, and root again when
re-entering after any reboot). The book's recommended durable fix is to
add both lines to `~/.bash_profile` (or `.bashrc`, if logging in via a
graphical/non-login shell) for both the `lfs` user and `root`.

## 2.6 Mounting

```sh
mkdir -pv $LFS
mount -v -t ext4 /dev/<LFS_PARTITION> $LFS
chown root:root $LFS
chmod 755 $LFS
```

Multiple partitions (e.g. separate `/home`) mount as subdirectories under
`$LFS` the same way. **Verify** the mount does **not** carry `nosuid` or
`nodev` (these will break later chroot/package-install steps) — check
with a bare `mount` and remount without those options if present.

If the host might reboot before the LFS partition is permanently needed,
add a line to the **host's** `/etc/fstab` so it remounts automatically,
and `swapon` any dedicated swap partition.

## 2.7 Checklist for this document

- [ ] Confirm target architecture (x86_64 assumed — Document 1 §1.2)
- [ ] Run and pass `version-check.sh` on the actual build host
- [ ] Decide single-session vs. multi-session build (Document 9)
- [ ] Decide partition layout (root-only vs. +`/boot`)
- [ ] Partition, format (ext4), set `$LFS`/`umask`, mount
