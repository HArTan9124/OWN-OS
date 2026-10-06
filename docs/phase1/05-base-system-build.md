# 05 — Base System Build (LFS Chapter 8)

Source: LFS Chapter 8 — Introduction, Package Management, and the full
~90-package build sequence, About Debugging Symbols, Stripping, Cleaning
Up.

This is the largest chapter in the book by far and the longest-running
part of Phase 1 in wall-clock terms. Every package below is built
**natively, permanently, with test suites** (where applicable) — this is
the real, final OWN-OS userland, not a temporary stand-in.

## 5.1 Ground rules for this chapter

- **No custom compiler optimizations.** The book explicitly recommends
  against hand-tuned `-march`/`-mtune`/custom optimization flags for a
  first build: the risk of subtle miscompilation is not worth the
  marginal speed gain, and untested flag combinations specifically risk
  breaking Binutils/GCC/Glibc. Default package optimization levels
  (`-O2`/`-O3` as each package's own build system already chooses) are
  kept.
- **No static libraries, with two necessary exceptions.** `--disable-static`
  (or equivalent) is passed wherever a package supports it. Static
  linking is obsolete for most purposes and creates a security-patching
  trap (every statically-linked consumer needs re-linking after a
  library security fix). Glibc and GCC are the two packages where static
  libs remain structurally necessary.
- **Build order is fixed and intentional** (dependency-driven — see
  Appendix C of the book, "Dependencies", for the full graph). Do not
  reorder packages casually.
- **Each package page in the book provides:** a short description,
  an approximate SBU time, approximate disk space needed, the exact
  `./configure`/`make`/`make install` invocation (with every non-obvious
  flag explained), and a "Contents" subsection listing every
  binary/library installed and what it's for. Implementers should pull
  each package's actual page at build time rather than relying on a
  paraphrase here — see the scope note in the top-level README.

## 5.2 Full build order (Chapter 8)

Man-pages → Iana-Etc → **Glibc** (2nd/final build — see §5.2.1) → Zlib →
Bzip2 → Xz → Lz4 → Zstd → File → Readline → M4 → Bc → Flex → Tcl →
Expect → DejaGNU → Pkgconf → **Binutils** (final) → GMP → MPFR → MPC →
Attr → Acl → Libcap → Libxcrypt → Shadow → **GCC** (final) → Ncurses →
Sed → Psmisc → Gettext → Bison → Grep → Bash → Libtool → GDBM → Gperf →
Expat → Inetutils → Less → Perl → XML::Parser → Intltool → Autoconf →
Automake → OpenSSL → Libelf (from Elfutils) → Libffi → Python →
Flit-core → Packaging → Wheel → Setuptools → Ninja → Meson → Kmod →
Coreutils → Diffutils → Gawk → Findutils → Groff → GRUB → Gzip →
IPRoute2 → Kbd → Libpipeline → Make → Patch → Tar → Texinfo → Vim →
MarkupSafe → Jinja2 → **Udev** (from Systemd source, built standalone —
see note) → Man-DB → Procps-ng → Util-linux → E2fsprogs → Sysklogd →
**SysVinit** → *(About Debugging Symbols → Stripping → Cleaning Up)*.

**Note on Udev:** LFS extracts only the udev component out of the full
Systemd source tree and builds it standalone (via the `udev-lfs` helper
tarball), specifically so LFS's base system keeps SysVinit rather than
pulling in all of systemd-as-init. This is a deliberate, named design
decision in the book worth flagging because it is the first real init
system fork-in-the-road OWN-OS will face later (Document 10).

### 5.2.1 Why Glibc is rebuilt again here

Even though Glibc was already built once cross-compiled in Chapter 5,
it is rebuilt fully natively in Chapter 8 (no cross-compilation
limitations, full feature set, full test suite). The book flags this as
one of the chapter's most consequential single steps — an incorrectly
upgraded/rebuilt Glibc in a *running* system (relevant for future OWN-OS
upgrades, not this initial build) can break everything, so its own page
has extensive upgrade-safety notes worth reading in full at
implementation time even though not strictly needed for a from-scratch
first build.

## 5.3 Package management — decision needed now, not after

The book deliberately ships **no package manager** and does not endorse
one, for two stated reasons: it would shift focus away from the
teaching/understanding goal, and no single approach suits every
audience. It does catalogue the common community techniques (Document
summary below) precisely so a project like OWN-OS can pick one
*before* Chapter 8 starts, since retrofitting tracking onto an
already-built system is much harder than tracking as you go.

| Technique | Summary | Fit for OWN-OS Phase 1 |
|---|---|---|
| "It's all in my head" | No tooling; rebuild everything if something changes | Not viable past Phase 1 |
| Install in separate dirs (`/opt/pkg-x.y` + symlink) | Simple, but explicitly **never** recommended system-wide | Rejected by upstream guidance |
| Symlink-style (Stow/Epkg/Graft/Depot) | Fake-install via `DESTDIR` into a side tree, then symlink into `/usr` | Viable later, adds complexity now |
| Timestamp-based (install-log) | Snapshot filesystem mtimes before/after install | Simple but unreliable with parallel installs |
| Tracing installs (`LD_PRELOAD` shim or `strace`) | Capture every file-system syscall during `make install` | Accurate but invasive, more Phase-2 territory |
| Package archives (RPM/dpkg/Slackware-tar style) | Fake-install to a side tree, then archive it | Most "real distro"-like; most work to stand up |
| User-based (one UID per package) | Identify a package's files by owning UID | LFS-specific hint-project technique; unusual |

**Decision needed (see Document 10):** pick one (or explicitly "none for
Phase 1, revisit in Phase 2") before starting Chapter 8. Recommendation:
**defer** — build Phase 1 with no package manager (matches upstream LFS
exactly, keeps Phase 1 scope minimal and teaching-focused), explicitly
plan package management as a named Phase 2 decision once the base system
boots. This avoids adding build-time complexity to an already-large
first milestone.

## 5.4 Finishing Chapter 8

1. **About Debugging Symbols / Stripping (optional but recommended):**
   stripping debug symbols from binaries/libraries can save ~2 GB and
   50–80% of file size per binary, at the cost of losing in-place
   debuggability. The book's exact `strip --strip-unneeded` / selective
   `objcopy --only-keep-debug` procedure (preserving a few debug
   artifacts for Glibc/GCC-adjacent libraries, for future BLFS use with
   gdb/valgrind) should be followed verbatim — it's easy to crash a
   running process by stripping its backing file in place, which is why
   the book's procedure copies to `/tmp`, strips there, then reinstalls
   via `install` rather than overwriting live files directly.
   **Decision needed:** strip for a leaner Phase 1 image, or skip to keep
   full debuggability while the OS is this young. Recommendation: skip
   stripping for the *first* Phase 1 build (debuggability while
   diagnosing our own first-build issues is worth more than 2 GB), revisit
   once the build is proven stable.
2. **Cleaning up:** purge `/tmp/*`, delete remaining libtool `.la` files,
   remove the now-fully-obsolete `$LFS_TGT`-prefixed cross-compiler
   remnants, and `userdel -r tester` (the temporary Chapter 7 test
   account).

## 5.5 Checklist for this document

- [ ] Package-management strategy decided (recommend: none for Phase 1)
- [ ] Optimization-flag policy confirmed (recommend: defaults only)
- [ ] Static-library policy confirmed (disable except Glibc/GCC)
- [ ] All ~90 packages built in book order, test suites run per package guidance
- [ ] Stripping decision made and applied (or explicitly deferred)
- [ ] `/tmp` cleaned, `.la` files removed, cross-compiler remnants removed, `tester` account deleted
