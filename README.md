# NxNative Pocket Builder

Build an unsigned iPhone or iPad app (`.ipa`) from your own game dumps using GitHub Actions. Builds run on a Mac runner, so you can start them from your phone, tablet or computer.

**Not every game will work.** A successful build does not guarantee correct graphics or gameplay on your device.

## What you need

- These workflows in a repository with GitHub Actions enabled.
- Access to the workflow's **`xcode-27` runner**, with Xcode **27.1** available.
- Your own game dump, matching updates/DLC, and keys where required.
- A suitable installation/signing method, such as your existing LiveContainer setup.

Supported package formats include **NSP, NSZ, XCI, XCZ and NUP**. Supported RAR archives can also be extracted. File-format support is not a promise of game or console-generation compatibility.

## Set up your repository

1. Put `build.yml` and `nxnative-ipa-builder-controls-test.yml` in **`.github/workflows/`**.
2. Open **Settings → Secrets and variables → Actions**.
3. Add **`Keys_URL`**: a download link to your own `prod.keys` that the runner can access.
4. Optionally save your game links as **`Game_URL`**, **`Update_URL`** and **`DLC_URL`** secrets.

**Never commit keys or game files to the repository.** Keep key links in secrets and do not share them publicly.

When using the supplied ZIP, use the `main/` pair on Main and the `pocket-settings-test/` pair on Pocket. Replace complete files; do not merge branches. Put this README and `TROUBLESHOOTING.md` at the repository root if wanted.

## Choose a workflow

- **Generic IPA Builder:** the regular build workflow.
- **Controls Editor Test:** the separate controls/editor build. The advanced layout editor currently targets NX2Apple; options vary by engine.

Use **Main** for normal builds. **Pocket** is the separate settings/testing branch.

## Choose an engine

| Engine selection | Use |
|---|---|
| **NX2Apple Patched — Default** | Patched engine for the newer devices named in the dropdown. |
| **Standard** | Alternative standard engine for those newer devices. |
| **Metal 3 + BC fallback** | Legacy option for the 2020 A12Z iPad Pro and older devices covered by that selection. |

Choose for the device that will **run the IPA**, not the device you use to open GitHub.

## Start a build

1. Open **Actions**, select a workflow and click **Run workflow**.
2. Choose the branch, engine and **Build IPA** mode.
3. Enter the base game URL and any matching update/DLC URLs.
4. Start the run and follow its progress.

Google Drive and direct HTTP/HTTPS downloads are supported. A file-host webpage is not always a direct download link.

**Saved links:** entering a base URL uses the popup's game/update/DLC fields together. Leaving it blank uses `Game_URL`, with saved update/DLC links unless you override them in the form.

### Recommended starting settings

| Setting | Choose |
|---|---|
| Build mode | **Build IPA**. Survey Only produces a preliminary report, not an IPA. |
| RomFS | **Bundle** for the easiest installation. |
| Shader cache | **Off** for a first test. Required off for the Downwell and A Short Hike legacy workarounds. |
| Language | Blank for the workflow's UK English default, or enter a language code. |
| FPS display | Your preference. The counter does not increase performance. |
| Profile override | **Blank**, unless you know which profile is needed. |

The workflow checks its tools, downloads and extracts files, builds shaders, then packages the IPA. Large games and individual shaders can take a long time.

## Download and install

Open the completed run's summary:

- **GoFile:** temporary download, when available; configured to expire after five hours.
- **GitHub artifact:** one-day fallback that may require sign-in. Extract its ZIP to get the IPA.

The IPA is **unsigned**. Install/sign it using your chosen method. Back up saves and keep a working IPA before replacing an app.

### What is RomFS?

RomFS is the game's data.

- **Bundle:** includes it in the IPA.
- **Separate:** provides a smaller IPA and a separate data download. Follow the run summary to place the matching image, named `romfs.img`, in that game's Documents folder.

