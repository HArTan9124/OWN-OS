# 04 — Toolchain Bootstrap (LFS Chapters 4–7)

Source: LFS Chapter 4 (Final Preparations), Part III preliminary material
(Introduction, Toolchain Technical Notes, General Compilation
Instructions), Chapter 5 (Compiling a Cross-Toolchain), Chapter 6 (Cross
Compiling Temporary Tools), Chapter 7 (Entering Chroot and Building
Additional Temporary Tools).

This is the technical heart of the whole project. Get this stage wrong
and the final system (Document 5) can be "subtly broken" in ways that
only surface much later.

## 4.1 Final preparations (Ch. 4) — before touching a compiler

1. **Minimal directory layout**, as `root`:
   ```sh
   mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
   for i in bin lib sbin; do ln -sv usr/$i $LFS/$i; done
   case $(uname -m) in x86_64) mkdir -pv $LFS/lib64 ;; esac
   mkdir -pv $LFS/tools   # cross-toolchain install prefix
   ```
   **Hard rule carried through the entire book:** `$LFS/usr/lib64` must
   **never** exist. LFS deliberately avoids a separate lib64 directory;
   its accidental presence (e.g. from a stray binary package) is known to
   break the system. Check for it periodically.

2. **Add the unprivileged `lfs` build user** — Chapters 5–6 are built as
   this user specifically so a mistake can't damage the host:
   ```sh
   groupadd lfs
   useradd -s /bin/bash -g lfs -m -k /dev/null lfs
   passwd lfs   # only needed if logging in directly as lfs
   chown -v lfs $LFS/{usr{,/*},var,etc,tools}
   case $(uname -m) in x86_64) chown -v lfs $LFS/lib64 ;; esac
   su - lfs
   ```

3. **Set up the `lfs` user's shell environment** — this is the single
   most consequential config file in the whole build. `~/.bash_profile`:
   ```sh
   exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
   ```
   `~/.bashrc`:
   ```sh
   set +h
   umask 022
   LFS=/mnt/lfs
   LC_ALL=POSIX
   LFS_TGT=$(uname -m)-lfs-linux-gnu
   PATH=/usr/bin
   if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
   PATH=$LFS/tools/bin:$PATH
   CONFIG_SITE=$LFS/usr/share/config.site
   export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
   export MAKEFLAGS=-j$(nproc)
   ```
   Why each line matters: `set +h` disables bash's path hashing so newly
   built tools in `$LFS/tools/bin` are found immediately, not a
   previously-hashed host binary; `LC_ALL=POSIX` guarantees locale-free,
   predictable `configure` behavior; `LFS_TGT` is a deliberately
   non-default (vendor field = "lfs") but valid GNU target triplet used
   as `--host`/`--target` everywhere to force the cross/isolated build
   path; putting `$LFS/tools/bin` first in `PATH` ensures the
   cross-compiler is picked up before anything on the host;
   `CONFIG_SITE` is blanked to a path that doesn't exist yet, preventing
   `configure` from silently loading host-distro-specific site defaults.
   **Also check for and move aside `/etc/bash.bashrc`** on the host if
   present — some distros inject environment-polluting content there
   that bypasses the clean-room setup above.
   **Decision needed:** confirm how many parallel build jobs (`-j`) to
   use — default plan is `$(nproc)` (all logical cores); never pass a
   bare `-j` with no number (unbounded parallel jobs = system instability).

4. **About SBUs (Standard Build Units)** — a hardware-independent time
   unit used throughout the book: 1 SBU = however long Binutils pass-1
   (Document 4 §4.3) takes to build on 1 core on this specific machine.
   Every other package's build time is quoted as a multiple of that.
   Useful for estimating total Phase 1 wall-clock time once we've built
   the first package — see Document 9 §9.2 for the time-budgeting plan.

5. **About test suites** — running package test suites is strongly
   recommended for the "big three" (Binutils, GCC, Glibc) since a subtle
   toolchain bug there can be catastrophic later, but is **pointless in
   Chapters 5–6**: those binaries are cross-compiled and literally cannot
   run on the build host. Test suites only become meaningful from Chapter
   7 onward. A common false-failure cause is the host's `devpts`
   filesystem not being set up correctly (PTY exhaustion) — not a real
   bug in the package.

## 4.2 Toolchain technical notes — why any of this isolation is needed

The overall goal of Chapters 5–6 is a temporary, host-independent toolset
built via (faked) **cross-compilation**. Even though the build machine
and the target machine are the same physical computer, cross-compilation
is used anyway because of one property: *anything cross-compiled cannot
accidentally depend on the build/host environment*.

