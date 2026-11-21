# NxNative Pocket Builder

Build an unsigned iPhone or iPad app (`.ipa`) from your own game dumps using GitHub Actions. The build runs on a Mac runner, so you can start it from your phone, tablet or computer.

**A successful build does not guarantee that a game will run correctly.** Results depend on the game, engine and device.

## What you need

- A repository containing these workflows, with GitHub Actions enabled.
- Access to the `xcode-27` runner used by the workflows, with Xcode **27.1** available.
- Your own game dump and any matching update or DLC.
- Your own `prod.keys` where required, supplied through the `Keys_URL` repository secret.
- A suitable sideloading/signing method, such as your existing LiveContainer setup.

Supported input formats include **NSP, NSZ, XCI, XCZ and NUP**. The builder also handles supported RAR archives containing game packages. Accepting a file format does not guarantee game or console-generation support.

## One-time setup

1. Put the two YAML files in your repository's **`.github/workflows/`** folder.
2. Open **Settings → Secrets and variables → Actions → New repository secret**.
3. Add **`Keys_URL`**: a download link to your own `prod.keys` that the runner can access without an interactive login.
4. Optionally add **`Game_URL`**, **`Update_URL`** and **`DLC_URL`** for saved game links.

**Never commit keys or game files to the repository.** Keep key links in secrets, and do not share them publicly. Workflow input fields and downloadable artifacts should not be treated as private storage.

If installing from the supplied ZIP, use the `main/` files on Main and the `pocket-settings-test/` files on Pocket. Each folder contains `build.yml` and `nxnative-ipa-builder-controls-test.yml`. Replace complete files; do not merge the branches. You can place this README and the troubleshooting guide at the repository root.

## Choose a workflow and engine

### Workflows

- **Generic IPA Builder:** your regular build workflow.
- **Controls Editor Test:** the separate controls/editor build. The advanced layout editor currently targets NX2Apple; options differ between engines.

Use **Main** for normal builds. **Pocket** is the separate settings/testing branch; it is not required for every build.

### Engines

| Selection | Purpose |
|---|---|
| **NX2Apple Patched — Default** | Patched engine for the newer devices named in the dropdown. |
| **Standard** | Alternative standard engine for the newer devices named in the dropdown. |
| **Metal 3 + BC fallback** | Legacy option for the 2020 A12Z iPad Pro and older devices covered by that selection. |

Choose for the device that will run the IPA, not the device you use to open GitHub. These labels are not a promise that every game works.

## Build an IPA

1. Open **Actions** and select a workflow.
2. Click **Run workflow** and choose the branch.
3. Select your engine and **Build IPA**.
4. Enter the base game URL and, if wanted, matching update/DLC URLs.
5. Use the recommended settings below, then start the run.

Google Drive and direct HTTP/HTTPS downloads are supported. A normal file-host page is not always a direct download link.

**How saved links work:** entering a base game URL uses the popup's game/update/DLC fields as a set, so old update/DLC secrets are not mixed in. Leaving the base URL blank uses `Game_URL`; update/DLC fields can then override their saved secrets.

| Setting | Recommended starting point |
|---|---|
| Build mode | **Build IPA**. Survey Only makes a preliminary report, not an IPA or a guarantee of compatibility. |
| RomFS | **Bundle** for the simplest installation. |
| Shader cache | **Off**, especially for a first test. It must stay off for the integrated Downwell and A Short Hike legacy workarounds. |
| Language | Leave blank for the workflow's UK English default, or enter a language code. Game language availability varies. |
| FPS display | Your preference; the counter does not increase performance. |
| Profile override | Leave blank unless you have a specific reason to change it. |

The workflow checks its tools, downloads and extracts the packages, builds shaders and creates the IPA. Watch the current step for progress. Some large games or individual shaders take a long time.

## Download and install

After a successful run, open its summary:

- Use the temporary **GoFile download**, when available. Its configured lifetime is five hours.
- The **GitHub IPA artifact** is the fallback, retained for one day. It may require sign-in; extract the downloaded ZIP to get the IPA.

The IPA is **unsigned**. Install/sign it using your chosen method. Back up saves and keep a working IPA before replacing an app.

### Bundle or separate RomFS?

*RomFS is the game's data.*

- **Bundle:** includes the game data in the IPA. This is the easiest option.
- **Separate:** produces a smaller IPA and a separate RomFS download. Follow the run summary: extract the image, name it `romfs.img`, and place it in that game's Documents folder. The exact location depends on your sideloader.

IPAs and RomFS files can contain game data. Anyone with access to the published downloads may obtain it. To disable GoFile publication, set the repository Actions variable **`GOFILE_UPLOAD`** to **`false`**; GitHub artifact uploads remain enabled.

## In-game controls and settings

Use the in-game menu for the options available in your selected engine. The workflow preserves English host menus, handheld defaults, display/control settings and supported touch modes. The legacy engine includes ASTC conversion for eligible BC textures; no extra flag is needed for its default behavior.

You do not need diagnostic `env.txt` flags for normal builds. Host menu language does not change text that is unavailable in the game's own language files.

## Need help?

Read **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)**. Start with the first failed step and keep your keys and private links out of screenshots and shared logs.
