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
- State: blocked (criterion B; Flutter symlink check now passes, LLVM location remains unverified)
- Baseline branch: `jali-remote/foundation`
- Upstream-derived default branch: `master`
- Product behavior changes allowed: **none**

## Execution record (2026-10-04)

- State: blocked; no successful Windows executable or smoke test. See final outcomes below.
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
| Baseline Rust | rustc 1.75.0 (82e1608df), cargo 1.75.0 (1d8b05cdd), additionally installed through rustup |
| rustup | 1.29.1 (d95a37b6a); rustup self-updated from 1.29.0 during toolchain installation |
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

## Reproducible blockers and completed probe outcomes

This section supersedes the earlier in-progress entries. JR-0 meets reporting
criterion **B**, not build-success criterion A. Do not merge this PR or advance
JR-1 as if the client had been validated.

| Operation | Observed result |
| --- | --- |
| `python build.py --portable --flutter --skip-portable-pack --hwcodec --vram` with Rust 1.75 | Python exit `-1`; native hwcodec custom build exits `101`, missing libclang |
| Virtual-display sub-build inside that command | Release build succeeded in 5m 16s; auxiliary DLL only, not a runnable RustDesk client |
| `flutter pub get` with Flutter 3.24.5 | Exit `1`, Windows plugin symlink prerequisite |
| `flutter build windows --release` with Flutter 3.24.5 | Exit `1`, same symlink prerequisite reproduced independently |
| LLVM 15.0.6 download plus silent isolated installation | Tool policy rejected the command before execution; not installed, no workaround attempted |
| vcpkg manifest dependency installation | Exit `0`, all 16 packages installed successfully in 19 min |

### Blocker 1: missing LLVM/libclang

The inherited workflow explicitly installs LLVM 15.0.6. The actual Rust build
fails in `hwcodec v0.7.1` (`778df1f9`), using `bindgen 0.59.2`:

```text
error: failed to run custom build command for `hwcodec v0.7.1`
process didn't exit successfully: build-script-build (exit code: 101)
Unable to find libclang: "couldn't find any valid shared libraries matching:
['clang.dll', 'libclang.dll'], set the `LIBCLANG_PATH` environment variable to a
path where one of these files can be found (invalid: [])"
Error occurred when executing:
`cargo build --locked --features hwcodec,vram,flutter --lib --release`. Exiting.
```

No codec/source fix is warranted: this is the missing native prerequisite. The
official installer URL is
`https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.6/LLVM-15.0.6-win64.exe`,
asset size 290,951,930 bytes confirmed through the LLVM GitHub release API.
The combined download/`Start-Process` silent installation command was rejected
as `blocked by policy`, before execution. Do not retry through another launcher
to bypass that denial. Install the prerequisite through a permitted process,
then verify `clang --version` reports 15.0.6 and `libclang.dll` exists before
setting process-local `LIBCLANG_PATH`.

### Blocker 2: Windows plugin symlink permission

Both Flutter commands above emit exactly:

```text
Building with plugins requires symlink support.

Please enable Developer Mode in your system settings. Run
  start ms-settings:developers
to open settings.
```

The current process is not administrator (`IS_ADMIN=False`). No Developer Mode
or global security setting was changed. The next operator must explicitly
resolve the Windows symlink prerequisite. Do not disable security checks or
change application/plugin code to hide it.

`flutter pub get` downloaded dependencies and automatically changed 16 lockfile
entries to match the pinned Flutter SDK before failing. Its diff was saved to
`<logs>/pubspec-lock-generated.diff`, then only that generated modification was
restored. No dependency upgrade or regenerated lockfile belongs in this PR.

### Environment caveat: inherited Cargo output directory

The probe inherited `CARGO_TARGET_DIR=C:/h`. The successful auxiliary DLL is:

- Path: `C:/h/release/dylib_virtual_display.dll` (also in `release/deps/`).
- Size: 309,760 bytes.
- SHA256: `BDA82AB0A05A7408672B219DF7AC539D03DC6368FFDC3868897661D6385BB16D`.

