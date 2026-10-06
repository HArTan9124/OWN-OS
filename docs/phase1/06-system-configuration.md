# 06 — System Configuration (LFS Chapter 9)

Source: LFS Chapter 9 — Introduction (SysVinit run levels),
LFS-Bootscripts, Overview of Device and Module Handling, Managing
Devices, General Network Configuration, System V Bootscript Usage and
Configuration, Configuring the System Locale, `/etc/inputrc`,
`/etc/shells`.

## 6.1 Init system choice: SysVinit (as shipped) vs. systemd

LFS presents **two boot-process families** and picks SysVinit for the
main book:

**System V** (what LFS 12.4 actually ships): classic `init` +
numbered run levels (0 halt, 1 single-user, 2/4 user-definable, 3
multi-user, 5 multi-user+display-manager, 6 reboot). Pros: established,
well understood, easy to hand-customize. Cons: slower boot (serial
task execution; ~8–12s kernel-message-to-login on medium hardware, book's
own benchmark), no native cgroups/per-user fair-share scheduling, adding
scripts means manually choosing numeric sequencing.

**systemd**: mentioned by LFS only as "an alternative boot procedure" it
does not cover — explicitly out of the main book's scope (BLFS territory
if ever adopted).

**Decision needed (Document 10):** Phase 1, as planned here, follows
upstream LFS exactly and ships **SysVinit**. This is the single biggest
architectural fork OWN-OS could choose to diverge on later, so it's
called out explicitly rather than assumed silently. Recommendation:
ship SysVinit for Phase 1 (it's what the entire rest of this book's
bootscripts/config assume; switching to systemd as init is a Phase 2+
BLFS-level undertaking, not a Phase 1 tweak).

## 6.2 LFS-Bootscripts package

Installs the actual run-level machinery: `rc` (master run-level
controller — runs numbered symlinks per level), `functions`
(shared helper routines + status/error reporting), `checkfs` /
`cleanfs` / `mountfs` / `mountvirtfs` / `swap` (filesystem lifecycle),
`console` (keymap/font), `ifup`/`ifdown`/`network`/`localnet`
(networking), `modules` (module autoload from
`/etc/sysconfig/modules`), `setclock`/`sysctl`/`sysklogd`,
`udev`/`udev_retry`, `sendsignals`/`reboot`/`halt`, and a `template` for
writing new custom service scripts. Installed under `/etc/rc.d`,
`/etc/init.d` (symlink), `/etc/sysconfig`, `/lib/services`, `/lib/lsb`
(symlink).

## 6.3 Device & module handling (udev)

Historical context worth retaining institutionally: static `/dev`
(MAKEDEV-era) → `devfs` (deprecated/removed 2006, race-condition-prone,
bad naming policy) → current model: kernel-side `devtmpfs` creates
nodes as hardware is detected, `udevd` (userspace, reading `sysfs`
events) layers permissions/ownership/symlinks on top via rules in
`/etc/udev/rules.d`, `/usr/lib/udev/rules.d`, `/run/udev/rules.d`
(merged, numerically ordered).

Known gotchas worth pre-empting rather than debugging later:
- A module not auto-loading usually means its driver lacks a proper
  `sysfs` `modalias` export (common for ISA-class buses) — not a udev
  bug; load it explicitly via `/etc/sysconfig/modules`.
- An unwanted module loading because of an overly broad alias match
  → blacklist it in `/etc/modprobe.d/blacklist.conf`.
- A "wrapper" module (e.g. `snd-pcm-oss` over `snd-pcm`) that should load
  *after* its target → use a `softdep ... post:` line, not a blacklist.
- Device *naming order* is intentionally non-deterministic across boots
  by design (parallel uevent handling) — never hardcode assumptions
  about which of several identical devices gets which `/dev/sdX`;
  instead write udev rules keyed on stable attributes (serial number,
  vendor/device ID) for anything that needs a stable name (CD-ROM
  symlinks, duplicate webcam/tuner-style devices — both covered with
  worked examples in the source material).

**Network interface naming:** modern udev assigns names like `enp5s0`
based on firmware/bus/slot data specifically so names don't shuffle
across reboots the way old `eth0`/`eth1` numbering could. Traditional
`ethN` naming can be restored via the `net.ifnames=0` kernel command-line
parameter (set in GRUB, Document 7) if preferred — **decision needed**,
recommend keeping the modern persistent naming scheme unless there's a
specific reason not to.

## 6.4 Network configuration

