# Game Compliance Pack for Unity

A local-first Unity Editor extension that scans your Unity project for third-party components and generates release-ready attribution artifacts (third-party notices, credits, and a machine-readable compliance manifest).

This repository is documentation and examples only. It does not include the commercial source code.

**Get the tool**
- Unity Asset Store: (add link)
- itch.io: (add link)

## What this solves (Unity license compliance at release time)

Unity projects routinely accumulate third-party software from multiple sources:
- Unity Package Manager (UPM) dependencies, including transitive dependencies
- Asset Store plugins and embedded third-party code
- DLLs and native plugins (.dll, .so, .dylib, .bundle)
- Copied utility folders and middleware

At ship time you need to:
- identify what is third-party
- find the actual license terms
- assemble correct notices and attributions
- keep the result stable across updates and builds

Without a system, this becomes manual, error-prone, and often happens too late.

## Core outputs (generated files)

By default, the tool exports these files into a `Compliance/` folder at your project root:

- `THIRD_PARTY_NOTICES.txt`  
  Full attributions plus license text when available.

- `CREDITS.md`  
  A short, human-readable credits block suitable for in-game credits or store pages (no full license dumps).

- `compliance-manifest.json`  
  Machine-readable source of truth for diffs, audits, and repeatable builds.

- `compliance-overrides.json`  
  Your manual fixes, attributions, license selections, and notes (committable so teams share the same decisions).

Outputs are generated in a stable order so diffs stay clean.

## Key design principles

- **Local-first**: scans your project on disk inside Unity (no uploading project data by default).
- **Deterministic**: stable component keys and stable ordering so diffs are meaningful.
- **No guessing**: if the license is not verifiable, it stays **UNKNOWN** until you resolve it.
- **Reviewable**: outputs are plain text and JSON so they are easy to audit.

This is not legal advice and does not guarantee compliance. It is a workflow tool that makes it harder to forget obligations and easier to review what you ship.

## How it works (pipeline)

1. **Scan** your project sources for third-party components
2. **Normalize** results into stable component entries
3. **Resolve** licenses conservatively (file evidence or explicit user choice)
4. **Apply overrides** from `compliance-overrides.json`
5. **Generate** the outputs
6. **Diff** against the previous `compliance-manifest.json` to detect changes

## What it scans

### Unity Package Manager (UPM)
- Reads `Packages/manifest.json` for direct dependencies
- Reads `Packages/packages-lock.json` for resolved direct and transitive dependencies
- Captures name, version, and source (registry, git, local path)
- Extracts license hints only from verifiable metadata or included files (never assumptions)

### File system plugins and embedded components
Scans common locations (configurable), typically:
- `Assets/Plugins`
- `Assets/ThirdParty`

Detects common binary formats:
- `.dll`, `.so`, `.dylib`, `.bundle`

### License file discovery
For each detected component, searches nearby paths for:
- `LICENSE`, `LICENSE.txt`, `LICENSE.md`
- `NOTICE`, `NOTICE.txt`
- `COPYING`, `COPYING.txt`
- `Third Party Notices`, `ThirdPartyNotices`

If found, the license or notice text is treated as strong evidence and can be used as the source of truth.

## Quick start

1. Put third-party assets in one of the scanned folders (defaults):
   - `Assets/Plugins`
   - `Assets/ThirdParty`
2. Open the tool: `Tools > Compliance Pack`
3. Click `Scan` (or `Scan All`)
4. Resolve anything flagged as **UNKNOWN** or "missing required fields":
   - Select a component and fill missing info in the right panel
   - Use `Resolve Next` to jump through unresolved items
   - If the license is known, pick an SPDX ID (MIT, Apache-2.0, BSD-3-Clause, etc.)
   - If the license is not an SPDX license, use a `LicenseRef-*` option and paste the full license text you were given
5. Click `Export Pack` to write the files to disk

Tip: the `Diff` tab compares the current scan to the previous `compliance-manifest.json` in your output folder so you can see what changed.

## Resolving UNKNOWN (recommended workflow)

The tool intentionally does not guess license IDs.

When a component is **UNKNOWN**, do one of these:

- **Find the upstream license** (recommended)
  - Open the component's `LICENSE` / `COPYING` / `NOTICE` file (row context menu or evidence links)
  - If it matches an SPDX license, select the SPDX ID so the pack can use canonical text

- **Use a LicenseRef when it is not SPDX**
  - Choose a `LicenseRef-*` option and paste the full license text

- **Keep it UNKNOWN until confirmed**
  - If you cannot confirm the license yet, leave it as UNKNOWN and resolve it later
  - Consider enabling build warnings or failures so UNKNOWN cannot slip into release builds

All manual edits are stored in `compliance-overrides.json` and should be committed to source control.

## Manual folders (`+ Manual`)

Use `+ Manual` when:
- a dependency is not under your scanned folders,
- you have assets with no machine-readable license files,
- you must add credit or attribution text exactly as required by the author.

Manual entries are saved into `compliance-overrides.json` and included on export.

## Build-time behavior (pre-build hook)

Optionally, the pack can automatically scan and export on every build, then warn or fail if anything needs attention.

Common settings:
- `Generate On Build` (default: on)
- `Warn On Unknown` (default: on)
- `Fail Build On Unknown` (default: off)
- `Fail Build On Non-Commercial` (optional, if you want to block known non-commercial licenses)

### GPL and AGPL warning
If a GPL or AGPL component is detected, the tool should warn that these licenses can require distributing corresponding source code (or a written offer) to recipients. Listing the license in `THIRD_PARTY_NOTICES.txt` is not sufficient by itself.

## Determinism and clean diffs

Determinism is a primary feature:
- stable component keys so a component is "the same" across scans
- stable ordering in all outputs
- no timestamps inside notices unless explicitly enabled
- sorted filesystem enumeration to avoid nondeterministic ordering

## Configuration

Settings live at `Assets/Settings/CompliancePackSettings.asset` and are also editable in the tool window.

Common options:
- output folder and filenames
- whether to scan UPM dependencies (`Packages/manifest.json` + `packages-lock.json`)
- which folders to scan, and which native extensions to treat as dependencies
- license detection confidence threshold
- ignore rules for known non-dependencies

## What this is not (non-goals)

- Not legal advice
- Not a guarantee of compliance
- Not a license guesser or AI inference tool
- Not a cloud service
- Not perfect detection of every possible third-party item in a Unity project

It aims to reliably catch the most common sources, surface UNKNOWN early, and make resolution fast and repeatable.

## Recommended repository contents (for docs and SEO)

If you are viewing this on GitHub, you should also include:
- `/docs/` (overview, workflow, outputs, FAQ)
- `/examples/` (sample `THIRD_PARTY_NOTICES.txt`, `CREDITS.md`, `compliance-manifest.json`)
- `/screenshots/` (Unity UI screenshots that show UNKNOWN resolution and outputs)

## FAQ (short)

**Does this guarantee compliance?**  
No. It generates reviewable artifacts and forces UNKNOWN visibility so you can make informed decisions.

**Why not just do this manually once?**  
Projects change. Deterministic scans plus diffs prevent regressions and last-minute surprises.

**Why keep UNKNOWN instead of best-guessing?**  
Guessing licenses is how teams ship incorrect notices. UNKNOWN is a deliberate "stop and verify" state.

---

If you want to improve this README further, add links for your store pages, include 2 to 4 screenshots, and commit realistic examples under `/examples/`.
