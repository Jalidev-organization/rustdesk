# Jali Remote — Project Charter

Status: bootstrap
Upstream: rustdesk/rustdesk
Fork: Jalidev-organization/rustdesk
Baseline upstream commit: e5bc204fe4dacc4db9c3cdb1f1338813986c89e7
Working branch: jali/main

## Goal

Build a personal/support remote-desktop application for reliable Windows-to-Windows support over the Internet, initially focused on use in Peru. Preserve RustDesk's proven low-latency remote-control foundation rather than rewriting capture, input, NAT traversal, codecs, or relay logic prematurely.

## Product name

Jali Remote (working product name).

## V1 scope

- Windows host and Windows controller.
- Remote screen viewing and control.
- Keyboard and mouse input.
- Clipboard.
- File transfer.
- Full-screen operation.
- Unattended access with explicit configuration and strong authentication.
- Direct/P2P connection when possible, relay fallback when required.
- Self-hosted infrastructure target.
- Adaptive image quality and hardware codec support where the inherited implementation supports it.

## Explicitly out of scope for V1

- Calisa integration.
- Mobile clients.
- macOS/Linux product support.
- Rewriting RustDesk networking or codec stack.
- Enterprise management features.
- New proprietary remote-control protocol.
- Removing attribution/license notices required by upstream licenses.

## Engineering principles

1. Measure before optimizing.
2. Preserve upstream behavior until a Jali requirement proves a change is needed.
3. Keep upstream synchronization possible.
4. Small reviewable changes; no giant rebrand commit mixed with networking changes.
5. Security-sensitive changes require explicit review.
6. Never weaken authentication, encryption, consent, or authorization merely to simplify support.
7. Maintain a decision log so another AI agent can resume work without rediscovery.

## Baseline observations

- Main Rust package version at bootstrap: 1.5.0.
- Upstream repository license file is GNU AGPL v3.
- Repository uses the hbb_common git submodule.
- Cargo features include hwcodec; hbb_common is built with WebRTC support.
- Windows-specific dependencies already exist.
- Before public/commercial distribution, perform a dependency/submodule license review and comply with all applicable notices/source obligations.

## Work stages

### JR-00 — Baseline and reproducibility
Document exact upstream baseline, build requirements, Windows build path, CI status, submodules, and dependency/license inventory. Produce an untouched Windows build before product modifications.

Exit criterion: reproducible Windows artifact from the fork and recorded build instructions.

### JR-01 — Product isolation and branding
Inventory every RustDesk-facing product identifier. Define what can safely be renamed without breaking protocol/config/update behavior. Apply Jali Remote branding in small commits.

Exit criterion: Jali Remote launches as a distinct test application while core remote connectivity remains unchanged.

### JR-02 — Self-hosted connectivity lab
Deploy/test rendezvous and relay infrastructure separately. Test direct vs relayed sessions across different networks and collect latency, bitrate, FPS, CPU/GPU and reconnect observations.

Exit criterion: stable Windows-to-Windows session across two unrelated Internet connections.

### JR-03 — Support-focused UX
Reduce UI to the support workflow without deleting underlying capabilities prematurely. Preserve ID/password or equivalent secure connection flow, fullscreen, clipboard and file transfer.

### JR-04 — Performance validation
Benchmark default inherited behavior first. Then optimize only measured bottlenecks: capture, encode/decode, transport, rendering or input latency.

### JR-05 — Security and unattended access
Review credential storage, authorization, unattended access, service privileges, logging and update path. No silent access mechanisms.

### JR-06 — Packaging and pilot
Produce signed/reproducible Windows packaging plan, upgrade/uninstall behavior and a controlled two-machine pilot.

### JR-07 — Optional Calisa integration
Only after the standalone remote-support product is stable.

## AI/model allocation policy

Do not spend the strongest model on repository discovery or mechanical edits.

- Repository inventory, searches, docs, simple rename mapping: economical/fast coding model.
- Normal implementation with clear acceptance criteria: GPT-6.1 Sol, medium effort.
- Cross-cutting Rust/Flutter/Windows changes or non-obvious failures: GPT-6.1 Sol, high effort.
- Networking/codec/performance investigation after measurements: GPT-6.1 Sol, high; escalate only if blocked.
- GPT-6 Astra: reserve for unresolved architectural/performance/security problems after a cheaper model has produced evidence and a concise handoff.
- Highest effort: only for a specific blocker, never for general exploration.

## Agent handoff rule

Every substantial agent task must leave:
- changed files,
- commands/tests executed,
- result,
- unresolved risks,
- next recommended action.

Do not let an agent perform broad autonomous refactors without an explicit task boundary.
