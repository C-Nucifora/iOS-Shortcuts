# Log Meal to Health — Design

**Date:** 2026-05-29
**Status:** Approved (pending spec review)

## Goal

An iOS Shortcut that logs a meal's nutrition into Apple Health. Claude (in the
iOS app) analyzes a meal photo, estimates nutrition, and hands the shortcut a
plain-text payload. The shortcut shows an editable confirmation, lets the user
pick/adjust the date, then writes each provided nutrient to Apple Health.

The same shortcut works whether the payload arrives via a tappable run link or
the clipboard — input handling is modular.

## Trigger / handoff

Claude cannot directly press "run" on a shortcut from the iOS app, so it hands
the user one of two things:

- **Tappable run link (primary):**
  `shortcuts://run-shortcut?name=Log%20Meal%20to%20Health&input=text=<URL-encoded payload>`
  The user taps it; the shortcut opens with the payload pre-filled.
- **Copy/paste (fallback):** Claude outputs the payload as text; the user copies
  it and runs the shortcut, which reads the clipboard.

The shortcut supports both with no mode switch: it uses the passed-in Shortcut
Input if present, otherwise falls back to Clipboard.

## Payload format

Newline-separated `key: value` lines. Human-readable, URL-encodable, and
directly editable in a text box — one format covers link, clipboard, and the
confirmation surface.

```
name: Grilled chicken salad
date: now
calories: 420
protein: 38
carbs: 18
total_fat: 22
...
```

- `name` (optional) — meal label, used only in the summary.
- `date` (optional) — `now` or a parseable date/time; controls the Health
  sample timestamp. Absent ⇒ now.
- Any nutrient line Claude omits or leaves blank is skipped.
- Claude only emits lines it can estimate.

### Supported nutrients (full HealthKit dietary set)

Each maps to a HealthKit dietary quantity type with a fixed unit.

**Energy & macros**
| key | Health type | unit |
|-----|-------------|------|
| `calories` | Dietary Energy | kcal |
| `carbs` | Carbohydrates | g |
| `protein` | Protein | g |
| `total_fat` | Total Fat | g |
| `saturated_fat` | Saturated Fat | g |
| `monounsaturated_fat` | Monounsaturated Fat | g |
| `polyunsaturated_fat` | Polyunsaturated Fat | g |
| `cholesterol` | Dietary Cholesterol | mg |
| `sugar` | Dietary Sugar | g |
| `fiber` | Dietary Fiber | g |
| `sodium` | Sodium | mg |
| `water` | Dietary Water | mL |
| `caffeine` | Caffeine | mg |

**Vitamins**
| key | Health type | unit |
|-----|-------------|------|
| `vitamin_a` | Vitamin A | mcg |
| `thiamin` | Thiamin (B1) | mg |
| `riboflavin` | Riboflavin (B2) | mg |
| `niacin` | Niacin (B3) | mg |
| `pantothenic_acid` | Pantothenic Acid (B5) | mg |
| `vitamin_b6` | Vitamin B6 | mg |
| `biotin` | Biotin (B7) | mcg |
| `vitamin_b12` | Vitamin B12 | mcg |
| `vitamin_c` | Vitamin C | mg |
| `vitamin_d` | Vitamin D | mcg |
| `vitamin_e` | Vitamin E | mg |
| `vitamin_k` | Vitamin K | mcg |
| `folate` | Folate | mcg |

**Minerals**
| key | Health type | unit |
|-----|-------------|------|
| `calcium` | Calcium | mg |
| `chloride` | Chloride | mg |
| `iron` | Iron | mg |
| `magnesium` | Magnesium | mg |
| `phosphorus` | Phosphorus | mg |
| `potassium` | Potassium | mg |
| `zinc` | Zinc | mg |
| `copper` | Copper | mg |
| `manganese` | Manganese | mg |
| `molybdenum` | Molybdenum | mcg |
| `selenium` | Selenium | mcg |
| `chromium` | Chromium | mcg |
| `iodine` | Iodine | mcg |

## Flow inside the shortcut

1. **Get input (modular).** If Shortcut Input has a value, use it; else read
   Clipboard. Result is the raw payload text.
2. **Editable confirmation.** Pre-fill a multiline text input with the payload
   so the user can correct any number before logging. (Single text box is the
   only practical way to edit ~39 values without one prompt per nutrient.)
3. **Date.** Parse the `date` line; feed it as the default of a native date
   picker (Ask for Input, type Date), defaulting to now when absent or `now`.
   The user can backdate here.
4. **Parse.** Split the edited text into lines, then into key/value pairs.
5. **Log to Health.** One guarded *Log Health Sample* action per supported
   nutrient: if the parsed value is non-empty and numeric, log it with the
   correct type, unit, and the chosen date; otherwise skip.
6. **Summary.** A notification listing the meal `name` (if given) and which
   nutrients were logged.

## Error handling

- No usable input (empty Shortcut Input and empty/non-matching clipboard):
  show an alert explaining the expected payload format and stop.
- Blank, zero, or non-numeric nutrient value: silently skipped (not an error).
- Unparseable `date`: fall back to now.

## Deliverables

- `shortcuts/Log Meal to Health.xml` — canonical unsigned plist (committed).
- `signed/Log Meal to Health.shortcut` — installable build.
- README section "Log Meal to Health → Apple Health": the payload schema, the
  `shortcuts://run-shortcut` link recipe, and a copy/paste-ready instruction the
  user can give Claude so it emits the exact format.

## Out of scope (YAGNI)

- No nutrition API or vision model inside the shortcut — Claude does analysis.
- No persistent meal history beyond what Apple Health stores.
- No per-nutrient individual edit prompts.
