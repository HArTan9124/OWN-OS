# 07 — Making the System Bootable (LFS Chapter 10)

Source: LFS Chapter 10 — Introduction, Creating `/etc/fstab`, the Linux
kernel build, Using GRUB to Set Up the Boot Process.

## 7.1 `/etc/fstab`

```
# file system  mount-point    type     options             dump  fsck
/dev/<root>    /              ext4     defaults            1     1
/dev/<swap>    swap           swap     pri=1               0     0
proc           /proc          proc     nosuid,noexec,nodev 0     0
sysfs          /sys           sysfs    nosuid,noexec,nodev 0     0
devpts         /dev/pts       devpts   gid=5,mode=620      0     0
tmpfs          /run           tmpfs    defaults            0     0
devtmpfs       /dev           devtmpfs mode=0755,nosuid    0     0
tmpfs          /dev/shm       tmpfs    nosuid,nodev        0     0
cgroup2        /sys/fs/cgroup cgroup2  nosuid,noexec,nodev 0     0
```

Device identification note: device-node names (`/dev/sdaN`) can shift if
disks are added/removed/reordered; using `PARTUUID=` (from `lsblk -o
UUID,PARTUUID,PATH,MOUNTPOINT`) instead is more robust and is the
approach this plan recommends pairing with the matching GRUB
`search --fs-uuid` technique in §7.3 below — **decision needed**: plain
device paths (simpler, matches book's base example) vs. UUID-based
(more robust, recommended if the target disk layout might change, e.g.
across VM migrations).

## 7.2 Kernel build

This is flagged by the book itself as one of the most genuinely
difficult steps in the entire process — "almost 12,000 configuration
items," most irrelevant to any one machine, and getting it wrong risks a
kernel that won't boot at all. The book's own guidance, which this plan
adopts:

1. `make mrproper` first, always — never trust a tarball's tree to
   already be clean.
2. Start from `make defconfig` (sane baseline for the current
   architecture), then `make menuconfig` to adjust.
3. **Mandatory/critical settings** called out explicitly by the book
   (deviating risks an unbootable or malfunctioning system):
   - `CONFIG_DEVTMPFS=y` + `CONFIG_DEVTMPFS_MOUNT=y` (required for udev to function at all)
   - `CONFIG_UEVENT_HELPER` **disabled** (interferes with udev)
   - `CONFIG_CGROUPS=y` + `CONFIG_MEMCG=y`
   - `CONFIG_RELOCATABLE=y` + `CONFIG_RANDOMIZE_BASE=y` (KASLR)
   - `CONFIG_STACKPROTECTOR=y` + `CONFIG_STACKPROTECTOR_STRONG=y`
   - `CONFIG_WERROR` **disabled** (host compiler/config drift can otherwise hard-fail the build)
   - `CONFIG_EXPERT` **disabled** unless deliberately going off-script
   - Framebuffer/DRM chain for a usable console + panic diagnostics:
     `CONFIG_SYSFB_SIMPLEFB`, `CONFIG_DRM`, `CONFIG_DRM_PANIC` (+
     `CONFIG_DRM_PANIC_SCREEN=kmsg`), `CONFIG_DRM_SIMPLEDRM`,
     `CONFIG_DRM_FBDEV_EMULATION`, `CONFIG_FRAMEBUFFER_CONSOLE`
   - On x86_64 specifically: `CONFIG_X86_X2APIC`, and (dependency order
     matters in menuconfig) `CONFIG_PCI_MSI` before `CONFIG_IRQ_REMAP`
     before `CONFIG_X86_X2APIC`
   - **If the target disk is NVMe** (`/dev/nvme*` not `/dev/sd*`):
     `CONFIG_BLK_DEV_NVME=y` is non-negotiable — its absence means the
     system literally cannot boot from that disk.
   - UNIX98 PTY support must be on (same requirement as the host,
     Document 2 §2.1) — carried forward into the new kernel.
4. `make` (compile), `make modules_install` (if modules enabled).
5. Install artifacts into `/boot`:
   ```sh
   cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.16.1-lfs-12.4
   cp -iv System.map /boot/System.map-6.16.1
   cp -iv .config /boot/config-6.16.1
   cp -r Documentation -T /usr/share/doc/linux-6.16.1
   ```
6. **USB load-order fix** (avoids a boot-time warning, harmless but worth
   doing correctly): `/etc/modprobe.d/usb.conf` forcing `ehci_hcd` to
   load before `ohci_hcd`/`uhci_hcd`.
7. If retaining the kernel source tree for future reconfiguration
   (recommended — BLFS packages often need kernel config changes later):
   `chown -R 0:0` it, and remember **not** to re-run `make mrproper`
   casually on a retained tree during a config tweak (it nukes
   `.config` and every `.o`; only `make clean` after a *compiler* — not
   config — change). **Never** create a `/usr/src/linux` symlink — a
   pre-2.6-era convention explicitly flagged as harmful on a modern LFS
   system.

**UEFI note:** if the target will boot via UEFI (common for modern VMs
and hardware), kernel config needs additional adjustments documented in
BLFS, applicable even if the bootloader itself ends up being supplied by
a host/another distro rather than our own GRUB install. **Decision
needed:** BIOS/legacy-boot GRUB (what this plan's §7.3 below covers in
full) vs. UEFI GRUB (BLFS-documented, not detailed in this plan).
Recommendation: legacy BIOS boot for the first Phase 1 build (simpler,
fully covered by the base book), UEFI as a named follow-up if the actual
deployment target needs it.

## 7.3 GRUB (legacy BIOS boot path)

**Safety note up front:** GRUB installation overwrites the target disk's
boot sector. Have a rescue path ready (e.g. a bootable ISO/USB) before
running `grub-install` against any disk that matters.

GRUB's own disk/partition naming: `(hd<n>,<m>)` — drive numbers 0-based,
*partition* numbers 1-based (5+ for extended partitions); CD-ROMs are
not counted as hard drives in this numbering.

```sh
grub-install /dev/<disk>        # writes MBR + /boot/grub
```

(add `--target i386-pc` if the system was booted via UEFI during the
build but we specifically want a BIOS-mode install — mismatched target
files cause `grub-install` to silently reference files that were never
installed in Chapter 8).

`/boot/grub/grub.cfg`:
```
set default=0
set timeout=5
insmod part_gpt
insmod ext2
set root=(hd0,2)
set gfxpayload=1024x768x32

menuentry "GNU/Linux, Linux 6.16.1-lfs-12.4" {
        linux   /boot/vmlinuz-6.16.1-lfs-12.4 root=/dev/sda2 ro
}
```

Notes carried over because they're easy to get subtly wrong:
- `gfxpayload` sets the VESA framebuffer mode so the kernel's SimpleDRM
  panic-screen path (§7.2) actually has something to draw to.
- If using a separate `/boot` partition (Document 2 §2.3), drop the
  `/boot` prefix from the `linux` line and point `set root` at the boot
  partition, not the root partition.
- GRUB's `(hdX,Y)` designators can silently shift if disks are
  added/removed later (including USB thumb drives at boot time) — the
  more robust alternative is `search --set=root --fs-uuid <filesystem
  UUID>` plus `root=PARTUUID=<partition UUID>` on the kernel line
  (**not** `root=UUID=<filesystem UUID>`, which needs an initramfs LFS
  doesn't set up). Pairs with the `/etc/fstab` UUID decision in §7.1 —
  make both decisions together, consistently.
- **Never** run `grub-mkconfig`/rely on `/etc/grub.d/` scripts on an LFS
  system — those are built for binary distros and will blow away hand
  customization. Keep a backup of a working `grub.cfg` regardless.

## 7.4 Checklist for this document

- [ ] `/etc/fstab` written; device-path vs. UUID approach decided and applied consistently with GRUB config
- [ ] Kernel `.config` built from `defconfig` + the mandatory settings list above, including NVMe support if applicable
- [ ] Kernel compiled, modules installed, artifacts copied to `/boot`
- [ ] USB load-order modprobe fix applied
- [ ] Kernel source tree ownership/retention decision made
- [ ] BIOS vs. UEFI boot path decided (recommend BIOS for Phase 1)
- [ ] GRUB installed to the correct disk, `grub.cfg` hand-written and verified, rescue media available before first reboot
