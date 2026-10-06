# 03 — Sources & Package Inventory

Source: LFS Chapter 3 (Packages and Patches) — Introduction, All
Packages, Needed Patches. Also Chapter 4 §4.2 (directory layout) for
where sources live during the build.

## 3.1 Where sources live

All package tarballs and patches are downloaded once, into a directory
reachable from inside the eventual chroot — the book's convention is
`$LFS/sources/`:

```sh
mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources   # sticky bit: shared, world-writable, no cross-deleting
```

**Before downloading anything:** check the LFS security advisories
(https://www.linuxfromscratch.org/lfs/advisories/) in case a listed
version has a since-discovered vulnerability and a newer point release
should be substituted instead.

## 3.2 Full package list (LFS 12.4 stable)

~90 packages, every one individually justified in the book (full
rationale per package summarized in Document 1 §1.5 and repeated per
build-stage in Documents 4/5). Versions below are exactly what LFS 12.4
pins — **do not** silently upgrade a version without checking the LFS
errata/advisories page first; the book's instructions are written and
tested against these exact releases and mismatches are a common source of
build breakage.

| Package | Version | Package | Version |
|---|---|---|---|
| Acl | 2.3.2 | Libxcrypt | 4.4.38 |
| Attr | 2.5.2 | Linux kernel | 6.16.1 |
| Autoconf | 2.72 | Lz4 | 1.10.0 |
| Automake | 1.18.1 | M4 | 1.4.20 |
| Bash | 5.3 | Make | 4.4.1 |
| Bc | 7.0.3 | Man-DB | 2.13.1 |
| Binutils | 2.45 | Man-pages | 6.15 |
| Bison | 3.8.2 | Meson | 1.8.3 |
| Bzip2 | 1.0.8 | MPC | 1.3.1 |
| Coreutils | 9.7 | MPFR | 4.2.2 |
| DejaGNU | 1.6.3 | Ncurses | 6.5-20250809 |
| Diffutils | 3.12 | Ninja | 1.13.1 |
| E2fsprogs | 1.47.3 | OpenSSL | 3.5.2 |
| Elfutils | 0.193 | Packaging (Python) | 25.0 |
| Expat | 2.7.1 | Patch | 2.8 |
| Expect | 5.45.4 | Perl | 5.42.0 |
| File | 5.46 | Pkgconf | 2.5.1 |
| Findutils | 4.10.0 | Procps-ng | 4.0.5 |
| Flex | 2.6.4 | Psmisc | 23.7 |
| Flit-core (Python) | 3.12.0 | Python | 3.13.7 |
| Gawk | 5.3.2 | Readline | 8.3 |
| GCC | 15.2.0 | Sed | 4.9 |
| GDBM | 1.26 | Setuptools (Python) | 80.9.0 |
| Gettext | 0.26 | Shadow | 4.18.0 |
| Glibc | 2.42 | Sysklogd | 2.7.2 |
| GMP | 6.3.0 | Systemd (udev source only) | 257.8 |
| Gperf | 3.3 | SysVinit | 3.14 |
| Grep | 3.12 | Tar | 1.35 |
| Groff | 1.23.0 | Tcl | 8.6.16 |
| GRUB | 2.12 | Texinfo | 7.2 |
| Gzip | 1.14 | Time Zone Data | 2025b |
| Iana-Etc | 20250807 | Udev-lfs tarball | 20230818 |
| Inetutils | 2.6 | Util-linux | 2.41.1 |
| Intltool | 0.51.0 | Vim | 9.1.1629 |
| IPRoute2 | 6.16.0 | Wheel (Python) | 0.46.1 |
| Jinja2 (Python) | 3.1.6 | XML::Parser (Perl) | 2.47 |
| Kbd | 2.8.0 | Xz Utils | 5.8.1 |
| Kmod | 34.2 | Zlib | 1.3.1 |
| Less | 679 | Zstd | 1.5.7 |
| LFS-Bootscripts | 20250827 | | |
| Libcap | 2.76 | | |
| Libffi | 3.5.2 | | |
| Libpipeline | 1.5.8 | | |
| Libtool | 2.5.4 | | |

Plus **Python documentation** (3.13.7) and **Tcl documentation**
(8.6.16) and **Systemd man pages** (257.8) as optional reference
tarballs, and **MarkupSafe** (3.0.2) as a Jinja2 build dependency.

Note: Glibc and several others (Binutils, GCC, Ncurses, Sed, Gettext,
Bison, Grep, Bash, M4, File, Findutils, Gawk, Gzip, Make, Patch, Tar, Xz,
Util-linux, Perl, Texinfo) are **built twice** — once as temporary
tools (Document 4), once as the final, permanent version (Document 5).
This is intentional, not a mistake in this table.

Total download size: ≈ 400–450 MB of compressed source tarballs (the
book's own total omits a couple of entries due to a known page rendering
bug, so budget generously and expect disk usage several times larger
once extracted/compiled).

## 3.3 Required patches

Six small patches are mandatory for LFS 12.4 (fixing packaging/build
issues upstream hasn't addressed yet):

| Patch | Target | Size |
|---|---|---|
| bzip2-1.0.8-install_docs-1.patch | Bzip2 | 1.6 KB |
| coreutils-9.7-upstream_fix-1.patch | Coreutils | 4.1 KB |
| coreutils-9.7-i18n-1.patch | Coreutils | 159 KB |
| expect-5.45.4-gcc15-1.patch | Expect | 12 KB |
| glibc-2.42-fhs-1.patch | Glibc | 2.8 KB |
| kbd-2.8.0-backspace-1.patch | Kbd | 12 KB |
| sysvinit-3.14-consolidated-1.patch | SysVinit | 2.5 KB |

All patches come from `https://www.linuxfromscratch.org/patches/lfs/12.4/`.
An optional, community-maintained patches database also exists
(`/patches/downloads/`) for non-mandatory fixes — not needed for a
first Phase 1 build.

## 3.4 Acquisition & verification plan

1. Script a single download pass (wget/curl) pulling every URL above into
   `$LFS/sources`, each with its documented MD5 sum checked (the book
   provides an MD5 for every package and patch — verify, don't skip).
2. Treat a failed/renamed upstream URL as a signal to check the LFS
   advisories page first (vulnerability-driven removals happen) before
   substituting a mirror or newer point release.
3. Do **not** mix tarball versions from a different LFS book release —
   the book is explicit that reusing a prior release's scripts/sources
   against this release's instructions causes subtle breakage.
4. Keep the downloaded tarballs after extraction — Chapter 7's backup
   step (Document 9 §9.3) archives the whole `$LFS` tree including
   `sources/`, so a mid-build restore doesn't require re-downloading.

## 3.5 Checklist for this document

- [ ] Create `$LFS/sources` with sticky-bit permissions
- [ ] Check current LFS security advisories before finalizing versions
- [ ] Download all ~90 package tarballs + 6 patches, verify MD5 sums
- [ ] Confirm total disk budget (sources + build + final system, see Document 2 §2.3)
