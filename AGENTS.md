# AGENTS.md

Guidance for AI agents (and humans) working in this repository. Read this before editing — it explains what the repo actually is, where the real logic lives, and which facts are easy to get wrong.

## TL;DR

- This is an **OP13-only, nikitos4683-branded fork** of [`WildKernels/OnePlus_KernelSU_SUSFS`](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS).
- It contains **no kernel source**. It holds one device config, one manifest, one local patch, and the GitHub Actions pipeline that fetches sources and builds the kernel in CI.
- The single build target is **OnePlus 13 (`OP13`) on OxygenOS 16 / Android 16 (`A16`), kernel `android15-6.6` (GKI 6.6)**.
- Builds are **manual-only** (`workflow_dispatch`). There is no local build and no way to fully validate a change on Windows — most failures originate in upstream sources or external repos, not here.
- The real system of record is [`.github/actions/build-kernel/action.yml`](.github/actions/build-kernel/action.yml). Treat it as high-blast-radius.

## Fork Identity & Upstream Relationship

- `origin` → `nikitos4683/OnePlus_KernelSU_SUSFS_OP13`, working branch **`op13`** (also the default branch).
- `upstream` → `WildKernels/OnePlus_KernelSU_SUSFS`.
- Upstream is multi-device; this fork is deliberately narrowed to OP13 by **filtering the build matrix** and by **deleting all other device configs/manifests** (only `a16/OP13.*` remain).
- Upstream historically renamed the OS folders `oos14/15/16 → a14/15/16` and the JSON field value `OOS16 → A16`. This fork follows that scheme: everything is `a16` / `A16` now. If you see `oos*`/`OOS*` anywhere, it is stale.

### Sync strategy (agreed with the maintainer)