Terminology used throughout: **build** = the machine compiling the code;
**host** = the machine that will *run* the code (confusingly, not the
same sense of "host" used elsewhere in the book for "the existing distro
we're bootstrapping from" — context disambiguates); **target** = the
machine the *compiler itself* produces code for.

LFS's actual staged plan (table from the book, Toolchain Technical
Notes):

| Stage | Build | Host | Target | Action |
|---|---|---|---|---|
| 1 | pc | pc | lfs | Build cross-compiler cc1 using the host's own compiler, on the host |
| 2 | pc | lfs | lfs | Build the LFS-targeted compiler cc-lfs using cc1, still on the host |
| 3 | lfs | lfs | lfs | Rebuild (and test) cc-lfs using cc-lfs, now inside the chroot |

Implementation mechanics worth internalizing because they explain *why*
specific `configure` flags appear in Document 4 §4.3–4.4:

- The **system triplet** (`cpu-vendor-kernel-os`) names a platform;
  `config.guess` (shipped with most package sources) or `gcc
  -dumpmachine` reveal the host's own triplet. LFS deliberately renames
  the vendor field to `lfs` (`$LFS_TGT`) and passes it as `--host` to
  force autoconf-based build systems into "cross-compilation mode" —
  meaning they stop trying to *run* any just-built binary (which might
  not work yet) and only use the prefixed cross-tools
  (`$LFS_TGT-gcc`, `$LFS_TGT-readelf`, etc.) for anything meant to
  execute on the target.
- `--with-sysroot=$LFS` tells the cross-linker/cross-compiler where to
  find target libraries, which (combined with deleting stray libtool
  `.la` archives and patching an outdated bundled libtool) prevents
  anything in Chapter 6 from accidentally linking against the host's
  libraries.
- **Binutils goes first**, always — both GCC's and Glibc's `configure`
  scripts probe the assembler/linker's capabilities to decide which of
  their own features to enable. Get Binutils wrong and the breakage in
  GCC/Glibc can be invisible until much later.
- The **libgcc/glibc/libstdc++ chicken-and-egg problem**: GCC needs libc
  to build its own internal runtime support library (libgcc) fully;
  libstdc++ needs libgcc. LFS solves this by building a deliberately
  *degraded* libgcc first (no threads/exceptions), using it to build the
  real Glibc, then rebuilding gcc pass 2 to produce a fully-functional
  libstdc++ linked against a freshly rebuilt (non-degraded) libgcc.
- **Everything gets rebuilt a third time in Chapter 8**, even packages
  that already "worked" in Chapters 6–7. This is not redundancy for its
  own sake — temporary builds have optional features disabled (missing
  deps, or cross-compilation mode) that the final, permanent build needs
  enabled for a stable, reproducible system.

## 4.3 Chapter 5 — Compiling the cross-toolchain (as user `lfs`)

Installed into `$LFS/tools` (kept separate; discarded at the end of
Chapter 7). Build order matters and is fixed:

1. **Binutils — pass 1**. Dedicated out-of-tree build dir. Key flags:
   `--prefix=$LFS/tools --with-sysroot=$LFS --target=$LFS_TGT
   --disable-nls --enable-gprofng=no --disable-werror
   --enable-new-dtags --enable-default-hash-style=gnu`. (~1 SBU by
   definition; ~678 MB build disk.)
2. **GCC — pass 1**. Degraded (no full libstdc++ yet).
3. **Linux API headers** (sanitized) — lets Glibc talk to the kernel's
   exposed interface, from the Linux 6.16.1 source tree, *not* a full
   kernel build.
4. **Glibc 2.42** — the first package actually cross-compiled
   (`--host=$LFS_TGT --build=$(../scripts/config.guess)`, installed via
   `DESTDIR` into `$LFS`). After this, run the sanity checks the book
   specifies (confirm the installed dynamic linker name matches
   expectations for the architecture, e.g.
   `ld-linux-x86-64.so.2` on x86_64).
5. **Libstdc++ from GCC source** — built against the just-installed
   Glibc.

## 4.4 Chapter 6 — Cross-compiling temporary tools (as user `lfs`)

Still running under the host OS, but now linking only against the
Chapter-5 libraries, installed to their **final** locations (not
`$LFS/tools`) even though they can't be run yet:

M4 → Ncurses → Bash → Coreutils → Diffutils → File → Findutils → Gawk →
Grep → Gzip → Make → Patch → Sed → Tar → Xz → **Binutils pass 2** →
**GCC pass 2** (this GCC pass 2 is the first genuinely *native* LFS
compiler, and is also the last package in the book that is still
cross-compiled — a known, accepted limitation: nothing after GCC pass 2
can be safely cross-compiled with this toolchain, which is fine because
nothing after this point needs to be).

**Safety rule, repeated because it's the one most likely to be
violated under time pressure:** every single command in Chapters 5–6
must run as `lfs`, never `root`. Running as root here risks writing
outside `$LFS` directly into the live build host.

## 4.5 Chapter 7 — Entering chroot (as `root`, then self-contained)

1. **Change ownership** of everything under `$LFS` from `lfs` → `root`,
   so no stray host UID ends up owning files permanently.
2. **Mount virtual kernel filesystems** into `$LFS`:
   ```sh
   mkdir -pv $LFS/{dev,proc,sys,run}
   mount -v --bind /dev $LFS/dev
   mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
   mount -vt proc proc $LFS/proc
   mount -vt sysfs sysfs $LFS/sys
   mount -vt tmpfs tmpfs $LFS/run
   # /dev/shm: bind or mount tmpfs depending on whether host's is a symlink
   ```
3. **Enter chroot:**
   ```sh
   chroot "$LFS" /usr/bin/env -i \
       HOME=/root TERM="$TERM" PS1='(lfs chroot) \u:\w\$ ' \
       PATH=/usr/bin:/usr/sbin \
       MAKEFLAGS="-j$(nproc)" TESTSUITEFLAGS="-j$(nproc)" \
       /bin/bash --login
   ```
   From this moment, `$LFS` is no longer needed/used — `/` *is* the new
   system. `/tools/bin` is deliberately **not** in `PATH` any more (the
   cross-toolchain's job is done). The prompt will read `I have no
   name!` until `/etc/passwd` exists (next step) — expected, not a bug.
4. **Create full FHS directory tree** (`/boot /home /mnt /opt /srv`,
   full `/etc`, `/usr/{local,share,...}`, `/var/...`), with correct
   modes (`/root` 0750, `/tmp` and `/var/tmp` 1777 sticky). Re-confirm
   `/usr/lib64` still does not exist.
5. **Create essential files/symlinks**: `/etc/mtab → /proc/self/mounts`,
   a minimal `/etc/hosts`, and critically the **first** `/etc/passwd`
   and `/etc/group` (root, bin, daemon, messagebus, uuidd, nobody users;
   a fixed, book-defined set of system groups keyed to specific GIDs
   that later udev rules and `/etc/fstab` depend on — notably GID 5 =
   `tty`, used by the `devpts` mount above and in `/etc/fstab` later). A
   temporary `tester` user/group (uid/gid 101) is added here too, used
   by some Chapter 8 test suites, and deleted again at the very end of
   Chapter 8 (Document 5 §5.4).
   Restart the shell (`exec /usr/bin/bash --login`) once these files
   exist — this is what clears the "I have no name!" prompt.
   Initialize `/var/log/{btmp,lastlog,faillog,wtmp}` with correct modes.
6. **Build the last temporary tools, now natively, inside chroot:**
   Gettext → Bison → Perl → Python → Texinfo → Util-linux.
7. **Clean up and optionally back up** (Document 9 §9.3 has the full
   backup/restore procedure — strongly recommended as a checkpoint
   here): remove `/usr/share/{info,man,doc}/*` and stray `.la` files,
   delete `/tools` (its job is done, ~1 GB reclaimed). This is the
   natural "save point" before the much longer Chapter 8 build begins.

## 4.6 Checklist for this document

- [ ] Minimal `$LFS` directory layout created, `/usr/lib64` absent
- [ ] `lfs` user created, owns `$LFS` subtrees
- [ ] `.bash_profile`/`.bashrc` for `lfs` set up exactly as specified; `/etc/bash.bashrc` on host neutralized if present
- [ ] Binutils → GCC(1) → headers → Glibc → libstdc++ built, in that order, under `$LFS/tools`
- [ ] Chapter 6 temporary tools built, in order, as `lfs`, no root usage
- [ ] GCC pass 2 (first native compiler) confirmed working
- [ ] Ownership flipped to root, virtual filesystems mounted, chroot entered
- [ ] FHS tree + essential files created, `/etc/passwd`+`/etc/group` in place
- [ ] Final Chapter 6/7 temporary tools (Gettext..Util-linux) built natively in chroot
- [ ] Cleanup done; backup checkpoint taken before Chapter 8
