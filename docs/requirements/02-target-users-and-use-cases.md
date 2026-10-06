# 02 — Target Users & Use Cases

**Status:** Draft — this entire document is downstream of the single
unanswered question in `01-prd-own-os.md` §6 / `docs/phase1` D15. Every
persona and scenario below is a *candidate*, not a confirmed target.

## 1. Candidate personas

### P1 — The builder (confirmed, exists today)
The project owner, building and operating OWN-OS directly: writing
config, running the build, debugging boot failures, deciding what goes
in next. This persona is **real and confirmed** — it's who Phase 1 is
being built for right now, regardless of how the other personas below
resolve.

**Needs:** a working shell and core toolchain (satisfied by Phase 1
itself), reliable boot, enough logging/diagnostics to debug problems
without external tooling, a documented and repeatable build.

### P2 — Workstation user **[CANDIDATE — confirm or reject]**
Someone (possibly still just the builder, wearing a different hat) using
OWN-OS as a daily-use desktop/laptop OS: needs a display stack, a
browser, file management, possibly office/productivity software.

**If confirmed**, this pulls in a large BLFS package set
(`docs/phase1/08-finalization-and-post-lfs.md` §8.5) — a graphical
environment, desktop environment, and end-user applications — and
likely tips several Document-10 decisions in `docs/phase1` toward
"richer" defaults (locale beyond `C.UTF-8`, DHCP over static IP, a
package manager sooner rather than deferred).

### P3 — Server/appliance **[CANDIDATE — confirm or reject]**
OWN-OS running a specific service (web, database, file, build server,
etc.) with no interactive graphical use at all.

**If confirmed**, pulls in a much smaller, targeted BLFS package set
(the specific service's dependencies only) and favors the *leaner*
defaults already recommended in `docs/phase1` (static IP, no desktop
stack, minimal footprint, stripped binaries reconsidered for size).

### P4 — Embedded/minimal target **[CANDIDATE — confirm or reject]**
OWN-OS as a small-footprint image for a specific device or VM image,
optimized for size/boot time over general-purpose flexibility.

**If confirmed**, this has the most knock-on effects of any candidate:
revisits stripping (`docs/phase1` D5) toward "strip," revisits kernel
config toward a minimal/custom config rather than `defconfig`-based,
and likely wants an immutable or read-only root filesystem design not
currently planned anywhere.

### P5 — Reference/teaching artifact **[CANDIDATE — confirm or reject]**
OWN-OS exists primarily as a finished example of a from-scratch build —
not meant for daily operational use by anyone, success is measured by
"it builds, it boots, it's documented," not by ongoing service.

**If confirmed**, this *lowers* the bar on several later-phase
requirements (no urgency on a package manager, no urgency on a display
stack) and *raises* the bar on documentation quality/completeness
(this folder and `docs/phase1/` become the actual deliverable, not just
supporting material).

## 2. Use-case scenarios (write once a persona is confirmed)

This section is intentionally left as a template rather than filled in
speculatively — filling in detailed scenarios (e.g. "user opens a
browser and reads email" vs. "service receives an HTTP request and
queries a database") for every candidate persona above would mean
writing, and later discarding, four sets of requirements no one asked
for. Once P1–P5 above is resolved (even partially — "P1 + P3, revisit
P2 later" is a valid answer), fill in:

- **Scenario:** one-paragraph description of a concrete session of use.
- **Preconditions:** what must already be true/installed.
- **Steps:** what the user/operator actually does.
- **Success:** what observably correct looks like.
- **Linked requirements:** `FR-xx` / `NFR-xx` IDs from Documents 3–4 that this scenario depends on.

## 3. Non-candidates (explicitly not being considered)

- Multi-architecture support beyond the single confirmed target
  (`docs/phase1` D1) — not a use case under consideration at all right
  now.
- Multi-user, multi-tenant, or any shared/hosted deployment model —
  nothing in this project's conversation history suggests this, so it's
  not listed as a candidate above. If wrong, add it here rather than
  assuming it's implicitly covered by P2/P3.

## 4. Decision needed

**Which of P1–P5 (any combination) is OWN-OS actually for?** Tracked as
the top entry in [08-open-questions-and-decision-log.md](./08-open-questions-and-decision-log.md).
Recommendation if a quick default is wanted rather than a full
deliberation: confirm **P1 (builder) + P5 (reference artifact)** as the
*immediate* target — i.e., Phase 1's own finish line is the whole
current goal, nothing past it is committed to yet — and treat P2/P3/P4
as explicitly deferred rather than rejected. This matches the
"Phase 1 is deliberately scoped to just reaching a login prompt"
framing already established in `docs/phase1/01-overview-and-methodology.md`
§1.7, and avoids over-committing this PRD to a direction that hasn't
actually been decided.
