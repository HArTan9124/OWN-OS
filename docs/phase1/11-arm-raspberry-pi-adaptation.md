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

### 3.1 Two real bugs found and fixed during execution (2026-10-06)

Both discovered building GCC pass 1 / libstdc++ in this session — recorded
here because they'll recur if this plan is ever re-executed from a clean
checkout, and the original draft of this document got both wrong.

**Bug 1 — the `--with-gxx-include-dir` sysroot trick doesn't apply to our
layout.** `04-toolchain-bootstrap.md`'s GCC-pass-1 flags (inherited
from the book) include `--with-gxx-include-dir=$LFS_TGT.../include/c++/...`
for the later libstdc++ build, relying on a book-specific fact: stock
LFS's `$LFS/tools` is *nested inside* `$LFS` itself, so DESTDIR-prefixing
a sysroot-relative path lands in the right place. Our `$OWNOS_TOOLS`
(`/build/tools`) and `$OWNOS_ROOT` (`/build/root`) are **siblings, not
nested** — so the same flag, installed via `DESTDIR=$OWNOS_ROOT`,
produced a nonsense double-prefixed path
(`/build/root/build/tools/.../include/c++/15.2.0`) instead of where the
compiler actually looks. Confirmed by checking the *compiler's own*
header search path (`$TGT-g++ -v -c`, not the book's assumptions) —
GCC silently omits nonexistent search directories from that output,
which is what made this discoverable: once the headers existed at the
*right* path, they showed up. **Fix applied:** install libstdc++'s
headers directly under `$OWNOS_TOOLS` (no `DESTDIR` sysroot-prefixing
for that specific path), not under `$OWNOS_ROOT`.

**Bug 2 — aarch64 has its own `lib64`-by-default, just like x86_64
does.** §4.1 of the main toolchain document carries the book's
x86_64-only `sed` patch to `gcc/config/i386/t-linux64` (forcing 64-bit
libs into `lib` instead of `lib64`), and this document originally said
that patch was "irrelevant since our target is aarch64, not x86_64."
**That reasoning was wrong.** GCC's aarch64 backend has its own,
exactly analogous file —
`gcc/config/aarch64/t-aarch64-linux` — setting
`MULTILIB_OSDIRNAMES = mabi.lp64=../lib64...`. Left unpatched, it
silently splits the final system across both `/usr/lib` (Glibc, which
has its own explicit `libc_cv_slibdir=/usr/lib` override) and
`/usr/lib64` (everything GCC itself places, starting with libstdc++) —
exactly the split-library-directory mess
`04-toolchain-bootstrap.md` §4.1 names as a hard rule to avoid.
**Fix applied:** `sed -e '/mabi.lp64=/s@../lib64@../lib@' -i gcc/config/aarch64/t-aarch64-linux`,
done *before* GCC pass 1 is configured. This value gets compiled into
the cross-compiler at **its own** configure/build time, not read fresh
by every later package that uses it — discovered the hard way, after
patching the source too late and having to reconfigure and rebuild
GCC pass 1 a second time to pick it up. **Any clean re-execution of
this plan must apply this patch before GCC pass 1 is first configured,
immediately after extracting the GCC source** (same point in the
sequence as the book's own x86_64 `t-linux64` patch) — not as an
afterthought once a problem shows up downstream.

### 3.2 Methodology simplification: Chapter 6 ("temporary tools") is skipped entirely

Realized while starting what would have been the Chapter-6-equivalent
pass (M4, Ncurses, ...): **stock LFS's Chapter 6 exists for exactly one
reason — to bootstrap just enough of a cross-compiled toolchain to
`chroot` into, since the host's own tools can't be trusted/used inside
that isolated environment.** Our methodology (D20) never chroots at
all; every package is cross-compiled directly from this x86_64 session,
using *this session's own host tools* (bash, make, sed, tar, gawk —
already verified present in Stage 1) to drive the cross-compilation
process, exactly like every command in this whole log already does.
There is no point in the build where a "temporary, bootstrap-only"
version of bash/coreutils/make/sed/etc. is ever actually needed — we
already have a perfectly good bash/make/sed driving the build, it's
just the host's own x86_64 copy, which is completely fine since it
never has to run *on* the target.

**Consequence: for every package the book lists in both Chapter 6 and
Chapter 8 (Binutils, GCC, Ncurses, Sed, Gettext, Bison, Grep, Bash, M4,
File, Findutils, Gawk, Gzip, Make, Patch, Tar, Xz, Util-linux, Perl,
Texinfo — the list in `03-sources-and-packages.md` §3.2's closing
note), build it exactly once, using Chapter 8's (fuller-featured,
final) configure flags, not Chapter 6's (deliberately limited) ones.**
Skip Chapter 6's separate, simpler builds of these packages entirely —
they would just get overwritten by the Chapter 8 pass anyway, in stock
LFS too. This also drops **Binutils pass 2** and **GCC pass 2**
specifically: those exist in the book purely to produce a *native*
(not cross) compiler for use inside the chroot — meaningless here,
since GCC pass 1 (already built, Document 11 §3) remains the only
compiler this entire project ever uses, for every remaining package.

