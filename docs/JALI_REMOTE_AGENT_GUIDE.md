# Jali Remote — Agent Operating Guide

This file adds project-specific rules on top of the inherited `AGENTS.md`.

## Mission

Jali Remote is not a clean-room rewrite. It is a focused Windows-first remote-support product derived from this RustDesk fork.

The priority order is:

1. reliability,
2. low interaction latency,
3. visual clarity,
4. reconnection quality,
5. simplicity for support use,
6. branding,
7. new features.

## Before coding

For every task:

1. Read `AGENTS.md`.
2. Read `docs/JALI_REMOTE_PLAN.md`.
3. Identify the smallest runtime path involved.
4. Verify whether upstream already implements the requested behavior.
5. Do not modify code until the current behavior is understood.
6. If the task is performance-related, collect or identify baseline evidence first.

## Forbidden patterns

Do not:
- redesign the protocol preemptively,
- replace codecs preemptively,
- replace working P2P/relay machinery preemptively,
- make repository-wide branding substitutions,
- mix a feature with cleanup/refactoring,
- edit unrelated platforms for a Windows-only goal,
- change `libs/hbb_common` just to avoid a small amount of client-side duplication,
- claim a performance win without before/after evidence.

## Windows-first rule

Unless an issue explicitly says otherwise:
- optimize for Windows -> Windows,
- keep changes behind platform/capability boundaries when appropriate,
- do not spend task scope repairing other platforms.

## Performance task template

A performance task is incomplete without:
- reproduction,
- baseline numbers or observable timings,
- suspected stage,
- instrumentation if necessary,
- smallest targeted change,
- before/after comparison,
- CPU/GPU/network tradeoff,
- direct and relay checks when relevant.

## Session/token continuity

When a coding session may end before completion:

1. Commit coherent progress even if the feature is incomplete.
2. Use a commit message that makes incompleteness explicit.
3. Update the GitHub issue with:
   - current hypothesis,
   - evidence,
   - files inspected/changed,
   - commands run,
   - next exact step.
4. Never leave the only useful analysis inside chat context.

The next model/account must be able to continue from GitHub alone.

## Completion format

Conclude every task with:

### Result
One paragraph.

### Files changed
Exact paths.

### Verification
Commands/tests and outcomes.

### Regression surface
Existing runtime behavior changed and why it was unavoidable.

### Known limits
Only real remaining limits.

### Commit
SHA.

### Next
One concrete next action.
