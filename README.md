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

## Log Meal to Health → Apple Health

Logs a meal's nutrition into Apple Health from a payload Claude produces after
analyzing a meal photo. Files: `shortcuts/Log Meal to Health.xml` (source),
`signed/Log Meal to Health.shortcut` (installable).

**How to use with Claude (iOS):**
1. Send Claude a photo of your meal.
2. Ask it to "estimate the nutrition and give me a Log Meal to Health payload".
3. Tap the run link it returns, or copy the payload text and run the shortcut.

The shortcut shows the values in an editable box and a date picker (defaulting
to now) before writing anything, so you can correct an estimate or backdate a
meal. Any nutrient left blank is skipped. First run prompts for Health access.

**Tell Claude to emit this exact format** — one `key: value` per line, and to
**only include a nutrient when it is genuinely confident; omit, never guess**
(a missing value is skipped cleanly, a fabricated one pollutes Health data):

```
name: <meal name>
date: now
calories: <kcal>
protein: <g>
carbs: <g>
total_fat: <g>
... (any of the supported keys below)
```

`name` and `date` are optional (`date` accepts `now` or a date string).

**Run-link form** (one tap, payload pre-filled):

```
shortcuts://run-shortcut?name=Log%20Meal%20to%20Health&input=text=<URL-encoded payload>
```

If no input is passed, the shortcut reads the payload from the clipboard
instead — so the copy/paste flow works with the same shortcut.

**Supported keys** (unit in parentheses):
`calories`(kcal), `carbs`(g), `protein`(g), `total_fat`(g), `saturated_fat`(g),
`monounsaturated_fat`(g), `polyunsaturated_fat`(g), `cholesterol`(mg),
`sugar`(g), `fiber`(g), `sodium`(mg), `water`(mL), `caffeine`(mg),
`vitamin_a`(mcg), `thiamin`(mg), `riboflavin`(mg), `niacin`(mg),
`pantothenic_acid`(mg), `vitamin_b6`(mg), `biotin`(mcg), `vitamin_b12`(mcg),
`vitamin_c`(mg), `vitamin_d`(mcg), `vitamin_e`(mg), `vitamin_k`(mcg),
`folate`(mcg), `calcium`(mg), `chloride`(mg), `iron`(mg), `magnesium`(mg),
`phosphorus`(mg), `potassium`(mg), `zinc`(mg), `copper`(mg), `manganese`(mg),
`molybdenum`(mcg), `selenium`(mcg), `chromium`(mcg), `iodine`(mcg).