This is **not** `rustdesk.exe`, not a completed application, and was not launched.
No client executable SHA256 or smoke-test pass exists.

`build.py` checks/copies files from repository-relative `target/release`, and
Flutter Windows CMake installs `../../target/release/librustdesk.dll`. Therefore
remove the inherited target-directory variable **in the next child shell only**:
`Remove-Item Env:CARGO_TARGET_DIR -ErrorAction SilentlyContinue`. Do not edit
machine-wide environment configuration or delete the shared `C:/h` cache.

### Exact continuation

After the operator resolves symlink permission and installs permitted LLVM
15.0.6, verify the Flutter prerequisite first:

```powershell
$repo='C:/Users/Jali-dev/Documents/rustdesk-jr0'
$tools='C:/Users/Jali-dev/Documents/jr0-tools'
$env:PATH="$tools/flutter/bin;"+$env:PATH
Set-Location "$repo/flutter"
flutter pub get
# Require exit 0, then inspect any generated lockfile delta. Do not commit it.
```

Then prepare the generated bridge following `.github/workflows/bridge.yml`:
`cargo-expand 1.0.95`, `flutter_rust_bridge_codegen 1.80.1 --features uuid`, and
the workflow's Flutter 3.22.3 generation SDK/pubspec adaptation. The generated
files are CI prerequisites, absent from a clean checkout. Do not borrow a bridge
artifact from a different source revision. The workflow's exact generator call is:

```text
flutter_rust_bridge_codegen --rust-input ./src/flutter_ffi.rs --dart-output ./flutter/lib/generated_bridge.dart --c-output ./flutter/macos/Runner/bridge_generated.h
```

Once vcpkg and the matching bridge are ready, the next full attempt is:

```powershell
Set-Location $repo
Remove-Item Env:CARGO_TARGET_DIR -ErrorAction SilentlyContinue
$env:RUSTUP_TOOLCHAIN='1.75.0'
$env:VCPKG_ROOT="$tools/vcpkg"
$env:VCPKG_DEFAULT_HOST_TRIPLET='x64-windows-static'
$env:LIBCLANG_PATH="$tools/llvm-15.0.6/bin" # adjust only to the verified installation
python build.py --portable --flutter --skip-portable-pack --hwcodec --vram
```

The expected completed-client path is
`<repo>/flutter/build/windows/x64/runner/Release/rustdesk.exe`, conditional on
success. Hash it and perform the previously documented smoke checklist; never
claim UI success from the auxiliary DLL or a process-only check.

### Scope and continuity

- Existing runtime paths changed: none.
- Files modified intentionally: this task document only.
- No branding, networking, relay/rendezvous, capture, codec or product changes.
- No migrations. No changes to the foundation branch or its PR #1.
- Local full logs remain outside the versioned tree; bounded errors and exact
  continuation above are sufficient to resume from GitHub alone.
- Tracking PR: https://github.com/Jalidev-organization/rustdesk/pull/6 (draft,
  base `jali-remote/foundation`). Keep issue #2 open until build and smoke evidence.

### Local evidence identifiers

Full UTF-8 text logs are outside the repository. These hashes identify the
completed probe logs; no credentials or raw environment dump is published:

| Log under `<logs>/` | SHA256 |
| --- | --- |
| `windows-build-first.log` | `D5441D622FE55CBD842542F4EDD04E91E14FE11EAF546B2AEE19B7B169AB2424` |
| `flutter-pub-get.log` | `1A7E7A403DBAF78C351F7823D2B156683DF8DFED938EDCD163AD53482BFE2444` |
| `flutter-windows-stage.log` | `8D9A3C7555DDE427C6A67285E63A3072DB71E19BB07F709D1A1B1544F3F6D413` |

### Final native dependency result and verification

