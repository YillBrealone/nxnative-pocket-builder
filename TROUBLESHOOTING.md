# Troubleshooting — regular and Controls Editor builds

Applies to both Main and Pocket, with any of the three engine selections. Always start with the **first red step**, not later steps marked skipped. A completed workflow or IPA download does not prove game compatibility.

## 1. New early-check step fails

Step: **Check dependencies and bundled shader tools before downloads**.

This step runs before game downloads. A failure here is not evidence that your game dump, keys or device is incompatible.

| Message or area | Meaning and next action |
|---|---|
| Unknown engine revision / NX2Apple upstream pin differs | The checkout/pin no longer matches this delivery. Restore the matching full workflow or have the new source reviewed. Do not change expected hashes to force a pass. |
| Missing/symbolic shader-tool input / source or lock differs | A required checker, chain source or committed lock is absent/changed. Check whether the workflow or engine was manually edited. Use the pinned revision; do not regenerate fingerprints to bypass this early check. |
| Python package/native-library smoke check failed | An import or native operation failed. Check the preceding **Install dependencies** log. Tested imports are `lz4.block`, `capstone`, `fontTools`, `backports.zstd`, `gdown`, `nsz`, `zbic`; Capstone must be major 5. The test also exercises ARM64 disassembly and LZ4/Zstd round trips. This failure does not request game keys. |
| .NET 10 runtime is missing | Review **Set up .NET**. An SDK/runtime for an unrelated version or architecture may not run the translator. Do not remove the runtime check just because `dotnet` exists. |
| Required command is missing | Review the dependency-install step. Commands checked include xcodegen, zstd, glslangValidator, unar, gdown and nsz. |
| Xcode 27.1 requirement | This delivery intentionally retains the existing reference toolchain. Do not substitute 27.2/latest or use `--allow-other-metal` as a blind workaround. Share the Xcode-selection output for review. |
| macOS/iPhoneOS SDK or Metal probe failed | The selected SDK/component is missing, unusable or timed out. The existing install step already attempts Metal-component installation. Review its output and which Xcode was selected. |
| Bundled translator/SPIRV-Cross check failed | A bundled executable could not run, timed out, or the checker failed. Check the selected engine/revision and runtime setup. Do not replace the bundled compiler with a random Homebrew copy. |
| Bundled shader tool failed its committed fingerprint check | A translator binary, SPIRV-Cross or embedded source fingerprint does not match the committed lock. A fresh download/review of the pinned source is appropriate; a shader-cache toggle does not repair the source. |
| Translator embedded source does not match repository source | Translator/source versions do not agree. Keep translator and source together. Do not relabel shader chains. |
| Invalid shader-tool response/inventory | The checker API or lock structure differs from what was reviewed. Treat it as a source review requirement. |
| Inputs changed during the read-only check | Files changed while checks ran. Investigate the checkout/runner; do not bypass the check. |
| Probe timeout | The child tool did not finish in its allotted time. The early step itself has a ten-minute limit. A fresh run can help a transient runner issue; repeated timeouts need investigation. |

**Note rather than failure:** “selected Xcode 27.1 Metal differs from the committed upstream lock” reports an existing difference. The selected compiler is still required to run, and strict Xcode 27.1 selection remains enforced. The new check does not establish compiler equivalence or rewrite the lock. The workflow's pre-existing later lock/header preparation and shader-chain checks are retained. Do not change Xcode/lock policy just to hide this note.

The early step suppresses arbitrary subprocess output to avoid dumping environment details. Send its exact labeled error plus the relevant earlier setup step—not keys or secret values.

## 2. Game download fails

Step: **Download game files and keys**.

The downloader is unchanged by this update. Game and key downloads run in parallel; nearby lines can describe different downloads. “Finished downloading prod.keys” does not mean the game download succeeded.

For a Google Drive game-download failure:

- Check that the game file/link still exists and that it is accessible to the runner without an interactive login.
- Check permissions, download quota and any warning/interstitial page.
- For a game dump you are authorized to use, try a supported direct file URL if Drive refuses automated downloads.
- “No UPDATE_URL configured” / “No DLC_URL configured” are informational when those inputs were intentionally blank.

Do not post `prod.keys`, key values or private key-download links. Do not make your keys public to debug a download. Store sensitive links in repository secrets where supported; workflow input fields and logs are not a privacy guarantee.

A failed download occurs before extraction/compilation and says nothing about Naruto or another game's runtime support. This update adds no new downloader categories or exit-code scheme.

## 3. A Short Hike / Downwell guards fail

- Choose the legacy **iPad (2020 A12Z) / iPhone 15 or older (Metal 3 + BC fallback)** engine for the targeted workaround.
- Use **Build IPA**, with **shader cache OFF**.
- Leave A Short Hike's profile override blank. A preliminary `DM_GAME=DEFAULT` is normal when no built-in profile exists; the corrected integration handles it and checks the final profile later.
- An unknown source hash means the patch cannot be applied safely. Do not change the hash or remove the guard blindly.
- A missing workaround marker in the IPA stops publication deliberately; do not install an IPA as a verified workaround if that check failed.

No shader-export checkbox or diagnostic `env.txt` changes are required for the integrated workarounds. Modern-engine builds and other titles skip these title-specific steps.

## 4. Progress looks quiet or different

The workflow selects upstream `--progress lines` only when advertised in the builder's help. It keeps the default output otherwise.

Stage/count updates come from the engine. Some work cannot be measured, and a single shader/compiler task can stay on one stage for a while. No percentage should be interpreted as proof that a hang is impossible. The output-mode change does not suppress nonzero exits or alter the existing job timeout.

Separate RomFS has two build passes: staging pauses intentionally for the existing image upload, then the final build resumes. An intentional staging pause is not necessarily a failure.

## 5. IPA missing or publishing blocked

Open the first failed build, game-specific verification or IPA-inspection step. Later upload steps being skipped is normally a consequence, not the root cause.

The existing IPA checks can reject missing output, invalid ZIP/plist metadata, missing game-fix markers, credential-like filenames or host UI audit failures. Do not bypass those checks to obtain a download. GoFile and GitHub fallback behavior are unchanged; GitHub artifacts may require sign-in/access.

## 6. IPA builds but the game crashes or renders incorrectly

This update changes build checks/output only. It does not add game compatibility, increase FPS or alter textures/memory/ASTC/MSAA behavior.

Send the new device result and guest log, along with the run link, engine selection and device/OS/LiveContainer details if they are not already known. For the known iPad, there is no need to repeat the previously supplied original A Short Hike crash log; a new result needs its new log. A successful early check or marker check is not a GPU/gameplay test.

## Sharing useful evidence safely

Send:

1. The workflow run/job link and selected branch/engine.
2. The first failed step's name and final error/traceback lines.
3. The relevant setup output if the error is in the early check.
4. Existing redacted build logs, if generated, after reviewing them yourself.

If logs require authentication, upload a reviewed log file or provide a temporary direct log link. Never include key files, key values, full private environment dumps, private package links or unreviewed raw logs. Do not rerun the old commit after uploading a corrected YAML: commit the new file and use **Run workflow** for a fresh run.

## Rollback

Use the preceding branch-matched files from `short-hike-integration.zip`, or restore the previous commit of each workflow. The working Downwell/A Short Hike test branches are unchanged. No branch merge or extra permanent branch is needed.
