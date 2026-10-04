# Jali Remote — Foundation Plan

## Goal

Build **Jali Remote** as a Windows-first remote-support application derived from the RustDesk codebase, optimized for reliable low-latency support sessions between PCs on different networks in Peru.

The first milestone is intentionally narrow: **Windows -> Windows** remote control with stable video, keyboard, mouse, clipboard, file transfer, fullscreen, unattended access, reconnect behavior, and a self-hosted rendezvous/relay path.

## Product boundaries for v0

### In scope
- Windows host and Windows controller.
- Direct peer-to-peer connection whenever possible.
- Relay fallback when direct traversal fails.
- Self-hosted rendezvous/relay infrastructure.
- Remote screen, keyboard and mouse.
- Clipboard sync.
- File transfer.
- Fullscreen.
- Unattended access.
- Basic connection quality diagnostics.
- Jali Remote branding after the technical baseline is proven.

### Explicitly out of scope for the first milestone
- Integration into Calisa.
- Android, iOS, macOS or Linux productization.
- New remote-desktop protocol.
- Replacing RustDesk codecs or transport before measurement.
- Large UI redesign.
- Account/billing system.
- Fleet management.
- Custom relay protocol.
- Broad refactors of RustDesk internals.

## Architectural rule

**Preserve RustDesk's proven data path until measurement proves a specific bottleneck.**

Do not rewrite capture, codec, transport, input, rendezvous or relay code simply because a cleaner design is possible.

The initial architecture remains:

1. Controller asks rendezvous infrastructure to locate the remote peer.
2. Attempt direct connection.
3. Fall back to relay when direct connectivity is unavailable or unsuitable.
4. Capture/encode on the host.
5. Transport video/audio toward the controller.
6. Send input/control events in the reverse direction.

## Existing code areas that matter

From the inherited RustDesk repository:

- `src/server/` — audio, clipboard, input, video and network services.
- `src/rendezvous_mediator.rs` — rendezvous path.
- `src/platform/` — platform-specific behavior.
- `libs/scrap/` — screen capture.
- `libs/enigo/` — input control.
- `libs/clipboard/` — clipboard.
- `libs/base/src/fs.rs` — file transfer.
- `flutter/` — current user interface.
- `libs/hbb_common/` — shared protocol/config code and a Git submodule.

The repository's existing `AGENTS.md` remains authoritative for coding hygiene.

## License guardrail

The inherited repository contains GNU AGPL v3 licensing. Jali Remote must retain required notices and source-availability obligations when modified versions are distributed.

Before any commercial/public distribution, review:
- the parent project license,
- third-party dependency licenses,
- the `libs/hbb_common` submodule,
- branding/trademark implications.

This document is an engineering guardrail, not legal advice.

## Milestones

### JR-0 — Baseline and reproducibility
Goal: prove that the fork builds and runs before meaningful modification.

Deliverables:
- reproducible Windows development/build instructions,
- exact toolchain versions,
- successful upstream-equivalent Windows build,
- baseline connection test between two Windows PCs on different networks,
- recorded baseline latency/quality observations,
- no product behavior changes.

Exit criterion:
A clean fork build can establish a normal remote-control session without Jali-specific code.

### JR-1 — Personal-PC self-hosted connectivity
Goal: remove dependence on public infrastructure for our tests while keeping the first deployment personal and minimal.

Initial deployment decision:
- Héctor's own Windows PC is the **first-choice host** for `hbbs` (ID/rendezvous) and `hbbr` (relay).
- Do **not** purchase or provision Hetzner/VPS infrastructure for v0 unless testing proves the home/office connection cannot reliably expose the required services (for example because of CGNAT or an equivalent network limitation).
- The server only needs to run when Héctor needs Jali Remote for support; 24/7 production availability is not a v0 requirement.
- Preserve RustDesk's direct P2P attempt. The relay is a fallback, not a reason to force all session traffic through the server.

Deliverables:
- run `hbbs` + `hbbr` on Héctor's Windows PC,
- verify public reachability / router / firewall prerequisites,
- detect and document CGNAT or other inbound-connectivity blockers if present,
- documented ports and startup procedure,
- client configuration procedure,
- direct-vs-relay verification,
- reconnection test.