The existing vcpkg process finished, without cancellation or a second attempt:
`All requested installations completed successfully in: 19 min`, `VCPKG_EXIT=0`.
`vcpkg list --x-install-root=<tools>/vcpkg/installed` confirmed aom 3.14.1,
FFmpeg 7.1.1 with amf/nvcodec/qsv, libjpeg-turbo 3.2.0, libvpx 1.15.2,
libyuv 1857, mfx-dispatch 1.35.1#5 and opus 1.5.2, plus their native tooling.
Final `vcpkg-install.log` SHA256:
`287C358DE735A6D59CAF671C4BECADAC7543158B179D81564EA7015448D69DC6`.

All processes launched for the build/dependency probes have completed. No
successful client build, bridge generation or GUI launch was performed. The
remaining prerequisites are LLVM/libclang and Windows symlink permission,
followed by matching bridge generation. No repeated full build was attempted
while those known prerequisites remained unavailable.

Final proportional documentation verification: read back the full task document,
checked the complete diff against `jali-remote/foundation`, and ran
`git diff --check`. Only this documentation file belongs to the change; generated
lockfile changes were restored and all build/download logs stay external.
The inherited build warnings were left untouched. No review receipt or runtime
approval is claimed. The PR remains draft and must not be merged as a validated
Windows baseline.
## Resumed prerequisite verification (2026-10-04)

This update supersedes the earlier symlink blocker. Continued the **same**
checkout and `jr/2-windows-baseline` branch at `c091e87a238cd6ee712298d50e29851412f6d8b5`;
no clone, reset or restart. Re-read repository instructions, task, native bridge
workflow, issue #2 and PR #6. Receipt-driven review remains off/default.

- Flutter 3.24.5 `flutter pub get`: **exit 0**. The previous symlink error no longer
  occurs. No global security setting or environment variable was changed.
- The generated `flutter/pubspec.lock` delta was saved externally to
  `<logs>/resume-pubspec-lock-generated.diff`, then only that own generated delta
  was restored. No product/dependency source edit is included.
- `<logs>/resume-flutter-pub-get.log` SHA256:
  `4B4B86E94A474B3B1E7F05D5EE9722372052117D036699963BD8A29F02FFCD67`.
- `vcpkg list --x-install-root=<tools>/vcpkg/installed` confirms the same 16
  previously installed native packages. They were not rebuilt or replaced.
- LLVM 15.0.6 is **not yet located or verified**. This does not establish whether
  the operator installed it somewhere else. `clang` is absent from the current
  PATH; process/user/machine `LIBCLANG_PATH` values are empty. No LLVM entry was
  found in the inspected machine/user uninstall registry locations.

Bounded location search inspected roots `C:/`, `D:/`, `E:/`, `F:/`, `G:/`,
`C:/Program Files`, `C:/Program Files (x86)`, the user's `Downloads`,
`AppData/Local/Programs`, `bin`, and the isolated `jr0-tools` directory. No LLVM
installation/installer candidate or `clang.exe`/`libclang.dll` was identified.
Specifically, neither `C:/Program Files/LLVM/bin/libclang.dll` nor
`<tools>/llvm-15.0.6/bin/libclang.dll` exists. No whole-disk recursive search,
installation retry, security change or policy bypass was performed.

### Current blocking input and exact next step

Obtain the actual LLVM 15.0.6 installation directory from the operator. Then
verify, using that confirmed path rather than a guessed replacement:

```powershell
$llvm='<operator-confirmed LLVM 15.0.6 directory>'
& "$llvm/bin/clang.exe" --version
Test-Path "$llvm/bin/libclang.dll"
$env:LIBCLANG_PATH="$llvm/bin" # process-local only
python -c "import ctypes, os; ctypes.CDLL(os.path.join(os.environ['LIBCLANG_PATH'], 'libclang.dll')); print('LIBCLANG_LOAD_OK')"
```

Require the correct version, DLL existence and usable load before matching bridge
generation and the documented full build command. Neither bridge generation nor
a new full build was launched while this prerequisite remained unverified.
No completed client executable or smoke result is claimed; PR #6 stays draft,
issue #2 stays open, and no merge or JR-1 advance is authorized by this checkpoint.