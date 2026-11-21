# NxNative builder reliability update

Complete replacements for the regular and Controls Editor workflows on **Main and Pocket**. Covers all three existing engine choices: NX2Apple, standard nxnative, and the legacy Metal 3 + BC fallback engine.

This is a **build-reliability update**, not a new engine or graphics patch. Your integrated Downwell and A Short Hike workarounds, settings, touch controls, DLC support and download methods are retained.

## Install the correct pair on each branch

| Branch in your repository | File from this ZIP | Destination |
|---|---|---|
| `main` | `main/build.yml` | `.github/workflows/build.yml` |
| `main` | `main/nxnative-ipa-builder-controls-test.yml` | `.github/workflows/nxnative-ipa-builder-controls-test.yml` |
| `pocket-settings-test` | `pocket-settings-test/build.yml` | `.github/workflows/build.yml` |
| `pocket-settings-test` | `pocket-settings-test/nxnative-ipa-builder-controls-test.yml` | `.github/workflows/nxnative-ipa-builder-controls-test.yml` |

Use the complete files. Do not paste patch fragments, upload the ZIP as a workflow, use Pocket files on Main, or merge the test branches. All new check code is embedded in the YAML; no companion scripts need uploading. The README and troubleshooting guide are documentation, not workflow dependencies.

**Recommended order:** upload the Pocket pair first, commit, and start a new Controls Editor run on Pocket. Check that the new early-check step passes and the build produces an IPA. Test that IPA on the appropriate device, then upload the Main pair. Before relying on every engine, run at least one fresh build for each engine selection you use; the legacy-device success does not verify modern-engine builds.

No new branch is required. Leave your working Downwell and A Short Hike reference branches unchanged during rollout. Keep your previous workflow revisions, working IPAs and save backups.

## What is new

### Early checks before game downloads

A new step, **Check dependencies and bundled shader tools before downloads**, runs after dependency installation. It checks:

- Python 3.12+ and required package imports.
- Capstone major version 5 and a small ARM64 disassembly test.
- LZ4 and Zstandard compression/decompression round trips.
- .NET 10 runtime availability.
- Required commands, with execution probes for xcodegen, zstd and glslangValidator.
- Both macOS and iPhoneOS SDK/Metal compiler availability.
- Bundled translator and SPIRV-Cross fingerprints against the engine's committed lock, plus translator/source agreement and SPIRV-Cross execution.
- The reviewed engine/tool-checker/lock revisions, including the NX2Apple upstream pin.

The step needs no game files or keys. Failed checks stop the build before large game downloads. Child-process probes have time limits, and arbitrary child output is not echoed into errors.

### Controlled Python environment

The job disables user-site package injection, clears inherited `PYTHONPATH` and `PYTHONHOME`, and enables unbuffered Python output. The new smoke-check subprocess also uses isolated Python mode. This is not a container or a new virtual environment for the build, and it does not change your selected engine or runtime game flags.

### Readable build progress

Bundled IPA builds, separate-RomFS staging/resume, and surveys use upstream **`--progress lines`** when the builder's help advertises it. You get real stage/count output, not a custom parser or invented percentages. If the option is absent, the existing engine output is retained.

Build commands still run directly: no wrapper swallows failures or turns a nonzero exit into success. Parallel game/key downloads and their existing messages are unchanged.

## Important compiler-policy detail

**This delivery keeps your existing strict Xcode 27.1 workflow policy.** It does not automatically select the latest Xcode or copy another project's version requirement.

The pinned upstream source/lock metadata can describe a different Metal compiler. The new check verifies bundled **non-Metal** tools against that committed lock, verifies that both selected Metal compilers run, and reports whether their fingerprints equal the committed values. A difference is explicitly reported, not passed off as equivalence.

Your existing later **Update shader toolchain lock and header** step is unchanged. The new early check itself does not rewrite a lock/header, relabel shader libraries, refresh engine pins, or set a compiler-override flag. Existing shader-chain consistency checks still apply. See TROUBLESHOOTING.md before changing versions or hashes.

## What stays the same

- All engine and action pins; no upstream engine switch.
- Both game-specific workarounds and their publication gates.
- Settings, English host UI, handheld defaults, controls/editor and motion/FPS changes.
- ASTC/BC handling, MSAA policy, memory behavior and texture/storage settings.
- Game/update/DLC inputs, key handling, downloader behavior and disk-saving safeguards.
- Bundled/separate RomFS, cache options, existing publication and retention behavior.
- No new shader-source-export option or diagnostic `env.txt` flags.

For **A Short Hike and Downwell legacy builds**, keep shader cache **OFF** as required by their existing guards. For A Short Hike, leave the profile override blank; the legitimate preliminary DEFAULT profile remains supported.

## Testing status

Local validation passed:

- 18 new reliability tests, including all four pinned source variants and failure cases.
- Four YAML files, 116 Bash blocks and 108 embedded Python blocks checked.
- Actual dependency imports, compression round trips and Capstone disassembly executed in an isolated **Linux** Python environment.
- Actual pinned tool-checker APIs exercised with **synthetic executables**, not Apple binaries.
- Existing A Short Hike suite: 20 tests; Downwell suite: 19 tests; integration checks also passed.
- Exact comparisons confirm all original workflow steps remain unchanged except the three documented progress additions; the only other changes are the new early-check step and four Python environment settings.

**No new Xcode compilation, macOS runner execution, IPA verification or device run of these updated workflows has been performed here.** Your earlier game successes remain valid user reports, but do not verify this new delivery. No remote repository changes or workflow dispatches were made by the assistant.

## Rollback

Restore the preceding workflow revisions or the branch-matched files from `short-hike-integration.zip`. That removes this reliability update while retaining the prior A Short Hike and Downwell integration. Start a new workflow run after committing; rerunning an older job uses its older workflow revision.

For failures and what evidence to share, read **TROUBLESHOOTING.md**.
