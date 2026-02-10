# Game Compliance Pack

Unity Editor tool for generating third-party compliance files from your project, with a workflow to resolve unknown licenses without guessing.

## What it generates

By default, exports these files into a `Compliance/` folder at your project root:

- `THIRD_PARTY_NOTICES.txt`
- `CREDITS.md`
- `compliance-manifest.json`
- `compliance-overrides.json` (your manual fixes and notes)

Outputs are generated in a stable order so diffs stay clean.

## Quick start

1. Put third-party assets in one of the scanned folders (defaults):
   - `Assets/Plugins`
   - `Assets/ThirdParty`
2. Open the tool: `Tools > Compliance Pack`
3. Click `Scan` -> `Scan All`
4. Resolve anything flagged as **UNKNOWN** or "missing required fields":
   - Select a component and fill in the missing info in the right panel
   - Use `Resolve Next` to jump through unresolved items
   - If the license is known, pick an SPDX ID (MIT, Apache-2.0, etc.)
   - If the license is not an SPDX ID, keep it `UNKNOWN` until you can confirm it, or use a `LicenseRef-*` option and paste the license text
5. Click `Export Pack` to write the files to disk

Tip: the `Diff` tab compares the current scan to the previous `compliance-manifest.json` (from the output folder), so you can see what changed.

## Resolving UNKNOWN (recommended workflow)

The tool intentionally does **not** guess license IDs.

When a component is `UNKNOWN`, do one of these:

- **Find the upstream license** (recommended):
  - Open the component's `LICENSE`/`COPYING`/`NOTICE` file (right-click a row -> `Open License File`)
  - If it matches an SPDX license, set the SPDX ID so the pack can use the canonical SPDX text
- **Use a LicenseRef when it's not SPDX**:
  - Choose a `LicenseRef-*` option and paste the full license text you were given
- **Keep it UNKNOWN until confirmed**:
  - If you cannot confirm the license yet, leave it as `UNKNOWN` and resolve it later (and consider enabling build warnings/failures; see below)

All manual edits are stored in `compliance-overrides.json`.

## Manual folders (`+ Manual`)

Use `+ Manual` to add a folder entry when:

- a dependency is not under your scanned folders,
- you have assets with no machine-readable license files,
- you need to add credit/attribution text exactly as required by the author.

Manual entries are saved into `compliance-overrides.json` and included on export.

## Build-time behavior (pre-build hook)

On every build, the pack can automatically scan/export and then warn or fail the build if anything needs attention.

Key settings:

- `Generate On Build` (default: on)
- `Warn On Unknown` (default: on)
- `Fail Build On Unknown` (default: off)
- `Fail Build On Non-Commercial` (default: on) - blocks known non-commercial licenses (for example, CC licenses with `-NC`)

GPL/AGPL warning: If a GPL or AGPL component is detected, the tool will show a warning that these licenses require
distributing corresponding source code (or a written offer) to recipients. Listing the license in
`THIRD_PARTY_NOTICES.txt` is not sufficient by itself.

## Configuration

Settings live at `Assets/Settings/CompliancePackSettings.asset` and are also editable in the tool window.

Common options:

- Output folder and file names (defaults shown in "What it generates")
- Whether to scan UPM dependencies (`Packages/manifest.json` + `packages-lock.json`)
- Which folders to scan and which native extensions to treat as dependencies (`.dll`, `.so`, `.dylib`, `.bundle`)
- License detection confidence threshold

## Notes and tips

- Right-click a component row for helpful actions (reveal in Explorer, open license file, reset override, mark ignored).
- After importing a `.unitypackage`, the pack may prompt you to move the imported folders under your scan root (so they are included in future scans).
