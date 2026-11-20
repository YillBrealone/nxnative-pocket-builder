# Troubleshooting

[Back to README](README.md)

Start with the **first failed step** in the Actions run, not just the final summary. A successful survey or IPA build does not guarantee gameplay.

Before sharing logs or screenshots, remove game links, keys links, tokens and private information. Never share `prod.keys`. Redaction is best effort; review even the `NxNative-Build-Logs` artifact.

## The workflow is missing or will not start

- Enable GitHub Actions in your own repository/fork.
- Check that the workflow is under `.github/workflows/` on the default branch.
- Choose the intended Actions entry: Generic IPA Builder or Controls Editor Test. The separate Final Fantasy workflow is retired.
- Check the selected branch and your permissions.
- On iPad, try Safari's Request Desktop Website or another browser/device if the Run workflow popup fails.

**Re-run jobs uses the original run's workflow definition.** After committing an updated workflow, start a new run to use it.

## A download or extraction fails

- Confirm the upload has finished and the runner can access it without an interactive login, CAPTCHA or browser-only confirmation.
- A direct URL must download the file, not an HTML sharing page.
- A supported Google Drive folder must expose the intended packages.
- For Switch 1 packages, check the `Keys_URL` repository secret and that it points to your own appropriate keys file.
- Check the archive is complete and contains supported packages for the same game.

A download, keys or extraction error does not prove game incompatibility. Never paste private URLs into issues.

## An update or DLC is missing

Supply the matching update and DLC packages when building. Supported Drive folders can contain multiple packages.

When a base game URL is entered in the run form, the form's update/DLC fields are used instead of old update/DLC secrets. Blank fields do not automatically bring those secrets back in.

The separate base/update RomFS image is not a complete DLC package. DLC handling remains engine-dependent. Switch 2 `.nup` support does not establish that combining separate base/update/DLC packages works.

## Title ID or profile errors

Include the complete base game. Leave `profile_id` blank for automatic detection unless you know the correct override. Do not select another game's profile merely to get past an error.

A profile directory or README is not certification that the game works. A provisional profile means the engine is attempting an unverified configuration, not that the game is definitely incompatible or definitely supported.

## Should I run Survey Only first?

It is a useful first step for an unconfirmed title. It does not create an IPA or test the game on your iPhone/iPad. Passing cannot confirm graphics, sound, saving or stability.

The “2-min” label is not a total-runtime promise. Downloads and setup add time. After reviewing the survey, start a new **Build IPA** run; do not assume the temporary survey data persists into it.

If the survey fails, distinguish input/download errors from engine analysis errors before drawing conclusions about compatibility.

## Which engine should an M-series iPad use?

For a supported M-series iPad, such as iPad Air M2, NX2Apple is the default starting point, subject to the selected revision's OS requirements. Its iPhone-focused option label does not exclude supported iPads. Standard is an alternative upstream path.

Do not select older-device BC fallback simply because its label mentions iPad. Do not select NX2Apple on an unsupported older device just to obtain the editor. See the engine requirements linked in the README.

## Shader compilation fails or reports a chain mismatch

Capture the first error, engine revision and expected/actual shader identifiers from the redacted logs. Keep the game/update versions and cache setting in your report.

**Do not rename shader libraries, copy unrelated chain headers, force a version identifier or bypass validation.** The runtime and generated shaders must agree. If an early check stops the job, that is a safeguard, not an instruction to suppress it.

Both workflows pin all engine selections and retain the existing shader-chain checks. NX2Apple revisions intentionally differ between Main and Controls Editor Test; see the README.

Only enable shader cache deliberately. It is off by default; cache reuse is not guaranteed and game-derived cache data may be accessible in a public repository. A warning about an unreadable shader is not by itself a diagnosis of a later crash.

## The runner runs out of disk space

Large games can need much more space than the compressed input: downloaded packages, extraction, shader intermediates and IPA packaging coexist. Separate mode reduces bundled IPA size but does not eliminate all runner storage requirements.

Main and Controls Editor Test contain guarded cleanup for Final Fantasy title `0100606026F12000` on their audited NX2Apple paths. It is not a universal cleanup for all titles or engines. Unknown script revisions are rejected rather than deleting unchecked paths. Shader intermediate cleanup is conditional, including cache-disabled requirements.

The separate Final Fantasy workflow has been retired. Use Main or Controls Editor Test with the appropriate engine. Do not add broad deletion commands while shader compilation or packaging still needs the files.

## The IPA or RomFS download is missing or expired

- Survey Only never creates an IPA.
- Check whether the job reached the relevant upload step.
- An early RomFS artifact does not prove the later IPA build succeeded.
- IPA and separate RomFS GitHub artifacts are retained for **one day**. Download the ZIP and extract it.
- GoFile uploads request and confirm approximately **five-hour folder expiry**; check the summary for the actual expiry and whether the upload succeeded.
- Both workflows attempt separate RomFS GoFile uploads, with GitHub artifacts as the fallback.
- Separate RomFS uploads happen before shaders; their GoFile link may expire before a long build ends. Use the GitHub fallback while available.
- If GoFile is disabled with the repository variable `GOFILE_UPLOAD=false`, use GitHub artifacts. Disabling GoFile does not disable artifact uploads.
- GitHub can require sign-in and appropriate repository access for artifact downloads. An expired link does not restore the file; you may need a fresh run.