Already-built-the-Chapter-6-way packages (M4, in this session, before
this was realized) don't need redoing — Chapter 8's M4 page uses the
same minimal flags, so nothing was wasted. **Document 00's Stage 4
section should be read as "build the Chapter 8 package list, once,
cross-compiled, in dependency order" — not "Chapter 6 then Chapter 8."**

### 3.3 Confirmed risk: loop mounts do not survive a container restart

Discovered the hard way mid-Stage-4: this session's underlying
container restarted at some point (evidenced by `/dev/loop0`'s device
node timestamp jumping forward with no process-visible cause), which
silently dropped the loop-device attachment and the mount at
`$OWNOS_ROOT` — `/build/root` was briefly an **empty directory** with
no indication anything was wrong short of checking `mountpoint -q` or
noticing files that should exist didn't. **The backing image file
itself (`/build/root.img`) was untouched and fully recoverable** — only
the in-kernel loop attachment and mount were lost, not the data.

**Mitigations now in place:**
- `/build/env.sh` (sourced at the start of every build command) now
  checks `mountpoint -q /build/root` and automatically re-attaches a
  loop device and remounts if needed, *before* exporting any build
  variables — so this failure mode self-heals on the next command
  rather than silently building into a phantom empty directory.
- A compressed checkpoint of `/build/root.img` + `/build/tools` +
  `/build/sources` is taken immediately after this was discovered
  (ties to `docs/requirements` D17's "periodic checkpoints, exported
  outside the container" resolution) — this risk is exactly what that
  decision was hedging against, and it was right to.
- **Any resumed session must run `source /build/env.sh` before any
  other build command** — if the mount silently dropped between
  sessions, the self-healing check only runs when this file is
  sourced, not automatically.

### 3.4 Non-autoconf / self-hosting packages: three more cross-compile patterns found (2026-10-06)

Most Chapter 8 packages are plain autoconf (`./configure --host=$OWNOS_TGT
--build=...`) and need nothing beyond that flag. A few packages have their
own build systems or run code they just built as part of building — those
needed individual investigation. Found and resolved while building
Bzip2/Flex/File/Bc:

- **Bzip2** — hand-written `Makefile`/`Makefile-libbz2_so`, no autoconf at
  all. `CC=/AR=/RANLIB=` passed as `make` command-line overrides (these
  beat the Makefile's own `CC=gcc` line). The bigger trap: `make all`'s
  default target chain ends in `test`, which runs the freshly built
  (aarch64) `bzip2` binary directly — fails with `Exec format error` on
  this x86_64 host. Fixed by building the specific named targets
  (`libbz2.a bzip2 bzip2recover`) instead of `all`, never invoking `test`.
  Same class of trap likely recurs for any package whose default `all`
  target is wired to its own test suite rather than `make check` being a
  separate opt-in target — check before assuming `make` alone is safe.
- **Flex** — plain autoconf, but cross-compiling makes `AC_FUNC_MALLOC`/
  `AC_FUNC_REALLOC` default their cache variables
  (`ac_cv_func_malloc_0_nonnull`, `ac_cv_func_realloc_0_nonnull`) to `no`
  (can't run a test program when cross-compiling, so autoconf assumes the
  worst). That pulls in gnulib's `malloc.c`/`realloc.c` replacement shims,
  which are written in obsolete K&R C (`void *malloc ();`) and fail to
  compile against glibc 2.42's prototyped `<stdlib.h>` under GCC 15. Fixed
  by exporting `ac_cv_func_malloc_0_nonnull=yes
  ac_cv_func_realloc_0_nonnull=yes` before `configure` (glibc's malloc/
  realloc are POSIX-conformant, so this is correct, not a hack). **This
  same cache-variable trick is the general fix any time a cross-compiled
  autoconf package pulls in a gnulib malloc/realloc replacement
  unexpectedly** — worth checking for on later packages (Coreutils, Sed,
  Grep, Bash, Gettext, Tar, etc. all use gnulib and could hit this).
  Separately: flex is also used by other packages' build systems as a
  *build-time* code generator (invoked on this x86_64 host while cross-
  compiling something else). The aarch64 flex just built into
  `$OWNOS_ROOT` can't run here, so a native x86_64 flex was installed via
  `apt-get install flex` onto the host's own `PATH`, kept entirely
  separate from the target flex in `$OWNOS_ROOT/usr/bin`.
- **File** — the book's recipe is pure native and doesn't work unmodified:
  File's own build compiles `magic.mgc` by *running* the `file` binary it
  just built against the Magdir source fragments. File's own
  `magic/Makefile.am` already anticipates this (`IS_CROSS_COMPILE`
  conditional): when cross-compiling, it looks for a `file` binary on
  `PATH` and refuses to proceed unless `file --version` reports the exact
  same version being built. Fixed by: (1) extracting a **second copy** of
  the File source, (2) configuring and building it natively (no
  `--host`), (3) `make install`-ing it to `/usr/local` on the build host
  (a bare copy of the libtool wrapper script is not enough — it execs a
  sibling `.libs/file` that doesn't exist outside the build tree, so it
  must be properly installed), (4) then re-running `configure`/`make` for
  the real aarch64-target copy with the native `file` now resolvable on
  `PATH`. This two-copy "build a native helper first" pattern is File's
  own documented expectation, not a workaround — expect to repeat it for
  any later package whose build process needs to execute its own output
  (Groff and Perl are candidates; watch for similar `IS_CROSS_COMPILE`-
  style guards or test-running build steps as each is reached).
- **Bc** — has first-class `HOSTCC`/`CC` separation built into its own
  `configure.sh` (not autoconf) specifically for this situation:
  `gen/strgen.c` (a small code generator that turns `gen/lib.bc`/
  `gen/lib2.bc` into embeddable C source) must run on the build host, but
  the rest of bc must compile for aarch64. Fixed simply by setting
  `CC="${OWNOS_TGT}-gcc -std=c99"` and leaving `HOSTCC` unset (defaults to
  the host's own `gcc`) — no native/target source-tree duplication
  needed, unlike File. `make test` (which runs the aarch64 binary) was
  skipped, same as every other package; `make`/`make install` alone are
  safe.

**General lesson for the rest of Stage 4:** before trusting a plain
`./configure --host=... && make && make install` for any package, check
whether its default `make`/`all` target depends on running its own
just-built output (test suites wired into `all` rather than a separate
`check`, or a "compile this generated-data file by running the binary we
just built" step). Autoconf-only packages are safe by construction; hand-
rolled Makefiles and anything with a `gen/`-style code-generation step are
not and need a case-by-case check.

### 3.5 Second methodology simplification: Tcl, Expect, DejaGNU dropped entirely (2026-10-06)

The book states this outright on the Tcl page: "This package and the next
two (Expect and DejaGNU) are installed to support running the test suites
for Binutils, GCC and other packages" — confirmed by also reading
Expect's and DejaGNU's own pages, neither of which names any other
LFS-internal consumer. Since D18 (already resolved: test suites are
deferred entirely to post-boot validation on the real Pi, because a
cross-compiled aarch64 binary cannot execute on this x86_64 build host to
run its own `make check`), these three packages have no purpose in this
build — nothing in this methodology will ever invoke `tclsh`, `expect`,
or `runtest`. **Dropped from the Stage 4 package list, same reasoning as
the Chapter 6 skip (§3.2).** Pkgconf is unaffected and still required —
it is a general build-time library-flag lookup tool used by other
packages' configure scripts, unrelated to testing.

### 3.6 Pkgconf needs the same host/target split as File

Pkgconf (2.5.1) itself is a normal autoconf cross-compile (no special
handling needed to build the aarch64 copy installed into
`$OWNOS_ROOT/usr/bin/pkgconf`, with a `pkg-config` compat symlink). But
later packages' `./configure` scripts run *on this x86_64 host* and may
call `pkg-config` at configure time to detect libraries — the aarch64
`pkgconf` binary just built can't execute here, so a **separate, native**
pkgconf is needed for the build itself. The build host already has one
(`pkgconf`/`pkg-config` 1.8.1, from the base container image). Pointed it
at the target sysroot instead of the host's own `.pc` files by exporting,
in `/build/env.sh`:

```sh
export PKG_CONFIG_SYSROOT_DIR=$OWNOS_ROOT
export PKG_CONFIG_LIBDIR=$OWNOS_ROOT/usr/lib/pkgconfig:$OWNOS_ROOT/usr/share/pkgconfig
```

(`PKG_CONFIG_PATH` deliberately left unset — setting it would additionally
search the host's own `.pc` files, exactly what must be avoided.) Verified
working: `pkg-config --cflags --libs zlib` now correctly returns
`-I/build/root/usr/include -L/build/root/usr/lib -lz`, the sysroot-adjusted
paths, not the host's.

### 3.7 GCC 15 defaults to `gnu23`: breaks old K&R-style configure-time test code

Hit while cross-compiling GMP 6.3.0: its configure script's own "long
long reliability" compile test uses a deliberately archaic, unprototyped
function definition (`void g(){}`, called later with 6 arguments — valid,
if undefined, under K&R/C89 rules). GCC 15 defaults its C dialect to
`gnu23`, and C23 made calling a function with more arguments than its
(implicit, empty) prototype declares a hard **error** instead of a
warning — so this and similar old test snippets fail to compile at all,
even though they're deliberately not "real" code, just probes. Not
specific to cross-compiling — would also break building GMP's configure
test natively, though nothing in this project has hit it outside a cross
context yet.

**Fixed globally**, not per-package: `/build/env.sh` now exports
`OWNOS_CC="${OWNOS_TGT}-gcc -std=gnu17"` and `OWNOS_CXX` alongside it, and
every subsequent `./configure` invocation passes `CC="$OWNOS_CC"`
explicitly. `gnu17` is a close superset of the C89/C99/C11 dialects most
LFS-era configure scripts and package code assume, pulls old-style
function definitions back down to a warning, and isn't expected to break
anything that doesn't specifically need `gnu23`-only features. Apply
`CC="$OWNOS_CC"` to every remaining package's configure call going
forward — this is exactly the kind of "silent toolchain-version drift"
problem Document 10/11 already expected from using a much newer GCC
(15.2.0) than the book was written against.

### 3.8 MPFR's decimal-float support needs an explicit runtime that isn't there

MPFR built fine, but linking anything against it (first hit: MPC's own
configure-time link check) failed with undefined references to
`__bid_*` symbols (`__bid_getd2`, `__bid_addtd3`, etc.) — MPFR's configure
auto-detected GCC 15's `_Decimal64`/`_Decimal128` *language* support and
enabled its optional decimal-float conversion functions
(`--enable-decimal-float` is the default when the compiler claims
support), but the matching *runtime* library (Intel's BID decimal
floating-point implementation, normally bundled into `libgcc` only for
targets that enable it, e.g. PowerPC/S390/x86 in distro-patched GCCs) was
never built into this project's `libgcc` and isn't linked automatically.
Decimal-float conversion is not something GCC's own build needs from
MPFR. **Fixed** by explicitly passing `--disable-decimal-float` to
MPFR's configure — not a workaround, the book doesn't use this feature
either, and disabling it removes the dependency on a runtime piece this
build doesn't have rather than trying to add one.

### 3.9 Binutils-final and GCC-final: a genuine Canadian cross

Unlike every other Chapter 8 package, Binutils and GCC's "final" builds
are not meant to run on this x86_64 build host at all — the whole point
is for the finished OS to have its own native compiler and assembler,
meaning the resulting `gcc`/`ld`/etc. binaries must themselves *run on*
aarch64 (the Pi). Since those binaries are still being produced *on*
x86_64, this is specifically what GCC's own build system calls a
**Canadian cross**: `build` (x86_64, where compilation happens) differs
from `host` (aarch64, where the result runs), and `host == target`
(aarch64, what the result itself compiles for) — distinct from Stage 3's
pass-1 cross-compiler, where `build == host` (x86_64) and only `target`
differs.

Both builds used `--build=x86_64-pc-linux-gnu --host=$OWNOS_TGT
--target=$OWNOS_TGT`, with `CC="$OWNOS_CC"` (Stage 3's pass-1 cross-
compiler — needed here because *this* build's output must run on
aarch64, exactly what pass 1 produces) and, for GCC specifically,
`CC_FOR_BUILD=gcc CXX_FOR_BUILD=g++` (the host's own native compiler,
for the handful of GCC-internal code generators — `genattrtab` and
friends — that must execute during the build itself, on x86_64, not on
the eventual aarch64 host).

Binutils-final needed nothing beyond the triplets — configure correctly
auto-detected the pass-1 `aarch64-unknown-linux-gnu-gcc` via the standard
`${host_alias}-gcc` naming convention, and the whole build/install
matched the book's Chapter 8 flags verbatim otherwise.

GCC-final needed one additional, important flag pair:
`--with-sysroot=/ --with-build-sysroot=$OWNOS_ROOT`. These serve
different purposes and are easy to conflate: `--with-sysroot` is the
default sysroot path *baked into the resulting compiler*, used when it
actually runs (on the Pi, where the real root is `/`, so this must be
`/`, not `$OWNOS_ROOT`). `--with-build-sysroot` is a separate override
used *only while GCC itself is being built right now*, so its own build
process can still find the aarch64 headers/libraries it needs — without
that path leaking into the installed compiler's permanent defaults.
Using `--with-sysroot=$OWNOS_ROOT` alone (the pass-1 pattern) would have
produced a GCC that only works correctly while still sitting inside this
build container, not after being copied onto the Pi's own root
filesystem. Verified via `strings`/`gcc`'s own `specs` file that the
installed driver uses `--sysroot=%R` (the runtime-relative mechanism
`--with-sysroot=/` produces), not a literal `$OWNOS_ROOT` path — the
only verification possible, since nothing in this build environment can
execute the resulting aarch64 binary directly.

One operational snag, unrelated to the Canadian-cross mechanics: a first
attempt's `build/` directory carried over a stale `config.cache` from
earlier in the session (Stage 3's pass-1 GCC work), causing `configure:
error: changes in the environment can compromise the build`. Traced to
`mv olddir newdir` silently moving the *contents* of a fresh extraction
**into** an already-existing `newdir` instead of replacing it, rather
than any container/mount issue — `rm -rf` on the target name before
`mv`, confirmed empty, is not sufficient if a *same-named* directory from
an earlier, unrelated extraction still exists elsewhere in the rename
path. Fixed by extracting to a distinctly-named directory
(`gcc-final-15.2.0`) and explicitly confirming no `build/` subdirectory
existed before configuring, rather than trusting `rm -rf` plus a rename.

Both results installed cleanly into `$OWNOS_ROOT`; `file` confirms
`gcc`, `g++`, `cc1`, `cc1plus`, `ld`, `readelf`, etc. are all genuine
aarch64 PIE executables, not more x86_64-hosted cross-tools. `/usr/bin/cc`
(a symlink to `gcc`) was created by hand — stock LFS creates it during
Chapter 6's GCC pass 2, which this methodology skips entirely, and
Chapter 8's own GCC page never recreates it since it assumes Chapter 6
already did.

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
