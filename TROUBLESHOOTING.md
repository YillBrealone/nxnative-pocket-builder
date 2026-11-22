# Troubleshooting

**Start with the first red step in the Actions run.** Later skipped steps are usually a result of that failure.

If no IPA was created, changing LiveContainer or in-game settings will not fix the build problem.

## I cannot start the workflow

- Check that Actions is enabled and you have permission to run it.
- Select the correct workflow and branch.
- On a phone or iPad, try desktop-site mode or another browser if the form does not work.
- If the job stays queued, check access to the required **`xcode-27` runner**.

After replacing a YAML file, commit it and start a **new run**. **Re-run jobs** uses the old run's workflow revision.

## Xcode or dependency checks failed

These failures happen before game downloads and do not mean the game is incompatible.

| Problem | What to check |
|---|---|
| Wrong Xcode | These workflows require **27.1**. Do not switch versions or bypass the check blindly. |
| Missing Python package, .NET or command | Read the preceding dependency/setup step. Retry if installation failed temporarily. |
| Missing SDK or Metal compiler | Check the selected Xcode and Metal installation output. |
| Source/hash/translator mismatch | Restore the matching engine/workflow revision. Do not edit expected hashes or rename shader libraries to force a pass. |
| Timeout | Retry once for a possible runner issue. Repeated failures need the exact error reviewed. |

When available, save the **NxNative-Build-Environment** artifact before its one-day expiry. Match its run ID/attempt with the Actions result. It records observed versions, not a guaranteed-good dependency set. Missing inventory data alone does not mean a game is incompatible.

A note that Metal differs from the committed upstream lock is not itself a failed check. It reports a difference while retaining the workflow's existing Xcode policy.

## Caching did not make the build faster

Check **Cache and build timing** in the run summary and the Actions step durations.

- **Miss:** normal on the first run or after a matching input/tool/workflow changes. Do not weaken the cache key to force a hit.
- **Exact hit:** an archive was restored; the builder may still need to compile work not covered by it. Compare the completed build time and total run time.
- **Unavailable:** cache restoration failed or the cache step did not run. Python installation can continue normally; review the step log.
- **Disabled:** expected when Python caching is disabled by its variable, or Shader cache is off.

Python download caching is not a dependency lock. Set **`PYTHON_DOWNLOAD_CACHE=false`** to compare an uncached run. The installer retries once without cache if installation fails; a second failure is still reported. Do not cache whole Homebrew/Xcode installations to work around failures.

Keep shader caching off for the existing Downwell/A Short Hike workarounds. Cache transfer and input hashing can outweigh savings, especially for a small game. GitHub caches are not covered by the one-day artifact retention setting.

### Can I use the generated constraints file immediately?

No. It is a draft snapshot, not an approved lock. Confirm that its matching full build succeeded, test the IPA, review which packages belong in the constraints, then repeat-build with those constraints before promoting them. A successful Survey Only run is not enough. No exact dependency versions were newly approved in this update.

## Game or key download failed

Common causes: expired links, missing files, login requirements, permissions, download quotas, or a webpage supplied instead of a direct file link.

- **Game:** check that the link works without an interactive login. If Drive blocks downloads, try a supported direct link for your own dump.
- **Keys:** check the `Keys_URL` secret and its access/expiry. **Do not make your keys public or share them for troubleshooting.**

Downloads run in parallel. “Finished downloading prod.keys” does not mean the game download succeeded.

“No update configured” and “No DLC configured” are normal when those inputs are intentionally empty.

## Extraction or title detection failed

Check that:

- The download is a complete game package, not an HTML page or damaged archive.
- You supplied the **base game**, not only an update or DLC.
- Updates, DLC and your keys match the intended game/packages.
- **Profile override** is blank unless you need a specific profile.

If you enter a base URL in the run form, enter any wanted update/DLC links there too. Saved update/DLC secrets are not used with a popup base URL.

### Duplicate packages or filename conflicts

Packages are skipped only when a filename/size collision also has matching SHA-256 contents. Hashing large colliding files can take time.

If two files have the same name but different contents, the builder uses a label-prefixed name when free. If that name also contains different data, it stops rather than overwriting anything. Give the conflicting files distinct names in your own archive/download folder and retry. Do not rename file extensions to hide an invalid signature.

## It says DEFAULT or provisional profile

The engine has no built-in profile for that game and may create one automatically. This is not automatically an error, but it does not guarantee compatibility.

Do not select another game's profile to hide the message.

## Shader compilation failed or progress looks stuck

Read the first compiler error. Some shaders take a long time without frequent progress updates.

Keep the engine and its shader tools together. Try a clean build with cache off. Do not bypass compiler checks, disable validation or rename shader chains.

For repeated failures, share the failed step and reviewed build logs.

## Downwell or A Short Hike's patch check failed

Their integrated legacy workarounds require **Metal 3 + BC fallback**, **Build IPA**, **shader cache off**, and a blank profile override.

An unknown source hash or missing IPA patch marker needs review. Do not remove the guard. These workarounds do not apply to unrelated games or modern-engine builds.

## Disk space ran out

Extraction and shaders can need much more space than the downloaded game. Check the disk-space messages and use a runner with enough storage.

Separate RomFS makes the final IPA smaller; it does not guarantee low build-time disk use. Keep the existing cleanup safeguards enabled.

## I cannot find or install the IPA

- Make sure you selected **Build IPA**, not Survey Only.
- Check the run summary for download links; extract GitHub's ZIP artifact.
- Download promptly—links and artifacts expire.
- If publication was blocked by a verification check, resolve that error first.
- The IPA is unsigned and still needs a suitable installation/signing method.

## The app crashes or graphics are wrong

Confirm the intended device/engine was selected. In separate-RomFS mode, check that the matching `romfs.img` is in the correct game's Documents folder.

Get a **new guest/crash log** from that attempt. Do not assume every crash is memory, signing or LiveContainer-related. A successful build does not prove device compatibility.

Keep your previous working IPA and back up saves before replacing it again.

## Controls or settings look different

Options depend on the workflow and engine. The advanced layout editor currently targets **NX2Apple** and is not identical across all engines. Check which branch and engine produced the IPA.

### A setting appears but makes no difference

Visibility is not a guarantee of game support. Render scale and dynamic resolution may not affect a particular title; frame generation may add artifacts or load. Start with your working defaults, change one option at a time, restart, and compare the same scene. Do not force higher FPS to fix a build failure.

Support notes describe uncertainty, not measured performance on your device. No renderer or game-fix changes were made in this reliability update.

## Asking for help

Send the run/job link, selected branch/engine, and the first failed step's final error lines. For an in-game problem, include the device, OS, installation method and new guest log.

Build-log and environment artifacts expire after **one day**. This does not automatically delete Actions console logs or run history. Download the records you need promptly and review them before sharing.

**Never send keys, private download links or unreviewed raw logs.** Automatic redaction is not a guarantee.

To roll back, restore the previous workflow files or reinstall a known-working IPA. No full branch merge is needed.