Exit criterion:
Two Windows PCs on unrelated internet connections can connect using Héctor's PC as the Jali-controlled rendezvous/relay host. External VPS infrastructure is considered only if this test demonstrates a real need.

### JR-2 — Windows support baseline
Goal: validate all functions needed for real customer support.

Required test matrix:
- mouse,
- keyboard,
- fullscreen,
- clipboard text copy/paste in both directions,
- image/clipboard behavior where supported,
- file transfer,
- Windows-to-Windows file copy/paste,
- Windows-to-Windows drag-and-drop of files (explicitly verify the actual UX; do not assume support),
- unattended access,
- UAC/elevation and Secure Desktop behavior during software installation,
- ability to interact with authorized administrative prompts when Jali Remote is correctly installed/elevated,
- restart/reconnect and return after Windows reboot,
- Windows login/lock-screen support where the inherited implementation permits it,
- fullscreen operation suitable for working as though seated at the remote PC,
- multiple displays if already supported without new work.

Exit criterion:
The application is usable for a real support session.

### JR-3 — Measurement before optimization
Goal: locate actual bottlenecks instead of guessing.

Measure:
- connection setup time,
- round-trip input latency,
- direct vs relay behavior,
- capture time,
- encode time,
- decode/present time,
- effective FPS,
- bitrate,
- CPU/GPU utilization,
- packet loss/jitter where observable.

Exit criterion:
Every proposed performance change is tied to an observed bottleneck.

### JR-4 — Jali Remote identity
Only after JR-0 through JR-3 are stable:
- product name,
- executable/application labels,
- icon and assets,
- minimal simplified home screen,
- support-oriented defaults.

Do not do broad string replacement across the repository.

### JR-5 — Targeted performance work
Only address measured problems.

Potential areas, if data justifies them:
- capture path,
- hardware encoding,
- bitrate/quality adaptation,
- relay selection,
- reconnection,
- responsiveness while the screen is changing,
- high-quality recovery when the screen becomes static.

### JR-6 — Support workflow integration
Future phase:
- known customer/device identity,
- support-request workflow,
- launch remote session from a support panel,
- optional Calisa integration.

This phase must not contaminate the remote-control core.

## Branching

- `master`: keep close to the fork baseline/upstream until we intentionally decide otherwise.
- `jali-remote/foundation`: planning and project-specific foundation.
- Feature work: `jr/<issue>-short-name`.

Never bundle unrelated tasks into one branch.

## Change policy

1. One issue = one narrow objective.
2. Prefer additive changes and thin hooks.
3. No speculative refactors.
4. Do not optimize without measurements.
5. Every performance PR must state the baseline and the measured result.
6. Every PR must list regression surface.
7. Preserve upstream behavior when the Jali-specific feature is disabled.
8. Avoid editing `libs/hbb_common` unless the task truly requires protocol/shared-config changes.

## Model allocation

Use expensive reasoning only where it changes outcomes.

### GPT-6 Luna
Use for:
- repository navigation,
- locating strings/files,
- straightforward documentation,
- mechanical low-risk edits,
- summarizing logs.

### GPT-6.1 Sol — Medium
Default implementation model.

Use for:
- contained Rust changes,
- Flutter changes,
- build fixes,
- CI work,
- tests,
- normal Windows-specific implementation.

### GPT-6.1 Sol — High
Use for:
- capture pipeline,
- input/UAC edge cases,
- rendezvous/relay logic,
- performance diagnosis,
- concurrency,
- non-trivial Rust ownership/async problems.

### GPT-6.1 Sol — Extra High
Use only after High fails or when a bug spans several subsystems and evidence is already collected.

### GPT-6 Astra
Reserve for:
- architectural deadlocks,
- exceptionally difficult cross-subsystem bugs,
- reviewing a high-risk performance/networking plan before implementation,
- cases where Sol has produced two serious but unsuccessful attempts.

Do not use Astra for repository exploration, planning boilerplate, branding, simple UI work or routine build fixes.

## Handoff rule for agents

Every agent task must end with:
- what was changed,
- files changed,
- tests/builds run,
- result,
- remaining uncertainty,
- regression surface,
- commit SHA,
- next recommended task.

If token/session limits are near, save progress in the branch and update the issue before doing more exploration.
