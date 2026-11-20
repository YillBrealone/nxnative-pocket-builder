# Install the two-workflow update

This is a prepared repository update, **not a GitHub push**. It is based on `YillBrealone/nxnative-pocket-builder` commit `ccb42165e20b7c473297cd3d9a1ef91be600d423`.

## What changes

- Keep **NxNative - Generic IPA Builder** and **NxNative - Controls Editor Test**.
- Delete the standalone **Final Fantasy Disk-Saving Test** workflow. Guarded Final Fantasy cleanup remains in the other two workflows, for the audited NX2Apple paths.
- Add settings dependencies/notes and explicit-override protection consistently across menu implementations.
- Add **Motion controls (Next launch)** and **Recenter Phone Motion** in Controls. Recentring preserves tilt and does not take motion away from a controller. No new held-button shortcut.
- Add experimental **Auto / 30 fps** for the Metroid Dread profile on NX2Apple and Standard only. Existing generic higher-target choices remain for other profiles; legacy frame pacing is unchanged.
- Pin the moving Standard engine to the already-audited Controls Test revision, pin Actions commits, disable persisted checkout credentials and verify Xcode 27.1.
- Run source-hash guards and a CPU/source regression suite in both workflows before continuing to survey/build.

No new pre-game menu, Toggle/Turbo, diagnostic env.txt flags, memory trimming or texture-conversion policy changes. No Labyrinth shader capture is included in these production workflow replacements; keep that temporary diagnostic on its own branch.

## Option A — GitHub website (phone/tablet friendly)

1. Create a branch from your current `main`, for example **`pocket-settings-test`**.
2. On that branch, replace these two files with the complete files in this package:
   - `.github/workflows/build.yml`
   - `.github/workflows/nxnative-ipa-builder-controls-test.yml`
3. **Explicitly delete `.github/workflows/disk-test.yml` on that branch.** Uploading a ZIP or the other files does not delete it automatically.
4. Replace `README.md` and `TROUBLESHOOTING.md` with the updated documents. `audit/` is optional reference material; the YAML files are self-contained and do not depend on it.
5. Commit the changes to the test branch. Keep your existing secrets, including `Keys_URL`; do not upload them.
6. Open Actions and choose Main or Controls Editor Test. In **Run workflow**, select **`pocket-settings-test`**. Start a **new run**, not Re-run jobs on an older run.
7. For the first real build use a game you already know works, your usual appropriate engine, **Build IPA**, **Bundle**, **shader cache off**, and a blank profile override. Preserve the same update/DLC inputs used in your baseline.
8. Check that the new **Apply guarded settings…** and **Test settings and CPU motion regressions…** steps pass, then let the normal IPA build finish.
9. Back up saves before installing. Test the checklist below. If it passes, merge the branch to main. There will then be two active workflow files. GitHub may still show historical runs for the retired workflow.

These are complete replacements, not YAML snippets. Do not append them to the previous files. Do not select an unsupported engine merely to obtain a feature.

## Option B — Git command line

The separate `pocket-builder-two-workflow-update.patch` applies replacements, documentation updates and the Final Fantasy workflow deletion together.

From a clean checkout of your repository at the base revision above:

```bash
git switch -c pocket-settings-test
git apply --check /path/to/pocket-builder-two-workflow-update.patch
git apply /path/to/pocket-builder-two-workflow-update.patch
git diff --stat
git add .github/workflows/build.yml .github/workflows/nxnative-ipa-builder-controls-test.yml .github/workflows/disk-test.yml README.md TROUBLESHOOTING.md
git commit -m "Update main and controls settings, motion and build checks; retire disk test"
git push -u origin pocket-settings-test
```

If `git apply --check` reports a mismatch, stop: do not force it over a newer repository revision. The patch intentionally excludes the optional audit copies and these installation notes.

## Device checklist

Test Main and Controls Test separately; they retain different NX2Apple pins.

- Immediate game startup; no new Play screen.
- English host labels; nested Display, Controls, Performance and Other Settings.
- 2D, 3D, Tap and Trackpad selection; taps still go to the touch location.
- Saved choices persist after a full app restart.
- Dependent settings follow their parent option; notes fit on iPhone and iPad menus.
- Motion Controls turns gyro input off/on after restarting, without changing the independent shake behaviour.
- Recenter Phone Motion is enabled with fresh phone motion and disabled with a real motion-providing controller in automatic mode. Verify heading behaviour in a game that reads orientation; it is not a guaranteed camera-centre command.
- Controls Test / NX2Apple: editor opens in 2D/3D; resize, opacity, split ABXY, saved layouts and right-stick sensitivity remain; combinations release on finger-up with no Toggle or Turbo.
- If testing the Metroid Dread cap, compare the same scene at Auto and 30 after restarting: animation/game speed, frame pacing and device temperature. The cap is not enabled by default and is not added to legacy.
- Updates, DLC, saves and (in a subsequent separate-mode test) matching IPA/RomFS still work as before.

## Verification and limits

The complete embedded patch/check steps passed local execution against all six reconstructed source variants. The ten-check suite produced **53 passes and 7 not-applicable skips**. Added C motion code and the extracted modern FPS function compiled and ran under a mocked CPU harness. Changed-source and repeated-application guards rejected all six variants. YAML, Bash and embedded Python syntax checks passed.

**No full Xcode/UIKit build, GitHub Actions run or iPhone/iPad gameplay test was performed for this update.** The next run is the integration test, not a promised working IPA. Source checks cannot establish battery life, heat, smoothness or game compatibility.

If Xcode 27.1 is no longer available on the runner, or a source hash fails, send the first error. Do not remove the guard to force success.

## Rollback

Before merging, return to your unchanged main branch. After merging, revert the update commit(s) to restore the previous workflows. Back up app saves/preferences separately; reverting repository files cannot undo app data changes.
