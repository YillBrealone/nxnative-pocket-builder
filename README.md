# nxnative IPA Builder

Build an unsigned iPhone or iPad IPA from your own Nintendo Switch game files using GitHub Actions. You can start a run from your phone or tablet; GitHub provides the Mac runner.

**A successful survey or IPA build does not guarantee that a game will run.** Results depend on the game, update, engine, device, OS and installation method. This is an unofficial community project, not an Apple or Nintendo product.

## Choose a workflow

| Workflow in Actions | File | Use it for |
| --- | --- | --- |
| **NxNative - Generic IPA Builder** | [build.yml](.github/workflows/build.yml) | Normal builds without the experimental advanced controller editor. |
| **NxNative - Controls Editor Test** | [nxnative-ipa-builder-controls-test.yml](.github/workflows/nxnative-ipa-builder-controls-test.yml) | Testing advanced touch editing on the NX2Apple engine and the added right-stick touch sensitivity setting. |


These workflows are not interchangeable snapshots. The main and controller-editor workflows include guarded Final Fantasy cleanup and newer separate-RomFS download handling; the dedicated disk test does not automatically inherit every change. Engine revisions can also differ.

## Before you start

You need:

- A GitHub account and a repository with Actions enabled.
- Your own legally obtained game packages and, for Switch 1 games, your own matching `prod.keys`.
- Google Drive links or direct HTTP/HTTPS links that the runner can download.
- A device and OS supported by the selected engine, and a compatible way to sign/install or run the unsigned IPA.
- Enough storage for the downloads, installation and game data.

Inputs accept `.nsp`, `.xci`, `.nsz`, `.xcz` and `.nup`; the downloader also attempts supported RAR, ZIP and 7Z archives. Accepted input formats do not imply game compatibility. Switch 2 mobile support is experimental; combining separate Switch 2 base, update and DLC packages has not been verified.

## Set it up and run

1. Fork this repository and enable Actions. Manual workflows must be present on the default branch to appear in the run menu.
2. Make your game, update and DLC packages downloadable. A supported Google Drive folder can contain multiple packages, such as DLC.
3. Open **Settings > Secrets and variables > Actions > New repository secret**. Add `Keys_URL` containing the download link to your own `prod.keys`.
4. Open **Actions**, choose a workflow from the table above, then select **Run workflow**.
5. Choose the engine and build mode; provide the base game and any matching update/DLC links. Leave `profile_id` blank for automatic detection unless you know the correct override.
6. Open the run to follow progress. Read its summary and download the outputs before they expire.

Optional repository secrets: `Game_URL`, `Update_URL` and `DLC_URL`. When you enter a base `game_url` in the run form, the update and DLC fields in that form are used instead of old update/DLC secrets. Leaving them blank omits those secret inputs.

**Never put keys or a keys link in the run form, repository, issues or shared logs.** Use secrets for sensitive links. The runner still needs permission to download them; storing a publicly accessible URL in a secret does not make its hosting private. Forks do not receive your secrets. Only give repository write/workflow access to people you trust.

## Choose an engine

| Engine option | Starting point | Important limitation |
| --- | --- | --- |
| **NX2Apple Patched (Default)** | Supported newer iPhones and Apple Silicon iPads, including iPad Air M2 | The form's iPhone-focused label does not exclude compatible iPads. Advanced editor is available only through the Controls Editor Test workflow. |
| **Standard** | Supported newer iPhones and iPads using upstream nxnative | No advanced layout editor in these workflows. |
| **Metal 3 + BC fallback** | Experimental older-device path, including the A12Z iPad Pro | Not a guarantee of compatibility; texture fallback can increase memory use. No advanced layout editor. |

