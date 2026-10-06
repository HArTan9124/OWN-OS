# 05 — Scope & Out-of-Scope

This restates and extends `docs/phase1/10-roadmap-and-open-decisions.md`
§10.3 at the product level, split by phase rather than by technical
stage.

## Phase 1 — In scope

Exactly the ~90-package LFS base system defined in
`docs/phase1/03-sources-and-packages.md`, built per
`docs/phase1/` Documents 2–8: a bootable, self-hosting, networked,
shell-accessible base Linux system with its own identity. No more, no
less. Full milestone checklist:
`docs/phase1/10-roadmap-and-open-decisions.md` §10.2.

## Phase 1 — Explicitly out of scope

(Verbatim from `docs/phase1/10-roadmap-and-open-decisions.md` §10.3,
restated here so this folder is self-contained:)

- Any BLFS package not in the Phase 1 list.
- Multilib / 32-bit compatibility on a 64-bit build.
- UEFI boot (unless `docs/phase1` D12 is changed before execution).
- Any package manager.
- Any init system other than SysVinit.
- DHCP / dynamic networking.
- Custom compiler optimization flags.
- Multi-architecture support.

## Phase 2+ — Candidate scope (nothing here is committed)

Gated entirely on `02-target-users-and-use-cases.md` resolving. Listed
here only so "what might come next" is visible, not because any of it
is approved:

- **If P2 (workstation) confirmed:** display server, window
  manager/desktop environment, browser and basic productivity
  applications, input device handling beyond console defaults, DHCP.
- **If P3 (server/appliance) confirmed:** the specific named service(s)
  and their dependencies only — not a general-purpose server toolkit.
- **If P4 (embedded/minimal) confirmed:** custom (non-`defconfig`-based)
  kernel configuration, stripped binaries reconsidered, possibly a
  read-only/immutable root design.
- **Any persona:** a package-management approach
  (`docs/phase1/05-base-system-build.md` §5.3), revisited once ongoing
  upgrade/removal needs actually exist rather than decided speculatively
  now.

## Scope change process

A scope change (something above moving from "candidate" to "in scope,"
or something currently in-scope being cut) must:
1. Be reflected as an updated requirement (`FR-xx`/`NFR-xx`) in
   Documents 3–4, not just mentioned in conversation.
2. Be logged with a date in
   `08-open-questions-and-decision-log.md`.
3. If it affects a technical decision already made in
   `docs/phase1/10-roadmap-and-open-decisions.md`, that table must be
   updated too — the two folders must never silently disagree about
   what's in scope.