- Merge upstream **only up to the last known-good RELEASE commit**, not `upstream/main` HEAD (recent upstream commits may be unstable). Find the release commit from the upstream release GitHub Actions run's `headSha`.
- Merge into a **new branch** (e.g. `sync-op13-release`), resolve conflicts, then let the maintainer decide when to merge into `op13`.
- Established style is **merge commits, not rebase**.
- On every sync, **preserve the fork customizations** listed under [Branding & Customizations To Preserve](#branding--customizations-to-preserve).

## Repository Layout

```
configs/a16/OP13.json                         # the ONLY active device config
manifests/a16/oneplus_13_w.xml                # the ONLY active local manifest
patches/local/anykernel_branding.patch        # the ONLY local patch (AnyKernel3 branding)
README.md                                      # public-facing overview
compatibility.md                               # device compatibility policy
AGENTS.md                                      # this file
.github/
  workflows/
    build-kernel-release.yml                   # MAIN entry point (manual build + optional release)
    oplus-kernel-monitor.yml                   # upstream OP13/A16 source tracker → status-page branch
    mirror-toolchains.yml                      # mirrors toolchains into the `toolchain-cache` release
    clean-up.yml                               # ccache / workflow-run maintenance
  actions/
    build-kernel/action.yml                    # REAL build logic (parse, patch, compile, package)
    kernel-source-sync/action.yml              # archive-based source + toolchain fetch
    cache/restore/action.yml                   # restore ccache/LTO cache from release assets
    cache/save/action.yml                      # save   ccache/LTO cache to   release assets
    disk-cleanup/action.yml                    # free runner disk space
  ISSUE_TEMPLATE/                              # bug report + feature request, scoped to OP13/A16
```

There are **no legacy `oos*` or other-device folders** — cleanup removed them entirely.

## In This Repo vs. Outside This Repo

**In repo:** device config, local manifest, local branding patch, the CI pipeline, release-note generation logic.

**Outside repo (fetched at build time):** the OnePlus kernel source trees, the SUSFS patch set, the shared `kernel_patches` stack, AnyKernel3, KernelSU / KernelSU-Next, and all toolchains. Consequently, local inspection explains *orchestration and intent*, but a compile or patch failure can live entirely in an upstream revision.

## Config Schema — `configs/a16/OP13.json`

The build matrix is generated from every `configs/**/*.json`, then filtered to `model == "OP13" && os_version == "A16"`. Current file:

```json
{
  "model": "OP13",
  "soc": "sun",
  "branch": "wild/sm8750",
  "manifest": "oneplus_13_w.xml",
  "android_version": "android15",
  "kernel_version": "6.6",
  "os_version": "A16",
  "lto": "thin",
  "zyc_compiler": "https://github.com/ZyCromerZ/Clang/releases/download/19.0.0git-20240723-release/Clang-19.0.0git-20240723.tar.gz",
  "c_compiler": "kernel_platform/prebuilts/clang/host/linux-x86/clang-r510928/bin",
  "rust_compiler": "",
  "bindgen": "",
  "rust_build": false,
  "disk_cleanup": false,
  "hmbird": true,
  "susfs": true,
  "ds": false,
  "bbg": false,
  "bbr3": true,
  "ttl": true,
  "ip_set": true,
  "unicode": false,
  "ntsync": false,
  "uname": "nikitos4683"
}
```

Field meanings (validation lives in `build-kernel/action.yml` → *Validate Inputs*):

| Field | Meaning / rules |
|-------|-----------------|
| `model` | Device id used in logs, artifacts, release notes. **Must start with `OP`**. |
| `soc` | SoC family (e.g. `sun`). `[A-Za-z0-9_-]+`. |
| `branch` | Manifest branch. A `wild/*` value triggers the **local-manifest** flow (see below). |
| `manifest` | XML filename **or** an `https://…xml` URL. |
| `android_version` | `android<N>` — the GKI base (`android15`), **not** the marketing OS. |
| `kernel_version` | `X.Y` (`6.6`). |
| `os_version` | Marketing OS; **must start with `A`** (`A16`). Drives the matrix filter and artifact names. |
| `lto` | `none` \| `thin` \| `full`. |
| `zyc_compiler` | Optional external clang tarball URL (highest clang priority). Empty = skip. |
| `c_compiler` | Optional in-tree clang path override. |
| `rust_compiler` / `bindgen` | Optional in-tree rust / bindgen path overrides (only relevant when `rust_build`). |
| `rust_build` | Enables rust/bindgen tooling and rust binder configs. |
| `disk_cleanup` | Run the `disk-cleanup` action before building. |
| `uname` | Localversion / branding string. **Max 44 chars.** |
| `hmbird`,`susfs`,`ds`,`bbg`,`bbr3`,`ttl`,`ip_set`,`unicode`,`ntsync` | Boolean feature toggles (`"true"`/`"false"`) — see [Patch & Feature Logic](#patch--feature-logic). |

> Note: `android_version` stays `android15` while `os_version` is `A16` — the GKI base is `android15-6.6` even though the marketing OS is Android 16. This mismatch is intentional; don't "fix" it.

## Manifest Conventions

Active manifest: `manifests/a16/oneplus_13_w.xml`. It pins:
- CodeLinaro (`clo-la`) prebuilts: kernel-build-tools and clang, at fixed revisions.
- OnePlusOSS (`origin`): `kernel_platform/common` and the modules/devicetree tree, at `oneplus/sm8750_b_16.0.0_oneplus_13`.
- WildKernels (`wild`): AnyKernel3 at a pinned revision.

Manifest resolution logic (in `kernel-source-sync` → *Download and Prepare Manifest*):
1. If `manifest` is an `https://` URL → download it.
2. Else if `branch` matches `wild/*` → **copy the local file** from `manifests/<os_version lowercased>/<manifest>` (i.e. `manifests/a16/oneplus_13_w.xml`). **This is the active path** and is easy to miss.
3. Else → fetch from `OnePlusOSS/kernel_manifest` at `<branch>/<manifest>`.

If you touch the config's `os_version` or `manifest`, keep the `manifests/a16/…` path in sync, or the `wild/*` flow breaks.

## Build Pipeline — `build-kernel-release.yml`

Manual-only (`workflow_dispatch`). Key inputs:

| Input | Purpose |
|-------|---------|
| `make_release` | Create a **draft** release after successful builds. |
| `ksu_options` | JSON array of KSU variants, e.g. `[{"type":"ksun","hash":"dev"}]`. Missing `hash` defaults: `KSUN→dev`, `KSU→main`. |
| `optimize_level` | `O2` or `O3`. |
| `clean_build` | Disable ccache. |
| `debug` | Dump debug artifacts / verbose logs. |
| `mirror_toolchains` | Run the toolchain mirror before building (rarely needed). |
| `build_timestamp` | Fixed `uname -a` timestamp for reproducibility (empty = current time). |
| `android15-6_6_susfs_branch_or_commit` | Override SUSFS ref (empty = `gki-android15-6.6`). |

Jobs, in order:

1. **`set-op-model`** — checks out `configs/` only, generates the matrix, filters to `OP13`/`A16`, normalizes `ksu_options`, and **pre-resolves hashes before building**:
   - KSU/KSUN ref → commit SHA via the GitHub GraphQL API (retry x3).
   - SUSFS ref → commit SHA via the GitLab GraphQL API; on release runs it also extracts `SUSFS_VERSION` from `susfs.h` and derives `susfs_base_version` (used for the release tag). Version mismatch across active GKI keys is a hard error on release.
   - Emits one matrix row per (device × KSU option), plus a build-plan summary.
2. **`mirror_toolchain`** — only if `mirror_toolchains=true`; calls `mirror-toolchains.yml`.
3. **`prepare_ccache`** — downloads the custom ccache binary **once** and shares it as an artifact (avoids ~30 parallel jobs hammering the same raw URL and getting throttled).
4. **`build`** — matrix job (`fail-fast: false`). Installs deps, restores ccache, strips KSU fields from the config, then runs the `build-kernel` composite action.
5. **`trigger-release`** — only if `make_release=true` and all builds succeeded: computes the next `<SUSFS_BASE_VERSION>-r<N>` tag, downloads artifacts, generates release notes from the ZIP filenames + matrix metadata, creates a **draft** release, and uploads the ZIPs.

## Composite Action — `build-kernel/action.yml`

This is the authoritative build logic. High-level flow:

1. Parse `op_config_json` → `OP_*` env vars; validate every field.
2. Download the `repo` tool to `/usr/local/bin/repo` and export `$REPO`. **It is never invoked** — source sync is archive-based (see below). This is dead legacy from the upstream `repo sync` era; don't build on it.
3. **Sync kernel source** via `kernel-source-sync`.
4. Read the real kernel version from the synced `Makefile` (`VERSION.PATCHLEVEL.SUBLEVEL`) and derive `SUSFS_KERNEL_BRANCH=gki-<android>-<X.Y>`.
5. Detect clang: priority **ZyC URL → `c_compiler` override → in-tree prebuilts → system clang**. Optionally detect/install rust + bindgen when `rust_build`.
6. Restore ccache and (if LTO) the LTO/LD cache from release assets.
7. Clone deps into `$GITHUB_WORKSPACE` (`kernel_patches`, `susfs4ksu`), apply the local AnyKernel3 branding patch, and check out the resolved SUSFS ref. **AnyKernel3 is not cloned — see below.**
8. Strip ABI-protected exports.
9. Add **KernelSU** or **KernelSU-Next** (compute `KSU_VERSION`, set hook needs).
10. Apply **SUSFS** patches (version-specific case block, `v1.5.8`…`v2.2.0`) + arch defconfig.
11. Apply optional feature patches (BBG, KSU hooks, other patches, NTSync, Unicode) and defconfig fragments (tmpfs, BBRv3-only, qdisc, TTL, IP set/IPv6 NAT, Droidspaces, build tuning, rust). The fork deliberately leaves vendor-module loading at the OnePlus factory behavior: no module overlay and no vendor-module blacklist patch.
12. Set branding localversion `-<android>-<uname>`.
13. `make gki_defconfig` → tweak `.config` (localversion, O2/O3, `-mcpu=oryon-1` for `wild/sm8750|wild/sm8850|wild/sm8845`, LTO mode) → `olddefconfig` → parallel `make Image`.
14. Validate the `Image` (see below), package into AnyKernel3 ZIP, compute SHA256s, upload artifact, save caches.

### Source sync — `kernel-source-sync/action.yml`

Not a classic `repo sync`. A Python step parses the manifest and, per project, downloads a **release/archive tarball** (`github.com/.../archive/<rev>.tar.gz`, googlesource `+archive`, or codelinaro `-/archive`) in parallel, extracting into the project path. Toolchain projects (clang, rust, clang-tools, build-tools, AnyKernel3) are first looked up in this repo's **`toolchain-cache` release** (single file or `.part*` splits); a cache miss for a toolchain is **fatal** and tells you to run the mirror workflow. `linkfile`/`copyfile` manifest directives are applied afterward. This makes the build heavily dependent on GitHub release availability and upstream archive endpoints.

### AnyKernel3 comes from the manifest, not from a clone

The AnyKernel3 tree that actually gets branded and packaged is **`$AK3_FOLDER` = `$GITHUB_WORKSPACE/OP13/AnyKernel3`**, synced from the manifest project (`WildKernels/AnyKernel3` @ the revision pinned in `manifests/a16/oneplus_13_w.xml`) and served through the `toolchain-cache` release, since `AnyKernel3` is in `TOOLCHAIN_MAP`.

So: **to change the packaged AnyKernel3, edit the manifest project revision.** Both the branding patch (`patch -d "$AK3_FOLDER"`) and packaging (`cp Image "$AK3_FOLDER/"`, `cd "$AK3_FOLDER"`) target that tree, and the step now hard-fails with a clear error if it is missing.

> **Upstream drift warning.** Upstream additionally runs `retry_clone https://github.com/TheWildJames/AnyKernel3.git` (branch `gki-2.0`) inside *Fetch SusFS and Other Dependencies*. That step's cwd is `$GITHUB_WORKSPACE`, so the clone lands at `$GITHUB_WORKSPACE/AnyKernel3` — a **different path** from `$AK3_FOLDER`, and nothing ever reads it. This fork removed that dead clone; **an upstream merge may reintroduce it — drop it again.**

Contrast: `kernel_patches` and `susfs4ksu` *are* consumed from their clones, because `$KERNEL_PATCHES_FOLDER`/`$SUSFS_FOLDER` do resolve to `$GITHUB_WORKSPACE/...`.

## Caching Architecture (release-asset based, NOT `actions/cache`)

Caches are stored as **assets on dedicated releases** in this repo, via `cache/save` + `cache/restore`:
- `ccache-cache` bucket — per (ksu_type, model, os, kernel, clang-fingerprint) ccache.
- `lto-cache` bucket — ThinLTO/LD cache (only when `lto != none`).
- `toolchain-cache` — deduplicated toolchain tarballs, populated by `mirror-toolchains.yml`.

Archives are zstd-compressed and split at ~1.86 GB into `.part*` assets. `clean-up.yml` handles `actions/cache` GitHub caches and old workflow runs separately (its device list is legacy multi-device heritage and does not reflect what this fork builds).

## Patch & Feature Logic

Feature toggle → what it does (all in `build-kernel/action.yml`):

| Toggle | Effect | Current OP13 |
|--------|--------|--------------|
| `susfs` | Copy SUSFS fs/include files, apply version-matched patch set + `CONFIG_KSU_SUSFS_*`. | ✅ on |
| `hmbird` | Apply OnePlus HMBIRD SCX (fengchi + overwriter + hmbird_config) patches. | ✅ on |
| `bbr3` | Backport BBRv3, explicitly disable BBRv1, select BBRv3 as the kernel default, and verify all three conditions in the final config. | ✅ on |
| `ttl` | TTL/HL target + match configs. | ✅ on |
| `ip_set` | IP set + IPv6 NAT configs + `IPv6_NAT_FIX.patch`. | ✅ on |
| `bbg` | Baseband Guard LSM (external setup script) + LSM Kconfig edit. | ❌ off |
| `ds` | Droidspaces (SYSVIPC/KABI patches + configs). | ❌ off |
| `ntsync` | NTSync primitives (model/GKI-matched patch). | ❌ off |
| `unicode` | Unicode bypass fix patch (version-selected). | ❌ off |

**Always-applied** (independent of toggles): `fake_config.patch`, tmpfs XATTR/POSIX ACL, qdisc set (`FQ`, `FQ_CODEL`, `CAKE`, `PIE`, `FQ_PIE`), and build-tuning configs. `fake_config.patch` only changes the configuration reported through the embedded config data; it does not disable the corresponding compiled features.

Most patches use `patch -p1 --forward` with fuzz — tolerant by design, which means an upstream source change can silently stop a patch from applying and break the build with no change on this side. There are also several `sed`/`perl` "fake patch" fixups gated on `android15-6.6` to make SUSFS apply cleanly.

## Build Output, Validation & Artifact Naming

`Image` validation: existence, ARM64 file format, minimum size **6,102,400 bytes**, warning count, SHA256, `uname` string extraction.

Artifact ZIP names (parsed later by the release step, so keep them stable):
- No SUSFS: `AK3-NIKITOS4683-<model>_<os>_<kernel-full>_<ksu-type>_<ksu-version>.zip`
- With SUSFS: `AK3-NIKITOS4683-<model>_<os>_<kernel-full>_<ksu-type>_<ksu-version>_SuSFS_<susfs-version>.zip`

The `AK3-NIKITOS4683-` prefix is fork branding and is stripped by the release-notes parser (`rest="${zipname#AK3-NIKITOS4683-}"`). Packaging copies the built `Image` into the AnyKernel3 tree before zipping.

## Release Flow

On `make_release=true`:
1. Compute the next tag: `<SUSFS_BASE_VERSION>-r<N>` (increment the `-r*` suffix of the latest matching tag, else `-r1`).
2. Download all ZIP artifacts.
3. Generate release notes dynamically from ZIP filenames + `matrix.json` feature flags (devices table, build config, feature list, manager links, install steps, credits). `SUSFS_BASE_VERSION` is derived from the SUSFS `susfs.h` at build time, not hardcoded.
4. Create a **draft** GitHub release and upload the ZIPs.

## Auxiliary Workflows

- **`oplus-kernel-monitor.yml`** — scheduled **every 12 h** (`0 */12 * * *`) + manual. Derives its tracking scope from `configs/a16/OP13.json` and the local manifest (OnePlusOSS `origin` projects only), diffs against state stored on the `status-page` branch, opens/updates a notification issue on change, and regenerates the `status-page` README. If you change this cron, update the "Update Frequency" line in `README.md` to match.
- **`mirror-toolchains.yml`** — Sundays 00:00 UTC + manual + `workflow_call`. Resolves unique toolchains across all configs and uploads them (split if >~1.86 GB) into the `toolchain-cache` release. Run this if source sync reports a fatal toolchain cache miss.
- **`clean-up.yml`** — manual ccache / workflow-run maintenance with dry-run and per-device/age filters. Its large device dropdown is inherited from upstream and is broader than this fork's single target.

## Branding & Customizations To Preserve

On every upstream sync, keep:
- `uname: nikitos4683`, ZIP prefix `AK3-NIKITOS4683-` (parser strips exactly this), release brand `nikitos4683`.
- `patches/local/anykernel_branding.patch` — the **only** local patch; rewrites `anykernel.sh` strings/URLs to nikitos4683. Applied to the manifest-synced AK3 tree (`$AK3_FOLDER`) with `--fuzz=0`, so it must stay in sync with the AnyKernel3 revision pinned in the manifest. If an upstream AK3 bump breaks it, regenerate the patch against that exact revision instead of relaxing the match.
- Matrix filter pinned to `OP13`/`A16`; all non-OP13 configs/manifests deleted.
- Optional patches off (`ds/bbg/unicode/ntsync = false`).
- The upstream general optimization patch stack is removed completely. Performance patches may only be introduced individually after OP13-specific validation; drop the stack again if an upstream sync reintroduces it.
- Rust build support off (`rust_build: false`).
- BBRv3 as the sole compiled BBR variant and kernel default (`bbr3: true`; BBRv1 disabled, `DEFAULT_BBR3=y`, and `DEFAULT_TCP_CONG="bbr3"` verified after `olddefconfig`).
- Stock OnePlus vendor-module behavior: no module intercept/overlay patch and no vendor-module debloat/blacklist patch. Drop both again if an upstream sync reintroduces them.
- Keep `fake_config.patch`; it is part of the root-hiding configuration-reporting behavior.
- Point new `kernel_patches` at `WildKernels/kernel_patches`.
- The removal of upstream's dead `TheWildJames/AnyKernel3` clone (see [AnyKernel3 comes from the manifest](#anykernel3-comes-from-the-manifest-not-from-a-clone)) — re-drop it if a sync brings it back.
- The OP13-scoped issue templates, and the deletion of upstream's `request_new_device.yml` / `request_compatibility.yml`.

## Docs & Known Drift

- `README.md` — public overview; advertises KSU/KSUN, SUSFS `v2.2.0`, HMBIRD, BBRv3, CAKE/PIE, TTL, IP set/IPv6 NAT, thin LTO, tmpfs XATTR, custom branding. Keep these claims aligned with the config flags and action behavior, and keep its "Update Frequency" line in sync with the monitor cron.
- `compatibility.md` — policy: stock OxygenOS 16 (Android 16) only, GKI 6.6.x on the `android15-6.6` KMI, kernel source tracked from OnePlusOSS `oneplus/sm8750_b_16.0.0_oneplus_13`; tells users to verify their KMI with `uname -r`; don't reuse ZIPs across major OTAs; nothing outside OP13/A16 is supported. Keep the KMI/source-branch facts in step with `configs/a16/OP13.json` and the manifest.
- `.github/ISSUE_TEMPLATE/` — **rewritten for this fork**: only `bug_report.yml` (OP13 / stock OxygenOS 16, KSUN/KSU managers, ZIP filename + `uname -r` required) and `feature_request.yml` (distinguishes "enable an existing disabled toggle" from "add a new patch") remain. Upstream's `request_new_device.yml` and `request_compatibility.yml` were **deleted** — this fork will never add a device and keeps no non-OnePlus compatibility list. `config.yml` routes other devices to upstream and labels the WildKernels Telegram as upstream community, not fork support. **An upstream merge may resurrect the deleted forms — delete them again.**
- `clean-up.yml`'s device dropdown and the generic GKI helper logic in `set-op-model` (SUSFS keys for android12-5.10 … android16-6.12) are likewise multi-device leftovers — harmless, but don't mistake them for supported targets.

## Common Maintenance Tasks

- **Change a build feature** → edit the toggle in `configs/a16/OP13.json`; for shared patch/config logic edit `build-kernel/action.yml`.
- **Change matrix / inputs / release behavior** → edit `build-kernel-release.yml` (careful with tag generation and the ZIP-filename parser).
- **Change packaging / artifact naming** → edit `build-kernel/action.yml` **and** the release-notes parser in `build-kernel-release.yml` together.
- **Update the manifest / kernel base** → edit `manifests/a16/oneplus_13_w.xml` (+ config), and update `compatibility.md`/`README.md`.

## Editing Rules For Agents

- Don't assume anything can be built or fully validated locally on Windows.
- Prefer small, well-scoped changes — CI blast radius is large and slow to verify.
- Keep JSON `os_version` (`A16`), the `manifests/a16/` path, and the `wild/*` flow consistent.
- If you change ZIP naming, update every downstream parser in the same change.
- Treat `build-kernel/action.yml` and `kernel-source-sync/action.yml` as high-blast-radius.
- On patch/compile failure, first suspect an upstream source revision, not this repo.
- This repo has **no test suite**. Practical validation = check JSON/manifest consistency, review workflow logic, trigger the workflow manually, read logs and artifacts. Changes can sit unvalidated until someone runs a build.

## Debugging A Failed Build — check in order

1. Did the matrix still include `OP13` on `A16` after the filter?
2. Does the config point to the right `branch`/`manifest`, and does `manifests/a16/oneplus_13_w.xml` exist (for the `wild/*` flow)?
3. Did an archive download, toolchain-cache lookup, or source sync fail? (Toolchain miss → run `mirror-toolchains.yml`.)
4. Did KSU/KSUN or SUSFS hash resolution fail in `set-op-model`?
5. Did a SUSFS or other patch stop applying cleanly (upstream drift)?
6. Did `.config` mutation (LTO/O-level/localversion) produce an invalid config?
7. Did `Image` validation fail (size/format), or did ZIP naming / release-note parsing break?

## Keep This File Updated When Changing

Config schema · directory layout · workflow inputs · caching/release-asset scheme · artifact naming · major dependency URLs · patch categories · monitor/mirror cadence · compatibility policy.