Check the [NxNative requirements](https://nxnative.org/requirements/) and selected engine's documentation for OS and hardware support. Upstream standard requirements describe iOS/iPadOS 26 or later with A17 Pro-class supported iPhones or M1-or-newer iPads; forks and pinned revisions can differ. Do not assume a currently advertised upstream feature exists in a pinned engine.

An iPad Air M2 should not use the older-device fallback merely because that option mentions iPad. Conversely, do not switch an unsupported older device to NX2Apple just to obtain the editor.

## Survey first or build directly?

For an unconfirmed game, **Survey Only** is a useful first step. It runs the engine's survey path without building an IPA and may expose input or analysis problems before a longer build.

- It does not run the game on your device or verify graphics, sound, saves or stability.
- A profile being present is not proof of playability. A missing profile is not automatically incompatibility; the engine may generate a provisional one.
- The form's “Fast 2-min” wording is not a total-runtime guarantee. Setup, downloads and analysis take additional time.
- To build afterward, start a new run with **Build IPA**. Do not assume the survey's temporary files carry into that run.

## Bundle or separate RomFS

### Bundle

Requests an all-in-one IPA with bundled game data. This is the simpler installation path, but can produce a very large IPA. Separate-RomFS staging and upload are skipped; game data still needs to be processed internally.

### Separate

Requests an IPA without bundled base/update RomFS. The workflows stage and upload the separate image **before shader compilation**, then verify the staged image during the resumed build.

1. Download the matching IPA and RomFS from the **same run**.
2. Extract the GitHub RomFS ZIP if using that fallback, and name the image exactly `romfs.img`.
3. Put it in that game's app data container at `Documents/romfs.img`, where supported by the engine.
4. For LiveContainer, use the imported game's own data container, not simply LiveContainer's top-level Documents folder.

Do not mix images between games, updates or unrelated runs. DLC is handled separately by the engine; the base/update RomFS artifact is not a complete DLC package. A RomFS upload can succeed even if the later IPA build fails.

## Downloads and retention

| Output | Main / Controls Editor Test | Final Fantasy Disk-Saving Test |
| --- | --- | --- |
| IPA GitHub artifact | 1 day | 1 day |
| Separate RomFS GitHub artifact | 1 day | 1 day |
| IPA GoFile page | Attempted, about 5-hour folder expiry | Attempted, about 5-hour folder expiry |
| Separate RomFS GoFile page | Attempted in separate mode, about 5-hour folder expiry | Not implemented |
| Redacted build-log artifact | 7 days | 7 days |

Download a GitHub artifact ZIP, extract its contents, and sign/install the IPA using your compatible method. GoFile links open download pages; they are not guaranteed direct file URLs. See the run summary for the confirmed expiry and fallback links.

**Separate RomFS is uploaded early. Its GoFile link may expire before a long shader build finishes.** Use the one-day GitHub fallback if necessary. Expired/deleted outputs cannot be recovered just from the old link; a fresh run may be needed.

To disable GoFile uploads, set the repository Actions **variable** `GOFILE_UPLOAD` to `false`. This does not disable GitHub artifacts.

**Outputs are not private storage.** Public-repository artifacts and GoFile pages may be accessible to others; bundled IPAs and RomFS images contain game data. Expiry cannot revoke copies already downloaded. Log-artifact retention does not mean every copy of Actions console logs is deleted on that schedule.

## Controls

Use **... > Control Mode** to choose **2D**, **3D**, **Tap** or **Trackpad**. Tap and Trackpad are intended for Handheld mode. Trackpad maps dragging to the left stick and a short tap to A; it does not provide every button a game may require.

### Controls Editor Test workflow

| Feature | NX2Apple | Standard / older-device |
| --- | --- | --- |
| Drag individual controls and resize them in the advanced editor | Yes | No advanced editor |
| Adjust editor opacity and split ABXY into separately editable buttons | Yes | No advanced editor |
| Save custom editor layouts in the game's app preferences | Yes | No advanced editor |
| Custom multi-button combinations | Up to 12 per layout set, hold-only | No advanced editor |
| Added right-stick touch sensitivity choices, 50–200% | Yes | Yes, source-adapted |
| Existing overall control-size and left-stick placement settings | Retained | Retained |

Open **Controls > Edit Control Layout** in 2D or 3D mode on NX2Apple. The editor is not offered in Tap or Trackpad. It supports global/per-control size, opacity, reset, cancel and done. Changing the underlying environment layout/size settings can reset saved layout/size edits; do not treat app preferences as a portable layout backup.

The newer editor patch adds single-line label handling, capped text scaling and adaptive card-width calculations. **Combo buttons work only while held. Toggle and Turbo are not offered.** No new show/hide or tap-to-hold feature is included. Touch sensitivity does not change physical-controller stick sensitivity.

Standard and the older-device engine lack the editor's required geometry integration; the older-device engine also uses a different menu system. Supporting them needs a separate port, not just removal of a workflow condition.

## Defaults and testing status

The workflows aim to start games immediately, with **Handheld** and **UK English** defaults. Available game languages vary; the language input can override the default. Host-source checks do not guarantee every game/resource is English.

The controller editor, readability update and cross-engine sensitivity integration have passed source-level checks, not a new complete Xcode build/device test. Other guarded workflow changes also require real-run verification. Do not interpret source checks, an existing profile, a successful survey or an uploaded IPA as proof of gameplay.

Shader cache is off by default. Enabling it can store game-derived data accessible to readers of a public repository and can use substantial runner/cache space.

## Help and credits

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md). Review logs before sharing: redaction is best effort. Back up saves before updating or reinstalling.

Built on [seedds/nxnative](https://github.com/seedds/nxnative), [LNPANDY/NX2Apple](https://github.com/LNPANDY/NX2Apple), [widmonstertony/nxnative](https://github.com/widmonstertony/nxnative), the controller editor from [bagasromadon/NX2Apple](https://github.com/bagasromadon/NX2Apple), and community contributions. Use your own legally obtained game files and keys.
