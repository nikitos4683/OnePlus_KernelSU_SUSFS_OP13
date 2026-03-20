# AGENTS.md

## Purpose

This repository is currently narrowed to a single build target: OnePlus 13 on OOS16. It does not contain the full kernel source tree. Instead, it stores the active device metadata, manifest file, and GitHub Actions logic that:

- fetch upstream kernel sources at build time,
- inject KernelSU or KernelSU-Next,
- optionally integrate SUSFS,
- apply a large patch stack,
- compile on GitHub runners,
- package the result as AnyKernel3 flashable ZIPs,
- optionally create a draft GitHub release.

If you are debugging or extending this repo, always remember that many failures originate outside this repo in upstream manifests, upstream kernel sources, or external dependency repos.

## Repository Summary

- Main audience: maintainers of WildKernels OnePlus releases.
- Main docs: `README.md` and `compatibility.md`.
- Active config inventory at the time of writing: `configs/oos16/OP13.json`.
- Active local manifest: `manifests/oos16/oneplus_13_w.xml`.
- Total active device configs: 1

## Top-Level Layout

- `configs/`: the active device build metadata lives in `configs/oos16/OP13.json`. Legacy OS folders may still exist as empty directories after cleanup.
- `manifests/`: the active local XML manifest is `manifests/oos16/oneplus_13_w.xml`. Legacy OS folders may still exist as empty directories after cleanup.
- `.github/workflows/build-kernel-release.yml`: main entry point, manual-only workflow, matrix generation, optional release creation.
- `.github/actions/action.yml`: composite action with the real build logic, validation, sync, patching, build, validation, and packaging.
- `README.md`: project overview, advertised features, install pointers, credits, and support links.
- `compatibility.md`: device compatibility notes and flashing caveats.

## What Is In This Repo vs. Outside This Repo

In this repo:

- Device definitions and feature toggles.
- Manifest files.
- Build pipeline logic.
- Release-note generation logic.

Not in this repo:

- The full OnePlus kernel source trees.
- The `kernel_manifest` upstream repository contents.
- The external patch repositories cloned during CI.
- Local developer helper scripts for building.

This means local inspection can explain orchestration and patch intent, but not every compile or patch failure.

## Config Schema

Each file in `configs/**/*.json` currently uses this key set:

- `model`
- `soc`
- `branch`
- `manifest`
- `android_version`
- `kernel_version`
- `os_version`
- `lto`
- `rust_build`
- `disk_cleanup`
- `hmbird`
- `susfs`
- `bbg`
- `bbr`
- `ttl`
- `ip_set`
- `wireguard`
- `unicode`
- `optimization_patches`
- `uname`

Field meaning:

- `model`: device identifier used throughout logs, artifacts, and release notes. Validation expects it to start with `OP`.
- `soc`: SoC family, for example `sun`.
- `branch`: upstream manifest branch or a `wild/*` branch convention used by this repo.
- `manifest`: XML manifest filename or HTTPS URL.
- `android_version`: Android generation, for example `android15`.
- `kernel_version`: kernel family in `X.Y` form, for example `6.6`.
- `os_version`: OOS generation, for example `OOS16`.
- `lto`: one of `none`, `thin`, or `full`.
- `rust_build`: whether bindgen/rust tooling is needed.
- `disk_cleanup`: whether the CI runner should free extra disk space before build.
- `hmbird`, `susfs`, `bbg`, `bbr`, `ttl`, `ip_set`, `wireguard`, `unicode`, `optimization_patches`: feature toggles that influence patching and config mutation.
- `uname`: custom local version / branding string.

Example observed config:

```json
{
  "model": "OP13",
  "soc": "sun",
  "branch": "wild/sm8750",
  "manifest": "oneplus_13_w.xml",
  "android_version": "android15",
  "kernel_version": "6.6",
  "os_version": "OOS16",
  "lto": "thin",
  "rust_build": false,
  "disk_cleanup": false,
  "hmbird": true,
  "susfs": true,
  "bbg": false,
  "bbr": true,
  "ttl": true,
  "ip_set": true,
  "wireguard": true,
  "unicode": false,
  "optimization_patches": false,
  "uname": "OP-WILD"
}
```

## Manifest Conventions

- The active local manifest is `manifests/oos16/oneplus_13_w.xml`.
- Legacy `manifests/oos14` and `manifests/oos15` directories may remain, but they no longer contain active XML files after the single-target cleanup.
- Example contents point to CodeLinaro remotes for common build pieces, OnePlusOSS repositories for device/common kernel trees, and pinned prebuilts for kernel build tools and clang.

Important behavior in the composite action:

- If `manifest` is an HTTPS URL, the workflow downloads it into `.repo/manifests/temp_manifest.xml`.
- If `branch` matches `wild/*`, the workflow copies the manifest from this repo under `manifests/<os_version-lowercase>/`.
- Otherwise it uses the configured `branch` and `manifest` directly against `https://github.com/OnePlusOSS/kernel_manifest.git`.

That `wild/*` behavior is easy to miss and matters when adding or editing device support.

## Main Workflow: build-kernel-release.yml

This is the main user-triggered build entry point.

Key inputs:

- `make_release`: create a draft release after successful builds.
- `ksu_options`: JSON array describing KSU variants to build.
- `optimize_level`: `O2` or `O3`.
- `clean_build`: disables ccache usage.
- `android15-6_6_susfs_branch_or_commit`

Important workflow behavior:

- Builds are manual only. There is no push-triggered validation in the main workflow.
- The matrix is built from every JSON file under `configs/`.
- The matrix is then explicitly filtered to `model == "OP13"` and `os_version == "OOS16"`.
- `ksu_options` is normalized with `jq`.
- If a KSU entry omits `hash`, defaults are `KSUN -> dev` and `KSU -> main`.
- Each device config is multiplied by each KSU option, producing one matrix row per combination.
- The workflow input surface is intentionally reduced to the OnePlus 13 OOS16 path only.

## Composite Action: action.yml

The composite action is the real system of record for build behavior.

High-level flow:

1. Parse `op_config_json` and export it into environment variables.
2. Validate every important field.
3. Optionally install `bindgen` if `rust_build` is enabled.
4. Install or locate the `repo` tool.
5. Initialize and sync the external kernel source tree.
6. Set directory paths for artifacts and dependencies.
7. Read the real kernel version from synced source files.
8. Detect clang and optional rust toolchains from prebuilts.
9. Configure ccache unless clean mode is requested.
10. Clone external helper repos.
11. Add KernelSU or KernelSU-Next.
12. Apply SUSFS and other patch stacks.
13. Generate and mutate `out/.config`.
14. Compile the kernel.
15. Validate the built `Image`.
16. Package the kernel into AnyKernel3 ZIP format.
17. Upload the ZIP as a workflow artifact.

## External Dependencies Cloned at Build Time

Observed clone targets include:

- `https://github.com/TheWildJames/AnyKernel3.git`
- `https://github.com/TheWildJames/kernel_patches.git`
- `https://gitlab.com/simonpunk/susfs4ksu.git`

KernelSU integration is fetched via remote setup scripts:

- KernelSU-Next setup script from `KernelSU-Next/KernelSU-Next`
- KernelSU setup script from `tiann/KernelSU`

The repo tool itself is fetched from:

- `https://storage.googleapis.com/git-repo-downloads/repo`

This repo is therefore highly dependent on external network availability and upstream stability.

## Patch and Feature Logic

The composite action applies a wide patch stack. Based on the current action, categories include:

- SUSFS enablement and SUSFS compatibility fixes
- Baseband Guard related patches when `bbg` is enabled
- manual hooks compatibility patch
- ptrace leak fix for older kernels
- HMBIRD patches for selected OnePlus devices
- native WireGuard config enablement when `wireguard` is enabled
- optional general optimization patches gated by `optimization_patches`
- Unicode bypass fix when `unicode` is enabled
- IPv6 NAT related patching

Many patch applications use `patch -p1 --forward` with fuzz or tolerant behavior. That is practical for maintenance, but it also means:

- upstream source changes can silently increase fragility,
- a patch may partially stop applying after an upstream change,
- build breakage may appear suddenly without any change in this repo.

Treat `.github/actions/action.yml` as high-blast-radius code.

## Build Output and Validation

Build steps currently revolve around:

- `make O=out gki_defconfig`
- `.config` mutation with `scripts/config`
- `make O=out olddefconfig`
- parallel `make`

Image validation includes:

- file existence check,
- ARM64 file format check,
- minimum image size check,
- warning count extraction,
- SHA256 generation,
- build-time reporting.

Observed minimum image size threshold:

- `6102400` bytes

Artifact naming:

- Without SUSFS: `AK3_<model>_<os>_<kernel-full>_<ksu-type>_<ksu-version>.zip`
- With SUSFS: `AK3_<model>_<os>_<kernel-full>_<ksu-type>_<ksu-version>_SuSFS_<susfs-version>.zip`

Packaging uses AnyKernel3 and places the built `Image` inside the AnyKernel3 tree before zipping.

## Release Flow

If `make_release` is enabled:

- the workflow creates a new tag based on `SUSFS_BASE_VERSION`,
- downloads artifacts,
- generates release notes dynamically,
- creates a draft GitHub release,
- uploads ZIP artifacts to that release.

Release notes are generated from built artifacts plus the matrix metadata, not from hand-written markdown.

## User-Facing Project Claims

The README currently advertises support or integration for:

- KernelSU
- KernelSU-Next
- WildKSU Manager support
- SUSFS
- optional BBG
- HMBIRD SCX
- BBRv1
- LTO
- optional optimization patches
- TTL target support
- native WireGuard support
- IP Set and IPv6 NAT support
- TMPFS XATTR and POSIX ACL support
- optional Unicode bypass fix

Keep README claims aligned with actual config flags and action behavior.

## Compatibility Notes From Docs

`compatibility.md` currently states:

- kernels are expected to work on stock ROMs,
- current kernels are built from Android 15 manifests,
- users should not reuse old ZIPs after a major Android OTA unless compatibility is confirmed,
- non-OnePlus devices may work if kernel/KMI compatibility matches and verification-related protections are handled by the user.

If you change manifests, supported devices, or kernel bases, update docs accordingly.

## Common Maintenance Tasks

### Add or Update Device Support

- Add or edit the JSON config in the correct `configs/oosXX/` directory.
- Add or edit the matching XML manifest in `manifests/oosXX/` if using the `wild/*` flow.
- Make sure `branch`, `manifest`, `android_version`, `kernel_version`, and `os_version` match the intended source base.
- Double-check feature toggles to avoid unintentionally enabling patches on unsupported devices.

### Change Build Features Across Devices

- For per-device feature defaults, edit JSON config flags.
- For shared patch logic or config mutation, edit `.github/actions/action.yml`.
- For changes that affect release notes or matrix generation, edit `.github/workflows/build-kernel-release.yml`.

### Change Packaging or Artifact Naming

- Update `.github/actions/action.yml`.
- Review release-note parsing logic in `.github/workflows/build-kernel-release.yml`, because it parses ZIP filenames.

### Change Release Behavior

- Update `.github/workflows/build-kernel-release.yml`.
- Be careful not to break tag generation, release-note generation, or artifact upload.

## Recommended Editing Rules for Agents

- Do not assume this repo can be fully tested locally on Windows.
- Prefer small, well-scoped changes because CI blast radius is large.
- Keep JSON and manifest naming aligned.
- If you change a `wild/*` config, verify the referenced manifest exists in the matching `manifests/<oos>/` folder.
- If you change ZIP naming, also update any code that parses artifact names later in the workflow.
- If you touch `.github/actions/action.yml`, assume all devices may be affected.
- If a failure appears during patching or compile, consider whether the real issue is in an upstream source revision, not in this repo.

## Validation Reality

There is no standalone unit test suite in this repo.

The practical validation path today is:

- check JSON and manifest consistency,
- review workflow logic,
- run the GitHub Actions workflow manually,
- inspect build logs and produced artifacts.

This means changes can sit unvalidated until someone explicitly triggers the workflow.

## Known Risks and Gotchas

- The repo is not self-contained.
- Builds depend on GitHub, GitLab, Google-hosted `repo`, OnePlusOSS, and CodeLinaro.
- Manual-only workflow means regressions may not be caught quickly.
- Patch application is intentionally tolerant but therefore fragile.
- The Android-version filters exclude OOS14 when selecting `android*-*.*`.
- Local documentation may drift from build reality if configs and action behavior change independently.

## Best Starting Points by Task Type

- Add or fix device support: start in `configs/` and `manifests/`.
- Change kernel features, patch flow, packaging, or validation: start in `.github/actions/action.yml`.
- Change matrix generation, workflow inputs, or release behavior: start in `.github/workflows/build-kernel-release.yml`.
- Update public-facing support expectations: review `README.md` and `compatibility.md`.

## If You Are Debugging a Failed Build

Check these in order:

1. Was the correct JSON config selected in the matrix?
2. Did `op_model` filtering exclude the expected device family?
3. Does the config point to the correct manifest and branch?
4. If `branch` is `wild/*`, does the local manifest file exist in the correct `manifests/<oos>/` directory?
5. Did an external clone or `repo sync` fail?
6. Did a SUSFS or other patch stop applying cleanly?
7. Did `.config` mutation produce an invalid kernel configuration?
8. Did artifact naming or release-note parsing break after a naming change?

## Maintenance Note

Keep this file updated when changing:

- config schema,
- directory layout,
- workflow inputs,
- release artifact naming,
- major dependency URLs,
- patch categories,
- compatibility policy.