GoFile pages are not guaranteed direct file URLs. Public-repository artifacts and GoFile links are not private storage. Expiry cannot revoke downloaded copies. Build-log artifacts are retained for seven days; Actions console-log retention is separate.

## Where do I put separate RomFS?

Use the image and IPA from the **same run**. Extract the RomFS artifact ZIP if necessary and name the image exactly `romfs.img`, not `romfs.img.zip` or `romfs.img.img`.

For engines supporting this handoff, put it at **the game's own `Documents/romfs.img`**. With LiveContainer, this means the imported game's data container, not merely LiveContainer's outer Documents directory. How you access that container depends on your LiveContainer/sideloading setup.

Do not substitute an image from another game, update or unrelated run. DLC remains a separate engine concern. Bundle mode does not produce a separate RomFS download and normally does not require this placement step.

## The IPA installs but black-screens, crashes or has graphical errors

Check:

1. Exact device and OS against the selected engine's requirements.
2. Whether the installation/signing method supports this IPA.
3. Game and update version, engine revision and any upstream game notes.
4. For separate mode, the correct image, filename and game data container.
5. Whether failure occurs immediately, after a menu, during loading or during gameplay.

An unsigned IPA must be handled by an appropriate signing/container method. Successful packaging does not prove launchability. A provisional profile may still need game-specific work, and older-device/Switch 2 paths are experimental.

Do not infer from a black screen alone that the cause is memory, shaders, keys or signing. Do not install unrelated quick patches or diagnostic environment flags as a blanket fix. Keep the original logs and back up saves before testing changes.

## The advanced controller editor is missing

Check both the **workflow** and **engine**:

- Use **NxNative - Controls Editor Test**.
- Select **NX2Apple Patched**.
- Select **2D or 3D** in Control Mode, then open **Controls > Edit Control Layout**.
- Tap and Trackpad do not offer this editor.

Standard and the older-device engine do not currently have the advanced editor. They retain existing overall control-size/left-stick placement settings and the controller workflow's source-adapted right-stick sensitivity, but not the advanced drag/per-button resize/opacity/split-ABXY/custom-layout editor. This is a missing port, not a hidden setting.

The updated NX2Apple editor has readability changes intended to reduce wrapped button text and adapt card width. These source changes have not been verified in a new complete iOS build/device test.

## Combo buttons or saved layouts behave differently than expected

Combo buttons are **hold-only**: selected buttons remain pressed while your finger is down and release when you lift it. Toggle and Turbo are deliberately not offered. There is no new tap-to-hold or show/hide feature.

Use Done to save editor changes; Cancel discards the current edit session. Layouts are stored in the game's app preferences, not a guaranteed cross-device export. Changing underlying environment layout/size settings can reset saved layout/size edits. Reinstalling into a new app container can also lose preferences; back up saves and data first.

The right-stick sensitivity setting changes virtual touch-stick gain, not physical-controller sensitivity.

## Tap or Trackpad does not provide enough controls

Tap sends touchscreen input where supported; not every game uses a touchscreen. Trackpad uses dragging for the left stick and a short tap for A. It is not a complete substitute for all gamepad buttons, and it does not gain editable overlay combo buttons from the NX2Apple editor.

Try 2D/3D controls or a supported physical controller if the game needs additional buttons. Tap/Trackpad are intended for Handheld mode. Their behaviour has not been verified for every game/device.

## Rebuild without losing saves

Back up saves and game data before replacing an IPA. Updates and DLC generally require rebuilding. With LiveContainer, preserve the correct existing app identity and data container when appropriate; installing as a new entry can leave saves in the old container. Backup and save compatibility are not guaranteed by a successful build.

## The toolchain or Pocket source guard fails

The update requires Xcode 27.1 and the exact source sets produced by its pinned engines plus the existing workflow patches. A moving runner image, edited patch, changed engine pin, or repeated application can trigger a stop. Do not remove the hash/version checks to force a build. Report the first error and workflow revision; a changed source needs review and updated tests.

The CPU motion tests do not need game data or Metal. If they fail, keep the first test/compiler error. They are not a replacement for the later iOS build.

## Recenter Phone Motion is disabled or does not move the camera

Motion controls must be enabled, and the phone must be delivering fresh gyro samples. In automatic-source mode a live controller takes precedence. Hold the phone in its normal playing orientation; an almost vertical heading axis can be ambiguous. Reopen the menu to refresh availability.

The action resets the host phone-heading reference, not a game's own camera or gyro calibration. Some games respond only to rotation rate, so a heading reset may have no visible effect. Physical-controller recentering and an L3+R3 shortcut are not included. Device validation is still needed.

## Auto / 30 fps is missing

The added cap is deliberately limited to Metroid Dread's profile on NX2Apple and Standard, and is not a universal older-device cap. Use the proper profile for your game; never override it with another title's ID to expose an option. Other profiles retain their existing frame-rate controls. The new cap applies on the next launch, not immediately.

## A setting is disabled or hidden

A setting can depend on another option, such as rumble strength requiring rumble. It can also be locked by an explicit launch/environment override recorded by the engine. Ordinary saved choices and defaults are not locks. The update does not add diagnostic files or flags to your app.

## Asking for help

Include:

- Workflow name and the run's workflow/engine revision, if available.
- Device and OS version.
- Game and update version; title ID if known.
- Engine, bundle/separate mode and shader-cache setting.
- First failed step and relevant redacted error lines, or the exact runtime failure point.
- Whether an IPA was built and how it was installed.

Do not include keys, private links, tokens, game packages, RomFS images or unreviewed logs.
