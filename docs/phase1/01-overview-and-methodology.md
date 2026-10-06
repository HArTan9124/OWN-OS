# 01 — Overview & Methodology

Source: LFS Preface (Foreword, Audience, Target Architectures,
Prerequisites, Standards, Package Rationale, Structure), Chapter 1
(Introduction), and Part III preliminary material.

## 1.1 What "building our OS" means here

Linux From Scratch is a book-length, hands-on process for building a
complete, usable Linux system **entirely from source code**, using an
existing Linux installation (the "host") only as scaffolding — a compiler,
shell, and basic toolchain to bootstrap a brand-new, independent system on
a separate partition. The end state is a system that:

- boots on real (or virtual) hardware with its own kernel and bootloader,
- is not derived from, and does not depend on, any binary packages from the host or any distro,
- gives us full control and understanding of every installed byte,
- is a deliberately minimal "foundation" — a base to customize into whatever OWN-OS is meant to become (desktop, server, embedded, etc.) in later phases.

This is the right starting point for an "our own OS" project because it
*is* the process of building an OS from scratch — LFS's stated purpose is
exactly: learn how a Linux system works from the inside out, gain full
control, and produce a compact, auditable, customizable system (LFS
Audience, "Foreword").

## 1.2 Target architecture

LFS 12.4 primarily targets **x86_64** (and legacy 32-bit x86). ARM and
PowerPC are supported with modifications but need an existing Linux
install for that architecture as the host (LFS "Target Architectures").

**Decision needed (see Document 10):** confirm OWN-OS Phase 1 targets
x86_64 only. All subsequent documents assume x86_64 unless told otherwise.

A 64-bit "pure" build (64-bit executables only, no multilib 32-bit
compatibility layer) is the default and what these documents plan for —
multilib roughly doubles build complexity (every library built twice) and
is explicitly discouraged by upstream LFS for first builds.

## 1.3 Prerequisites (skills, not just tools)

LFS assumes comfort with: the Unix/Linux command line, file/directory
manipulation, and building software from source in general. It is not a
tutorial on *using* Linux — it is a guide to *constructing* one. Two
pre-reading references the book itself recommends:

- Software-Building-HOWTO — https://tldp.org/HOWTO/Software-Building-HOWTO.html
- Beginner's Guide to Installing from Source — https://moi.vonos.net/linux/beginners-installing-from-source/

## 1.4 Standards compliance

LFS follows, as closely as practical:

- **POSIX.1-2008**
- **Filesystem Hierarchy Standard (FHS) 3.0**
- **Linux Standard Base (LSB) 5.0** — partially; LFS alone does **not**
  achieve full LSB certification (that needs many BLFS packages too). LFS
  supplies the LSB-Core packages (Bash, Binutils, Coreutils, Gawk, GCC,
  Glibc, Grep, Sed, Shadow, SysVinit, Tar, Util-linux, Zlib, etc.) and the
  one LSB-Languages package (Perl). Desktop/Imaging/Graphics LSB
  categories are entirely BLFS's responsibility.

This matters for OWN-OS because it tells us the resulting base system is
*standards-shaped* (predictable paths, predictable init behavior) even
though it is hand-built — useful if we ever want third-party software to
"just work" on it later.

## 1.5 Why every package is there

The book justifies each of the ~90 base packages individually (full table
in Document 3/5). The short version: everything in LFS is either (a) a
build-time dependency needed to compile the system (e.g. Binutils, GCC,
Make, M4, Perl, Python, Autoconf/Automake, Bison, Flex, Texinfo), (b) a
core runtime/userland piece required for a minimally usable, bootable,
networkable Linux system (Glibc, Bash, Coreutils, Util-linux, Shadow,
SysVinit, Udev, Grub, the Linux kernel itself), or (c) a library a
higher-level piece needs (Zlib, OpenSSL, Readline, Ncurses, GMP/MPFR/MPC
for GCC, etc.). Nothing is there for convenience alone.

## 1.6 The four-stage build model (the core thing to understand)

This is the single most important structural fact about the whole project
— everything else in this plan is detail under one of these four stages:

1. **Stage 0 — Host prep** (Document 2). Verify/augment the host's own
   toolchain versions, partition and format the target disk, set up the
   working environment (`$LFS`, `umask`, the `lfs` build user).

2. **Stage 1 — Cross-toolchain** (Document 4, LFS Ch. 5). Build a
   self-contained cross-compiler (Binutils, GCC pass 1, sanitized Linux
   API headers, Glibc, libstdc++) installed under `$LFS/tools`, built
   *for* the target but still *on/with* the host. This is "faked"
   cross-compilation (build machine = target machine) purely to guarantee
   nothing links against the host's libraries.

3. **Stage 2 — Temporary tools** (Document 4, LFS Ch. 6). Using the
   Stage-1 cross-compiler, cross-compile ~17 more packages (M4, Ncurses,
   Bash, Coreutils, Diffutils, File, Findutils, Gawk, Grep, Gzip, Make,
   Patch, Sed, Tar, Xz, then Binutils pass 2 and GCC pass 2 — this second
   GCC pass is the first **native** LFS compiler). Still running under
   the host OS, but no longer touching host libraries.

4. **Stage 3 — Chroot + final system** (Documents 4/5/6/7, LFS Ch. 7–10).
   `chroot` into `$LFS`, which from this point on is a fully isolated
   environment (host kernel only, nothing else shared). Build the last
   handful of temporary tools needed to be self-hosting (Gettext, Bison,
   Perl, Python, Texinfo, Util-linux), then rebuild **every** package a
   second and final time, natively, permanently, with test suites run.
   Configure the system, build/install the kernel, install GRUB.

The reason for all this isolation (explained in full in Document 4) is
simple: anything cross-compiled *cannot* accidentally depend on the host.
Skipping stages or shortcuts here is the single biggest source of a
"subtly broken" final system — failures that don't show up until near the
very end of the build.

## 1.7 How this maps to OWN-OS's own phase plan

- **Phase 1 (this folder)** = LFS Chapters 1–11 end-to-end: a minimal,
  standards-compliant, bootable, self-hosting base Linux system with
  networking, a shell, and a package build toolchain — nothing more.
- **Phase 2+ (future, not planned here)** = BLFS territory: decide
  workstation vs. server vs. something else, add a display stack / init
  system upgrade / package manager / OWN-OS-specific branding and tooling.
  LFS Chapter 11.5 ("Getting Started After LFS") is the handoff point —
  see Document 8.

Phase 1 is deliberately scoped to *just* reaching a login prompt on
OWN-OS's own kernel, built by OWN-OS's own toolchain, on OWN-OS's own
partition. Nothing else is in scope until that milestone is real.
