# 09 — Testing, Validation, Backups & Risk Register

Source: LFS Chapter 4 (About SBUs, About the Test Suites), Chapter 7
§7.13 (Cleaning up and Saving the Temporary System), Chapter 8 (test
suite guidance threaded through package instructions), general
troubleshooting references scattered through the book.

## 9.1 Test suite policy

- **Chapters 5–6: do not run test suites.** The binaries are
  cross-compiled and literally cannot execute on the build host — any
  "failure" here is a tooling artifact, not signal.
- **Chapter 7 onward: test suites become meaningful.** The book singles
  out **Binutils, GCC, and Glibc** as the three where running the full
  test suite is strongly recommended regardless of how long it takes
  (GCC's and Glibc's suites can run long, especially on modest
  hardware) — these three sit underneath everything else, so a defect
  here is the most expensive kind to discover late.
- **Known false-failure source:** a large number of failing
  Binutils/GCC tests is frequently caused by **PTY exhaustion** from a
  misconfigured host `devpts` filesystem, not a real defect. If test
  failures spike unexpectedly, check this before assuming the package
  itself is broken.
- **Reference point for "is this failure expected?"** — the LFS project
  publishes its own build logs (https://www.linuxfromscratch.org/lfs/build-logs/12.4/)
  showing which test failures are already known/accepted upstream for
  this exact release. Check there before spending time root-causing a
  failure that's already a documented, accepted non-issue.
- **Policy for OWN-OS Phase 1:** run test suites for every package that
  offers one from Chapter 7 onward (not just the "big three"), budget
  the extra wall-clock time for it (§9.2), and treat any *unexpected*
  failure (not matching the published build logs) as a stop-and-investigate
  event rather than something to wave through.

## 9.2 Time budgeting (SBU methodology)

LFS deliberately avoids absolute time estimates (hardware varies too
much — GCC alone ranges from ~5 minutes to multiple days depending on
the machine) and instead expresses every package's build time as a
multiple of the **Standard Build Unit (SBU)**: however long Binutils
pass 1 (Document 4 §4.3, step 1) takes to build on **one core**, on
*this* machine, measured end-to-end from `configure` through `make
install` (the book recommends literally wrapping that one command in
`time { ... }` to capture it precisely).

Practical plan:
1. Measure SBU on the actual build host/container as the very first
   timed step of Phase 1 execution.
2. Multiply by each subsequent package's book-quoted SBU figure to get
   a realistic total wall-clock estimate *before* committing to a single
   continuous build session (ties into the resume-strategy decision in
   Document 2 §2.2).
3. Re-derive the estimate if build parallelism (`-j` / `MAKEFLAGS`,
   Document 4 §4.1 step 3) changes mid-build — SBU figures in the book
   are themselves based on 4 cores (`-j4`) for everything except the
   pass-1 Binutils baseline itself (always 1 core, by definition).
4. Note that heavier parallelism makes *individual* SBU figures less
   reliable (interleaved build output, more system-load variance) —
   if a build step fails under high parallelism, the book's own advice
   is to retry single-threaded before concluding it's a real bug, purely
   to get clean, attributable error output.

## 9.3 Backup/restore checkpoints

The book provides one official checkpoint, at the end of Chapter 7 (end
of Document 4 in this plan) — right after the temporary toolchain is
fully built and cleaned up, and right before the much longer, much more
consequential Chapter 8 (Document 5) begins:

```sh
# from OUTSIDE the chroot, virtual filesystems already unmounted:
cd $LFS
tar -cJpf $HOME/lfs-temp-tools-12.4.tar.xz .
```

Restoring (only ever needed after a serious mid-build mistake):
```sh
cd $LFS
rm -rf ./*            # EXTREMELY DESTRUCTIVE if $LFS is wrong/unset — verify first
tar -xpf $HOME/lfs-temp-tools-12.4.tar.xz
# then re-mount virtual filesystems and re-enter chroot before continuing
```

**This repo's equivalent plan:** since source tarballs live under
`$LFS/sources` and are included in this archive, a restore never
requires re-downloading anything — treat this as cheap insurance, not
an optional nice-to-have. **Decision needed:** where this archive is
stored for our build (local disk vs. uploaded elsewhere) given the
build is expected to run in an ephemeral cloud session/container —
losing the container without having exported this checkpoint means
restarting Chapter 4–7 from scratch.

**Recommendation beyond the book's single checkpoint:** given OWN-OS is
running in an ephemeral session, also snapshot/export at the end of
Chapter 8 (end of Document 5, i.e. once the full permanent base system
is built but before Chapter 9–10 configuration) — the single longest,
most expensive stage to redo if something goes wrong afterward.

## 9.4 Risk register (known, named failure modes from the source material)

| Risk | Where it bites | Mitigation already in this plan |
|---|---|---|
| Host toolchain below minimum version, or above the untested upper bound | Any point in Ch. 5–8 | `version-check.sh` gate, Document 2 §2.1 |
| `/usr/lib64` accidentally created | Anywhere after Document 4 §4.1 | Explicit, repeated check called out in Documents 4 and 5 |
| Build run as `root` instead of `lfs` in Ch. 5–6 | Chapters 5–6 | Explicit safety rule, Document 4 §4.4 |
| `$LFS` unset/wrong when running a root-context command | Any reboot/resume point, especially Ch. 7 backup/restore | Document 2 §2.5–2.6, re-check ritual |
| PTY exhaustion from host `devpts` misconfig | Binutils/GCC test suites | Named explicitly, §9.1 above |
| Reused scripts/sources from a *different* LFS release | Anywhere | Explicit anti-pattern called out, Document 3 §3.4 |
| Static-vs-shared library confusion causing security-patch gaps later | Post-boot maintenance | Static-lib policy fixed in Document 5 §5.1 |
| Package manager retrofitted after the fact (harder than deciding up front) | Post-Chapter-8 | Explicit decision point, Document 5 §5.3 / Document 10 |
| Kernel config missing a mandatory option (devtmpfs, NVMe, etc.) → unbootable | Chapter 10 | Mandatory-settings checklist, Document 7 §7.2 |
| GRUB overwriting the wrong disk's boot sector | Chapter 10 | Explicit safety note + rescue-media requirement, Document 7 §7.3 |
| No root password set before first reboot | End of Chapter 11 | Explicit pre-reboot checklist item, Document 8 §8.2 |
| Mid-build container/session loss (environment-specific risk for this project) | Any stage, but worst in Ch. 8 | Backup/export checkpoints, §9.3 above |
| Drifting from upstream LFS pinned versions without checking advisories first | Document 3 sourcing | Explicit advisory-check step, Document 3 §3.4 |

## 9.5 Checklist for this document

- [ ] Test-suite policy agreed (recommend: run from Ch. 7 onward, all packages, not just the "big three")
- [ ] SBU measured on the actual build host as the first timed step
- [ ] Total wall-clock estimate produced before committing to a build-session plan
- [ ] Ch. 7 backup taken and exported somewhere durable (not just container-local disk)
- [ ] Additional end-of-Ch.8 checkpoint taken
- [ ] Risk register reviewed by whoever executes the build, not just read once at planning time
