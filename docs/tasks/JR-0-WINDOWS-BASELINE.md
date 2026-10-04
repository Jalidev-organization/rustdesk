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

## Execution record (2026-10-04)

- State: investigating; no successful Windows artifact or smoke test yet.
- Tracking issue: https://github.com/Jalidev-organization/rustdesk/issues/2
- Working branch: `jr/2-windows-baseline`; PR base must be `jali-remote/foundation`.
- Source baseline: `ee7dfd2fb514fa86e8b5a234396c38ffb24b11d9`.
- Recursive submodule: `libs/hbb_common` at `229b904508364c8997aad0fb5af57effac859f60`.
- Receipt-driven review: off, decided by default; no review approval claimed.
- Runtime/product source modifications: none.

### Inspected sources

`AGENTS.md`, `docs/JALI_REMOTE_PLAN.md`, `docs/JALI_REMOTE_AGENT_GUIDE.md`, this task,
`README.md`, `.github/workflows/flutter-build.yml` (x64 Windows job),
`.github/workflows/bridge.yml`, `build.py`, `vcpkg.json`, `Cargo.lock`,
`flutter/pubspec.yaml`, `flutter/windows/CMakeLists.txt`, `.gitignore`,
`src/lib.rs`, `flutter/lib/models/platform_model.dart`.
The official Windows guide at https://rustdesk.com/docs/en/dev/build/windows/
and README describe the deprecated Sciter build. The inherited Flutter workflow
is authoritative for this modern-UI baseline.

### Toolchain inventory and isolated preparation

| Component | Revision/version and result |
| --- | --- |
| OS | Windows 11 Pro x64, 10.0.26200.9457 |
| Visual Studio | Build Tools 2022 17.14.37516.0 (17.14.37); Flutter doctor validates Windows C++ tools |
| MSVC | 14.44.35207, detected by vcpkg |
| Windows SDK | 10.0.26100.0 |
| Python | 3.12.10 |
| Git | 2.55.0.windows.4 |
| Existing Rust | rustc 1.97.1, cargo 1.97.1; global default retained |
| Baseline Rust | rustc 1.75.0 (82e1608df), additionally installed through rustup |
| Flutter | 3.24.5, framework dec2ee5c1f98f8e84a7d5380c05eb8a3d0a81668 |
| Dart | 3.5.4 |
| vcpkg | 9e593bb18ea69cc5095e012465dcd675a822ed0d, executable 2026-07-27-98d7cb0 |
| CMake | Existing 4.4.2; workflow declares VCPKG_CMAKE_VERSION=4.3.0 |
| LLVM | Workflow requires 15.0.6; not installed/verified yet |
| Bridge | Workflow uses cargo-expand 1.0.95 and flutter_rust_bridge_codegen 1.80.1; not generated yet |

Isolated local paths (not required on another machine):
checkout `C:/Users/Jali-dev/Documents/rustdesk-jr0`, SDKs/cache
`C:/Users/Jali-dev/Documents/jr0-tools`, logs `C:/Users/Jali-dev/Documents/jr0-logs`.
No system PATH or Rust default was changed. Initial free C: capacity was 137.65 GB.
The clean native dependency build requires downloads, CPU and disk space; no
reliable total duration/size estimate was available before launch.

### Commands and outcomes to date

```powershell
# From the parent directory, then the isolated checkout:
git clone --branch jali-remote/foundation https://github.com/Jalidev-organization/rustdesk.git rustdesk-jr0
git switch -c jr/2-windows-baseline
git submodule update --init --recursive
rustup toolchain install 1.75.0 --profile minimal
# Outside the checkout:
git clone --depth 1 --branch 3.24.5 https://github.com/flutter/flutter.git <tools>/flutter
flutter --version
flutter doctor -v
flutter precache --windows
git -C <tools>/flutter apply <repo>/.github/patches/flutter_3.24.4_dropdown_menu_enableFilter.diff
git clone https://github.com/microsoft/vcpkg.git <tools>/vcpkg
git -C <tools>/vcpkg checkout 9e593bb18ea69cc5095e012465dcd675a822ed0d
<tools>/vcpkg/bootstrap-vcpkg.bat -disableMetrics
```

All above completed successfully. Flutter doctor validates Visual Studio, Windows
and network resources; Android is absent and intentionally out of scope. The
detached historical Flutter SDK reports an unknown-channel warning, not a
Windows prerequisite failure. Upstream's dropdown patch was applied to the
isolated SDK, not the application repository.

Downloaded the workflow's custom engine from
`https://github.com/rustdesk/engine/releases/download/main/windows-x64-release.zip`,
SHA256 `EC8CABF36EE4FF24C8D98DE25B00E70781EB03876265AEE84D0FE554A110036E`,
and copied its contents into the isolated SDK's
`bin/cache/artifacts/engine/windows-x64-release/`, as the inherited workflow does.
The release URL is mutable; record/compare the hash on any later download.

Launched manifest dependency installation from repository root:

```powershell
$env:VCPKG_ROOT='<tools>/vcpkg'
$env:VCPKG_DEFAULT_HOST_TRIPLET='x64-windows-static'
$env:VCPKG_BINARY_SOURCES='clear;files,<tools>/vcpkg-cache,readwrite'
& "$env:VCPKG_ROOT/vcpkg.exe" install --triplet x64-windows-static --x-install-root="$env:VCPKG_ROOT/installed"
```

vcpkg detected MSVC and scheduled 16 packages, including the repository's aom,
FFmpeg, libvpx, libyuv, opus and mfx-dispatch overlay ports. No result yet.

Launched the inherited Windows build entry point with the pinned Rust:

```powershell
$env:RUSTUP_TOOLCHAIN='1.75.0'
$env:PATH='<tools>/flutter/bin;'+$env:PATH
python build.py --portable --flutter --skip-portable-pack --hwcodec --vram
```

This first probe starts with virtual-display Cargo compilation and currently
fetches the locked Git dependencies. It does NOT certify full prerequisite
readiness: vcpkg was still building and LLVM/bridge preparation remained pending.
Do not interpret a later missing-prerequisite failure as an application defect.

### Smoke-test checklist (not executed)

1. Confirm `flutter/build/windows/x64/runner/Release/rustdesk.exe` and its bundled
   DLL/data files exist; record executable SHA256.
2. Launch the locally built executable and confirm a visible, responsive window
   with no fatal startup error. Process existence alone is not a UI pass.
3. Close and relaunch it; inspect application logs for startup failures.
4. With two authorized Windows PCs on different networks, verify a normal
   remote-control session and record connection/quality observations.

No executable path/hash or smoke success is claimed until these checks run.
Two-PC remote-control and latency evidence remain pending even if local launch
later succeeds. Do not advance to JR-1 or rename the product.

### Current hypothesis and exact next step

No repository-caused blocker has been established yet. Complete the first
pinned-toolchain probe and vcpkg installation, preserve their exact exit/error,
then install/verify isolated LLVM 15.0.6 and generate the bridge according to
`.github/workflows/bridge.yml` before the complete Windows build attempt.
Only diagnose a concrete failure; do not change dependency pins speculatively.

Regression surface: this execution record only. No existing application runtime
path changed. Rollback boundary: remove this execution record without touching
product source, SDK caches or other work. Documentation verification: structural
readback and `git diff --check`; runtime verification is pending, not applicable
to this passive documentation unit.
