# nxnative IPA Builder

Build an unsigned iPhone or iPad IPA from your own Nintendo Switch game files using GitHub Actions. You can start a run from your phone or tablet; GitHub provides the Mac runner.

**A successful survey or IPA build does not guarantee that a game will run.** Results depend on the game, update, engine, device, OS and installation method. This is an unofficial community project, not an Apple or Nintendo product.

## Choose a workflow

| Workflow in Actions | File | Use it for |
| --- | --- | --- |
| **NxNative - Generic IPA Builder** | [build.yml](.github/workflows/build.yml) | Normal builds without the experimental advanced controller editor. |
| **NxNative - Controls Editor Test** | [nxnative-ipa-builder-controls-test.yml](.github/workflows/nxnative-ipa-builder-controls-test.yml) | Testing advanced touch editing on the NX2Apple engine and the added right-stick touch sensitivity setting. |

There are now **two workflows**. The separate Final Fantasy disk-test workflow has been retired: guarded Final Fantasy cleanup remains integrated in both workflows, on the audited NX2Apple paths. It is not a general cleanup for all games or engines.

The workflows deliberately retain their existing NX2Apple revisions; Standard and the older-device pins are shared:

| Engine | Main | Controls Editor Test |
| --- | --- | --- |
| NX2Apple | `3739300556114bcb51937a62c7392464b1196939` | `4ab27a53ccb25545fc72dde841320ce5010c399d` |
| Standard | `20b59b6e59a9b8f060c19cd3894dacf4e5f4d68a` | same |
| Older-device | `b5ac83de5c3c01ab30b3e0dbac8eaf32fe2f9b14` | same |

GitHub Actions references are pinned to commit hashes, and checkouts do not persist credentials. The runner must provide **Xcode 27.1**, the version recorded in the reference build; an unavailable/mismatched version stops early for review. Python/.NET and package-manager dependencies are not fully reproducibly locked by this update.

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

| Output | Both workflows |
| --- | --- |
| IPA GitHub artifact | 1 day |
| Separate RomFS GitHub artifact | 1 day |
| IPA GoFile page | Attempted, about 5-hour folder expiry |
| Separate RomFS GoFile page | Attempted in separate mode, about 5-hour folder expiry |
| Redacted build-log artifact | 7 days |

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

## Settings, phone motion and lower-power choices

Both workflows add the following source changes on all three engine choices unless noted:

- **Settings notes and dependencies:** explanatory notes appear in menus; dependent settings such as rumble strength follow their parent setting. Existing Swift filtering is preserved and the legacy Objective-C menu now observes the same dependency rule.
- **Explicit overrides:** settings the engine records as explicit launch/environment overrides are disabled in both menu paths. Ordinary defaults and saved choices remain editable. No diagnostic `env.txt` flags are added.
- **Motion controls:** a Controls setting enables/disables tilt and gyro input on the next launch. Shake input remains separate.
- **Recenter Phone Motion:** an immediate Controls action that zeroes the phone stream's heading while preserving its tilt. It is disabled when motion is off/stale, its heading is ambiguous, or a controller is supplying motion in automatic-source mode. It does not replace controller motion, reset a game's camera, or change the existing gyro mapping. No L3+R3 shortcut is installed.
- **Profile-limited Auto / 30 fps:** the Metroid Dread profile (`010093801237C000`) gains an experimental Auto / 30 fps choice on **NX2Apple and Standard**. Auto is the default. The reference fork reports normal game speed at 30, but this port needs device testing. Restart the app after changing the choice. The older-device frame-limit policy is deliberately unchanged.
- **Avoid duplicate frame-rate controls:** the profile-limited cap replaces that profile's generic target row. Where NX2Apple already supplies a game/display pacing row, the competing generic row is suppressed. Existing 30/60/120 target choices remain for other profiles; selecting 120 does not prove the game runs at 120 or keeps correct speed.

There is no new pre-game screen. Use the existing **Display / Controls / Performance / Other Settings** menus. The advanced layout editor remains exclusive to Controls Editor Test with NX2Apple; this update does not port it to other engines.

### Verification of this update

- Reconstructed the six exact source variants (two workflows × three engine choices), including NX2Apple patches and the Controls editor where applicable.
- Executed the actual embedded source-patching and check steps locally for all six variants.
- Ran a ten-check CPU/source suite on each variant: **53 checks passed, 7 were not applicable** (editor/legacy cap exclusions).
- Compiled and executed the added motion helper and extracted host bridge against a mock sensor source; checked 1,000 heading/tilt rotations per variant, stale/disabled sources and controller precedence. Modern variants also exercised the actual extracted FPS function's override precedence.
- Checked generated settings with synthetic profiles, duplicate frame-rate prevention, explicit override handling, English source labels, four control modes and hold-only editor markers.
- All six changed-source tests and all six duplicate-application tests were refused. YAML, Bash blocks and embedded Python syntax passed.

**These are not full Xcode/UIKit compilation or device/gameplay tests.** The sensor tests use mocks; the settings profiles are synthetic. A real Build IPA run and device checks are still required. The same CPU/source checks run in Actions before survey/build proceeds. Unknown post-patch source hashes stop the update rather than silently applying to changed engine code.

## Defaults and testing status

The workflows aim to start games immediately, with **Handheld** and **UK English** defaults. Available game languages vary; the language input can override the default. Host-source checks do not guarantee every game/resource is English.

The controller editor, readability update, sensitivity integration and this settings/motion update have not been verified in a new complete Xcode build/device test. Guarded workflow changes require real-run verification. Do not interpret source checks, an existing profile, a successful survey or an uploaded IPA as proof of gameplay.

Shader cache is off by default. Enabling it can store game-derived data accessible to readers of a public repository and can use substantial runner/cache space.

## Help and credits

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md). Review logs before sharing: redaction is best effort. Back up saves before updating or reinstalling.

Built on [seedds/nxnative](https://github.com/seedds/nxnative), [LNPANDY/NX2Apple](https://github.com/LNPANDY/NX2Apple), [widmonstertony/nxnative](https://github.com/widmonstertony/nxnative), the controller editor from [bagasromadon/NX2Apple](https://github.com/bagasromadon/NX2Apple), and community contributions. The settings/motion roadmap and profile-limited Auto/30 work also draw on [mianmuhammadse/nxnative](https://github.com/mianmuhammadse/nxnative/tree/ea80fe638538e9a3c393f65d5d85855df3d4f30a); the recenter helper is adapted to the existing engines rather than replacing their motion stack. Use your own legally obtained game files and keys.
