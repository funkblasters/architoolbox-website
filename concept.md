# ArchiToolbox — Concept Notes

## Vision

A JetBrains-style toolbox for architecture professionals. Users can download and install
specialized tools (DXF Cleaner, future apps) from a single launcher. The toolbox is
optional — each app works as a standalone install — but provides a unified experience
for users who want it.

**DXF Cleaner** is the first app in the ecosystem.

## Core flows

1. User downloads the toolbox
2. Toolbox fetches a remote catalog of available apps (hosted API)
3. User installs apps directly from the toolbox (no separate installer needed)
4. Toolbox tracks installed versions and handles updates
5. Toolbox launches apps from a single place

Apps can still be distributed as standalone installers (AppImage/DMG/NSIS) for users
who prefer not to use the toolbox.

## Install strategy

Prefer the **managed directory** approach (JetBrains-style): the toolbox extracts each
app into a directory it controls, rather than delegating to a per-app installer. This
gives:

- No UAC/sudo prompts
- Multiple versions side by side
- Clean uninstall

## What NOT to do yet

Don't let this influence individual app architecture prematurely. The only
future-proofing needed in standalone apps (like DXF Cleaner) is a clean versioning
and update mechanism — which is good practice anyway.

## Discovery of standalone installs

Apps installed outside the toolbox won't be auto-discovered in the JetBrains model.
If that becomes desirable later, a drop-file convention (e.g. each app writes a small
JSON to `~/.architoolbox/apps/`) could enable it without tight coupling at build time.
