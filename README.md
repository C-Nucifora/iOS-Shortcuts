# iOS Shortcuts

Version-controlled backup of my Shortcuts, edited via the `shortcuts-playground` Claude Code plugin.

## Layout
- `shortcuts/` — canonical unsigned `.xml` plists (source of truth, diff-friendly)
- `signed/` — built `.shortcut` files for one-tap install (rebuild on demand)

## Workflow
- **Create**: `/shortcuts-playground:build <brief>` → save XML to `shortcuts/<name>.xml` → commit
- **Edit**: `/shortcuts-playground:remix shortcuts/<name>.xml <change>` → commit diff
- **Install on device**: build to `signed/<name>.shortcut`, then AirDrop or open the GitHub raw URL on iOS

## Importing from the Shortcuts app
### macOS (easiest)
1. Open Shortcuts.app
2. Right-click the shortcut → **Copy** (or File → Export → unsigned)
3. Save the `.shortcut` file somewhere
4. Hand the path to Claude; the playground skill converts it to XML in `shortcuts/`

### iOS
1. Open the shortcut → share sheet → **Share Shortcut** → *File*
2. AirDrop or save to iCloud Drive → pull onto your Mac → same as above

> Signed `.shortcut` files are AEA1-encrypted. The playground skill unwraps them to plist XML for git.
