# Troubleshooting

## Start here

Open the failed run under **Actions** and expand the **first red step**. Later steps marked skipped usually mean an earlier step stopped the build.

A build failure and an in-game crash are different problems. If the IPA was never created, changing LiveContainer or game settings will not fix that build failure.

## I cannot start the workflow

- Check that Actions is enabled and you have permission to run workflows.
- Choose the correct workflow and branch in **Run workflow**.
- If the form is awkward on a phone or iPad, try the browser's desktop-site mode or another browser.
- If the run stays queued, check that the repository has access to the required **`xcode-27` runner**.

After changing a YAML file, commit it and start a **new run**. **Re-run jobs** uses the old run's workflow revision.

## The Xcode, dependency or tool check failed

These checks happen before game downloads. They do not mean your game is incompatible.

- **Wrong Xcode:** these workflows require **27.1**. Do not switch to the latest version or bypass the check without reviewing the engine requirements.
- **Missing Python package, .NET or command:** inspect the preceding dependency/setup step. A fresh run may resolve a temporary installation failure.
- **Metal compiler or SDK missing:** inspect the selected Xcode and Metal installation output.
- **Source/hash/translator mismatch:** the engine files no longer match the expected version. Restore the matching workflow/engine revision; do not edit hashes or rename shader libraries to force a pass.
- **Timeout:** retry once if it looks like a temporary runner issue. Repeated failures need the exact step error reviewed.

A note that Metal differs from the committed upstream lock is not itself a failed check. The workflow reports that difference while retaining its existing Xcode policy. Do not change versions merely to hide the note.

## The game or keys would not download

Common causes are an expired link, missing file, login requirement, sharing restriction, download quota or a file-host page that is not a direct download.

For the **game file**, check that the link works without an interactive login. If Drive blocks automated downloads, use another supported direct link for a dump you are authorized to use.

For **keys**, check the `Keys_URL` secret and its access/expiry. **Do not make your keys public or send them to someone troubleshooting the build.**

Game and key downloads run in parallel. “Finished downloading prod.keys” can appear beside a game-download error; it does not mean both succeeded.

“No update configured” and “No DLC configured” are normal when you intentionally left them out.

## Extraction failed or the title was not detected

- Use a complete, supported game package—not an HTML download page saved with a game-file extension.
- Check that compressed files and archives finished downloading and are not damaged.
- Supply the base game, not just its update or DLC.
- Use matching updates/DLC and appropriate keys from your own console.
- Leave **Profile override** blank unless you know which profile is required.

If you entered the base URL in the run form, enter any wanted update/DLC links there too. Saved update/DLC secrets are deliberately not used with a popup base URL.

## It says DEFAULT or provisional profile

This usually means the engine has no built-in profile for that game. The builder may create one automatically.

It is **not automatically an error**, but it is also not proof that the game will work. Do not enter a different game's profile to hide the message.

## Shader compilation failed or progress looks stuck

Open the first actual compiler error. A long shader task can remain on one stage without regular progress updates.

- Keep the selected engine and its shader tools together.
- For a clean test, leave shader cache off.
- Do not bypass compiler checks, disable validation or rename shader chains.
- A survey or a successful shader build does not guarantee correct graphics on the device.

If a task repeatedly stops at the same point, share that step's error and the reviewed build logs.

## Downwell or A Short Hike's patch check failed

For their integrated legacy workarounds, use:

- **Metal 3 + BC fallback** engine.
- **Build IPA** mode.
- **Shader cache off**.
- **Profile override blank**.

A source-hash failure or missing patch marker means the workflow cannot verify the intended patch. Do not remove the guard. These workarounds do not apply to unrelated games or modern-engine builds.

## The runner ran out of disk space

Large packages, extraction and shader compilation can require much more space than the original download size.

Check the disk-space messages. Separate RomFS makes the final IPA smaller, but does not guarantee low build-time disk use. Use a runner with enough storage and keep the workflow's existing cleanup safeguards enabled.

## The build finished, but I cannot find the IPA

- Confirm you selected **Build IPA**, not **Survey Only**.
- Read the run summary for GoFile and GitHub artifact links.
- GitHub downloads arrive as ZIP files; extract them to find the IPA.
- Temporary links/artifacts expire. Download them promptly.
- If publication was blocked by an IPA or patch check, resolve that error rather than bypassing it.

An unsigned IPA must still be installed/signed through a suitable method; downloading it is not installation.

## The app opens, then closes—or graphics are black/wrong

Now the problem is on the device, not necessarily in the downloader or build setup.

- Confirm you built for the intended device/engine.
- If using separate RomFS, follow the run summary and place the matching `romfs.img` in the correct game's Documents folder.
- Check for a new guest/crash log from that attempt.
- Keep your previous working IPA and back up saves before replacing the app again.

Do not assume every crash is low memory or a signing problem. The log is needed to distinguish those from graphics, shader or game-specific failures. Avoid random diagnostic flags as a first fix.

## Controls or settings are different from another build

Options depend on the engine and workflow. The advanced layout editor currently targets **NX2Apple**; it is not identical across all engines. Check the workflow, branch and engine used to create the IPA before assuming a setting has disappeared.

## What to send when asking for help

Send:

1. The run/job link, branch and selected engine.
2. The first failed step's name and its final error lines.
3. For a runtime problem, the device model, OS, installation method and **new** guest/crash log.
4. Relevant build logs, after reviewing them for private information.

Do not send keys, private download links, complete environment dumps or unreviewed raw logs. Automatic log redaction is helpful but not a guarantee.

## Return to a working version

Restore the previous workflow files on the affected branch and start a new build, or reinstall your known-working IPA using your usual method. Keep backups and do not merge entire experimental branches just to recover one fix.