Downloads can contain game data and are not private storage. To disable GoFile, set the repository Actions variable **`GOFILE_UPLOAD=false`**. GitHub artifact uploads remain enabled.

## In-game performance settings

Start with the settings that already work for your game. Change one option at a time and restart before comparing the same scene.

- **Render scale (experimental):** effect depends on the engine and game. It may change output size without changing internal rendering resolution.
- **Dynamic resolution (game-dependent):** a visible toggle does not prove support. Leave it off if gameplay is already smooth.
- **Frame generation (experimental):** try off first when troubleshooting. It can add artifacts or GPU work and does not increase the game's simulation speed.
- **Frame rate:** a higher target does not guarantee more FPS; it may change game speed or heat.

This update adds explanations, not new graphics behavior. Existing defaults, saved choices and engine-specific visibility rules are preserved.

## Build records and retention

IPA, separate RomFS, redacted build-log and build-environment artifacts are retained for **one day**. GoFile downloads are configured for **five hours**. These settings do not delete GitHub's own Actions run history or console logs; manage those separately in the repository.

The **NxNative-Build-Environment** artifact contains:

- `nxnative-build-environment.json`: observed tool/package versions and run/engine revisions.
- `nxnative-python-requirements.txt`: the builder's existing Python requirements.
- `nxnative-python-constraints-candidate.txt`: a **draft**, version-pinned snapshot of observed Python packages. It includes unrelated installed packages, has no package hashes, and is **not a tested lock**. It is never applied automatically.
- `nxnative-build-metrics.json`: completed dependency/build-step durations and cache outcomes, when available.

These records exclude keys, game URLs, environment dumps and package source URLs. Records can exist even when a later build step fails; check the final run result.

Save this record from successful **full IPA builds** before it expires. Dependencies are not yet fully locked: successful Mac-build records and repeat-build testing are needed before choosing exact versions. Engine and GitHub Action revisions are pinned.

## Build speed and caches

### Python package downloads

An isolated cache stores public Python-package downloads, including downloaded prebuilt wheels. Only pip's `http-v2` download cache is shared; locally compiled wheels are not shared. Every run still installs the requirements and performs dependency checks. The cache does **not** preserve an installed Python environment, Xcode, Homebrew tools, games or keys. Keys include requirements, workflow revision, Python/pip, macOS and architecture; there are no broad fallback keys.

This can reduce repeated setup, but savings may be small. Set the repository Actions variable **`PYTHON_DOWNLOAD_CACHE=false`** to disable it. A failed Python installation is retried once without using the cache; repeated failures still stop the build.

### Shader cache (optional)

Keep **Shader cache off** for initial testing and for the current Downwell/A Short Hike legacy workarounds. Enabling it can expose game-derived shader data through a public repository's cache. Never use it to bypass a failed build check.

To assess reuse for another working game:

1. Run a successful build with **Shader cache on** to populate the cache.
2. Start another run with the same workflow revision, engine, game/update/DLC files and options. Do not edit the workflow between runs.
3. Check **Cache and build timing** in the summary. Compare total job time as well as the IPA build-step time, and test the IPA on your device. A restored archive does not prove all shaders were reused.

The first run still needs to compile. Runner/tool changes may correctly produce a cache miss. A cache-off run provides an additional baseline if you want to measure whether caching helps overall.

**Cache retention is separate from artifact retention.** GitHub's cache lifecycle applies; the one-day artifact setting does not delete caches. Disabling a cache stops future use but does not delete existing entries. Manage those under Actions → Caches.

All of this can be operated through Safari on an iPhone or iPad; the Mac work runs on the Actions runner. No renderer, texture-quality or game-fix changes are made for these speed improvements.

## Controls and help

Use the in-game menu for the settings supported by your engine. Normal builds do not need diagnostic `env.txt` flags. English host menus do not guarantee English text in every game.

Having a problem? Read **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)**.
