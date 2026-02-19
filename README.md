

# Third‑Party Notices & Credits: License Compliance for Unity

Local-first Unity Editor tooling to scan a Unity project for third-party components and generate deterministic, release-ready attribution artifacts (third-party notices, credits, and a compliance manifest).

This repository contains documentation and examples only. It does not include the commercial source code.

![Overview Image](Screenshots/Overview.jpg)

## Get the tool

- itch.io (primary): https://kevindevelopment.itch.io/compliance
- Product page: https://smartindie.dev/assets/game-compliance-pack/
- Unity Asset Store (available soon): https://assetstore.unity.com/packages/slug/358432

## Why this exists (Unity license compliance at release time)

Unity projects routinely accumulate third-party software from multiple sources:

- Unity Package Manager (UPM) dependencies, including transitive dependencies
- Asset Store plugins and embedded third-party code
- DLLs and native plugins (.dll, .so, .dylib, .bundle)
- Copied utility folders and middleware

At ship time you need to identify what is third-party, find the actual license terms, assemble correct notices and attributions, and keep the result stable across updates and builds. Without a system, this becomes manual, error-prone, and often happens too late.

![Who it's for](Screenshots/Who%20its%20for%20and%20not.jpg)

## What the tool generates (release-ready outputs)

The tool exports a compliance pack with these core files:

- `THIRD_PARTY_NOTICES.txt`  
  Full attributions plus license text when available. Always includes every detected component, even if unresolved.

- `CREDITS.md`  
  Short, human-readable credits suitable for an in-game credits section or store page (no full license text dumps).

- `compliance-manifest.json`  
  Machine-readable source of truth for diffs, audits, and repeatable builds.

Notes:
- Outputs are stable-sorted by a deterministic component key so diffs stay clean.
- If a component remains unresolved, it is clearly marked `UNKNOWN`. The tool does not guess licenses.

This tool is not legal advice and does not guarantee compliance. It is a workflow tool that reduces release risk by improving visibility and reviewability.

## Key principles

- Local-first: scans your project on disk inside Unity. No uploading project data by default.
- Deterministic: stable component keys and stable ordering so diffs are meaningful.
- No guessing: if the license cannot be verified, it stays `UNKNOWN` until you resolve it.
- Reviewable: outputs are plain text and JSON, easy to audit and commit.

## How it works (pipeline)

1. Scan project sources for third-party components
2. Normalize results into stable component entries
3. Resolve licenses conservatively (file evidence or explicit user choice)
4. Apply manual overrides saved in a project file (committable so teams share the same decisions)
5. Generate `THIRD_PARTY_NOTICES.txt`, `CREDITS.md`, and `compliance-manifest.json`
6. Compare against the previous manifest to detect added, removed, and changed components

## What it scans

### Unity Package Manager (UPM)

- Reads `Packages/manifest.json` for direct dependencies
- Reads `Packages/packages-lock.json` for resolved dependencies (direct and transitive)
- Captures package name, version, and source (registry, git, local path)
- Extracts license hints only from verifiable metadata or included files

### File system plugins and embedded components

Scans common locations (configurable), typically:

- `Assets/Plugins`
- `Assets/ThirdParty`

Detects common binary formats:

- `.dll`, `.so`, `.dylib`, `.bundle`

### License and notice file discovery

For each detected component, searches nearby paths for:

- `LICENSE`, `LICENSE.txt`, `LICENSE.md`
- `NOTICE`, `NOTICE.txt`
- `COPYING`, `COPYING.txt`
- `Third Party Notices`, `ThirdPartyNotices`

If found, the license or notice text is treated as strong evidence and can be used as the source of truth.

## License resolution rules (conservative by design)

Resolution priority:

1. Manual override  
   If you have set licenseId, attribution, URL, or license text for a component key, that wins.

2. Detected license or notice file  
   If a `LICENSE` or `NOTICE` file is found adjacent to the component, its contents are used as the license text source of truth.

3. Verifiable metadata (SPDX where possible)  
   If unambiguous metadata identifies the license, the tool can map to an SPDX license identifier and use canonical text packaged with the tool.

4. UNKNOWN  
   If nothing is reliable, the component stays `UNKNOWN` until you resolve it.

The tool must not fabricate license IDs or texts.

## Typical workflow

1. Open the tool in Unity.
2. Click Scan.
3. Resolve anything marked `UNKNOWN` by adding verifiable license info and the required attribution line.
4. Export the compliance pack.
5. Commit the outputs (and overrides) so the team shares the same compliance state.

## Diffs and audits (why the manifest exists)

`compliance-manifest.json` is the machine-readable record that enables:

- Diffs between scans to show added, removed, and changed components
- Auditable history in source control
- Repeatable generation of notices and credits

## Build integration (optional)

A pre-build hook can re-run Scan and Generate before building.

Recommended settings:

- Warn if `UNKNOWN` exists
- Optionally fail the build if `UNKNOWN` exists

This turns compliance from a release-time scramble into a routine check.

## Determinism and clean diffs

Determinism is a feature:

- Stable component keys so a dependency is consistently identified across machines and scans
- Stable ordering in all outputs
- Sorted filesystem enumeration to avoid nondeterministic ordering
- Avoid timestamps inside notice files unless explicitly enabled

## What this is not

- Not legal advice
- Not a compliance guarantee
- Not a license guessing tool
- Not a cloud service
- Not perfect detection of every possible third-party item in every Unity project

It aims to reliably catch the common sources, surface unknowns early, and make resolution fast and repeatable.

## FAQ

### Does this guarantee compliance?
No. It generates reviewable artifacts and forces `UNKNOWN` visibility so you can make informed decisions.

### Why not just do this manually once?
Projects change. Deterministic scans plus diffs prevent regressions and last-minute surprises.

### Why keep UNKNOWN instead of best-guessing?
Guessing licenses is how teams ship incorrect notices. `UNKNOWN` is a deliberate stop-and-verify state.

## Get the tool

- itch.io (primary): https://kevindevelopment.itch.io/compliance
- Product page: https://smartindie.dev/assets/game-compliance-pack/
- Unity Asset Store (available soon): https://assetstore.unity.com/packages/slug/358432
