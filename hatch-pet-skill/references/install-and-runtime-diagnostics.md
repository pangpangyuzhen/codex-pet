# Install and runtime diagnostics

## Contents

1. Pet package checks
2. First-wake and stale-image checks
3. Create-your-own-pet failure
4. Logging requirements

## Pet package checks

Inspect the installed package under `${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>`:

- parse `pet.json`
- resolve `spritesheetPath` inside the same directory
- require PNG or WebP at exactly 1536×1872
- require alpha support and transparent unused cells
- compare staged and installed SHA-256 hashes
- flag obsolete PNG/WebP siblings as diagnostic noise, even though the manifest path is authoritative

When the displayed pet does not match the contact sheet, trust `pet.json.spritesheetPath`, not the newest-looking sibling filename.

## First-wake and stale-image checks

The app loads custom atlases as data URLs. Large PNGs and same-path replacements can leave the overlay holding stale data.

Repair sequence:

1. Export an alpha WebP.
2. Use a new versioned filename.
3. Update and install `pet.json` with the atlas in one operation.
4. Refresh the pet list.
5. Reselect the custom pet once when the existing overlay object predates the refresh.
6. Verify state rows before changing animation art again.

## Create-your-own-pet failure

The Codex “Create your own pet” button first invokes `install-recommended-skill` for `hatch-pet`; it is not a Supabase pet-session API. Diagnose this installer before investigating Auth, RLS, or application environment variables.

Known failure pattern:

- UI uses `forceReinstall: true`.
- Installation fails because of network timeout, TLS/SSL, bundled-source, permissions, or destination errors.
- The active `${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet` directory may be missing while a complete cached copy remains under `${CODEX_HOME:-$HOME/.codex}/vendor_imports/skills/skills/.curated/hatch-pet`.
- A fixed `catch {}` toast hides the backend message.

Safe recovery:

1. Confirm the cached skill has `SKILL.md`, scripts, references, and agent metadata.
2. Restore it to `${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet` without modifying the signed application bundle.
3. Compare cached and active directories.
4. Reload skills or restart Codex.

Do not patch `Codex.app/Contents/Resources/app.asar` in place; doing so can invalidate the application signature. If source code is available, change the frontend to catch `error`, log the full object, and toast `error.message`. The main-process installer should return `{ success: false, error: message }` and log skill id, source, force-reinstall value, message, and stack.

## Logging requirements

For any failure, preserve:

- operation name and timestamp
- pet id or skill id
- source and destination paths
- manifest path and atlas filename
- HTTP status when a real HTTP request exists
- full error message and stack in local logs
- user-facing toast with a safe concrete message

Do not claim a 401/403/404/500 unless an actual HTTP response produced it. Distinguish network timeout, TLS error, filesystem error, bundled-source error, and authentication error.