Per-interface config files live at `/etc/sysconfig/ifconfig.<iface>`
(e.g. `ifconfig.eth0`), each setting `ONBOOT`, `IFACE`, `SERVICE`
(`ipv4-static` for the book's example — DHCP needs a BLFS-level addition
not covered in Phase 1), `IP`, `GATEWAY`, `PREFIX`, `BROADCAST`. Also
needed: `/etc/resolv.conf` (DNS servers — Google public DNS 8.8.8.8 /
8.8.4.4 cited as a fallback example), `/etc/hostname`, and a fully
worked `/etc/hosts` (loopback + FQDN + private-range IP guidance: 10.x
`/8`, 172.16–31.x `/16`, 192.168.x `/24`).

**Decision needed:** static IP (book default, simplest for a first
build/VM) vs. DHCP (needs BLFS `dhcpcd` or similar — out of Phase 1
scope as currently planned). Recommendation: static IP for Phase 1
bring-up/testing; revisit when OWN-OS targets real variable network
environments.

## 6.5 SysVinit configuration detail

`/etc/inittab` wires run levels to `/etc/rc.d/init.d/rc <N>`, plus
`Ctrl-Alt-Del` → shutdown, `sulogin` for single-user/rescue, and
`agetty` on 6 virtual consoles (tty1–tty6). `rc` executes the numbered
`K##`/`S##` symlinks in each `/etc/rc.d/rcN.d` (and `rcS.d` for the
sysinit phase) in ascending numeric order — `K` = stop, `S` = start, both
pointing at the same real script under `/etc/rc.d/init.d/`, called with
`start|stop|restart|reload|status`. The default run level (`initdefault`
in `inittab`) is conventionally **3** (multi-user, networked, no GUI) —
appropriate for Phase 1 since no display manager/GUI is in scope yet.

Also configured here: `/etc/sysconfig/clock` (`UTC=1` or `0` — whether
the hardware/CMOS clock is UTC or local time; get this wrong and every
timestamp on the system is off by a timezone), `/etc/sysconfig/console`
(keymap/font/UTF-8 console mode — matters once locale, §6.6, is
anything other than plain `C`), `/etc/sysconfig/createfiles`, and the
umbrella override file `/etc/sysconfig/rc.site` (boot-speed tuning:
skipping `udev settle` waits, skipping `fsck` verbosity/the whole check
on trusted reboots, skipping `/tmp` cleanup, shutdown signal-kill delay,
etc. — all legitimate "make our boots faster later" knobs worth revisiting
once Phase 1 is stable, not needed for the first correctness-focused
build).

## 6.6 Locale

The shell startup chain (`/etc/profile` for login shells, sourcing a
`LANG=<ll>_<CC>.<charmap>` setting) determines translated program
output, correct character classification for non-ASCII input, sort
order, paper size default, and monetary/date formatting. The book's own
default/fallback is `C.UTF-8` specifically when `$TERM = linux` (the bare
Linux console), to avoid programs emitting characters the plain console
framebuffer can't render; a fuller locale is used otherwise.

**Decision needed:** which locale(s) OWN-OS ships/defaults to.
Recommendation: `C.UTF-8` as the safe universal default for Phase 1
(matches the console-safe fallback the book itself uses), with the
mechanism in place to add a richer locale later without re-architecting
anything.

## 6.7 Small remaining files

- `/etc/inputrc` — global Readline key-binding config (used by bash and
  most interactive CLI tools); book provides a ready-to-use generic
  version (8-bit input, no bell, Home/End/Delete key sequences for
  console/xterm/Konsole).
- `/etc/shells` — lists valid login shells (`/bin/sh`, `/bin/bash`);
  consulted by `chsh` and some daemons (FTP, display managers) to decide
  what counts as a legitimate shell.

## 6.8 Checklist for this document

- [ ] Init system confirmed: SysVinit (Document 10 sign-off)
- [ ] LFS-Bootscripts installed
- [ ] udev rules reviewed; blacklist/softdep needs (if any, hardware-dependent) identified
- [ ] Network naming scheme decided (persistent vs. legacy `ethN`)
- [ ] Static-vs-DHCP networking decided; `/etc/sysconfig/ifconfig.*`, `/etc/resolv.conf`, `/etc/hostname`, `/etc/hosts` written
- [ ] `/etc/inittab`, `/etc/sysconfig/clock`, `/etc/sysconfig/console`, `/etc/sysconfig/rc.site` configured
- [ ] Locale decided (recommend `C.UTF-8` default) and `/etc/profile` written
- [ ] `/etc/inputrc`, `/etc/shells` created
