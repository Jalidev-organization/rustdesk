# JR-0 — Windows baseline build

## Recommended model
**GPT-6.1 Sol — Medium**

Do not use Astra for this task. Escalate to GPT-6.1 Sol High only if the build fails for a non-obvious repository/toolchain reason after one disciplined attempt.

## Objective
Prove that the fork builds and launches on Windows without changing Jali Remote product behavior.

## Agent prompt

You are working on Jali Remote, a Windows-first remote-support product based on this RustDesk fork.

Before doing anything:
1. Read `AGENTS.md`.
2. Read `docs/JALI_REMOTE_PLAN.md`.
3. Read `docs/JALI_REMOTE_AGENT_GUIDE.md`.

Your task is JR-0 only: establish a reproducible Windows baseline build of the current fork.

Requirements:
- Inspect repository-native build documentation and GitHub workflows before choosing commands.
- Determine exact prerequisites/toolchain versions actually needed by this revision.
- Attempt the supported Windows build path.
- Do not rename RustDesk/Jali Remote yet.
- Do not refactor.
- Do not optimize.
- Do not modify networking, capture, codecs, relay/rendezvous, or product behavior.
- If a build problem exists, diagnose the smallest actual cause. Do not “fix” unrelated warnings.
- Any code/config change must be strictly necessary to get this fork building and must be isolated.
- Record exact commands and results.
- Produce a minimal smoke-test checklist for launching the resulting executable.

Continuity rule:
If your model/session quota is getting low, stop exploration early enough to preserve state. Commit coherent progress and update `docs/tasks/JR-0-WINDOWS-BASELINE.md` with:
- current status,
- hypothesis/blocker,
- files inspected,
- commands run,
- exact error if any,
- next exact step.

Never leave the only useful analysis in chat.

Completion report:
### Result
### Files changed
### Verification
### Regression surface
### Known limits
### Commit
### Next

The task is complete only when either:
A) a Windows build succeeds and its artifact path plus smoke test are documented, or
B) a reproducible blocker is documented with enough evidence for another agent to continue immediately.

## Status
- State: ready
- Baseline branch: `jali-remote/foundation`
- Upstream-derived default branch: `master`
- Product behavior changes allowed: **none**
