# Log Meal to Health Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an iOS Shortcut, "Log Meal to Health", that takes a Claude-produced nutrition payload (via tappable run link or clipboard), shows an editable confirmation + date picker, and writes every provided nutrient to Apple Health.

**Architecture:** The shortcut is authored as an unsigned `.xml` plist via the `shortcuts-playground:build` agent, validated, signed to `.shortcut`, and committed. Claude does meal-photo analysis and emits a `key: value` text payload; the shortcut only parses, confirms, and logs. Input is modular: passed-in Shortcut Input if present, else Clipboard.

**Tech Stack:** Apple Shortcuts (WFWorkflowActions plist), `shortcuts-playground` plugin (build/sign), HealthKit dietary quantity types, git.

**Spec:** `docs/superpowers/specs/2026-05-29-log-meal-to-health-design.md`

---

## File Structure

- Create: `shortcuts/Log Meal to Health.xml` — canonical unsigned plist (source of truth, committed).
- Create: `signed/Log Meal to Health.shortcut` — signed installable build.
- Modify: `README.md` — add "Log Meal to Health → Apple Health" section with payload schema, run-link recipe, and the Claude instruction snippet.

The shortcut itself is one logical unit (single workflow). Internally it is structured as: input resolution → confirmation → date → parse → per-nutrient logging → summary.

---

## Reference: nutrient table

The build brief and the README both depend on this exact mapping. `key` is the payload key; `Health type` is the HealthKit dietary quantity; `unit` is fixed per type.

**Energy & macros:** `calories`=Dietary Energy(kcal), `carbs`=Carbohydrates(g), `protein`=Protein(g), `total_fat`=Total Fat(g), `saturated_fat`=Saturated Fat(g), `monounsaturated_fat`=Monounsaturated Fat(g), `polyunsaturated_fat`=Polyunsaturated Fat(g), `cholesterol`=Dietary Cholesterol(mg), `sugar`=Dietary Sugar(g), `fiber`=Dietary Fiber(g), `sodium`=Sodium(mg), `water`=Dietary Water(mL), `caffeine`=Caffeine(mg)

**Vitamins:** `vitamin_a`=Vitamin A(mcg), `thiamin`=Thiamin(mg), `riboflavin`=Riboflavin(mg), `niacin`=Niacin(mg), `pantothenic_acid`=Pantothenic Acid(mg), `vitamin_b6`=Vitamin B6(mg), `biotin`=Biotin(mcg), `vitamin_b12`=Vitamin B12(mcg), `vitamin_c`=Vitamin C(mg), `vitamin_d`=Vitamin D(mcg), `vitamin_e`=Vitamin E(mg), `vitamin_k`=Vitamin K(mcg), `folate`=Folate(mcg)

**Minerals:** `calcium`=Calcium(mg), `chloride`=Chloride(mg), `iron`=Iron(mg), `magnesium`=Magnesium(mg), `phosphorus`=Phosphorus(mg), `potassium`=Potassium(mg), `zinc`=Zinc(mg), `copper`=Copper(mg), `manganese`=Manganese(mg), `molybdenum`=Molybdenum(mcg), `selenium`=Selenium(mcg), `chromium`=Chromium(mcg), `iodine`=Iodine(mcg)

Total: 39 nutrients + optional `name`, `date`.

---

## Task 1: Author the shortcut via the build agent

**Files:**
- Create: `shortcuts/Log Meal to Health.xml`

- [ ] **Step 1: Dispatch the `shortcuts-playground:build` agent with the full brief**

Invoke `/shortcuts-playground:build` (the shortcut-builder agent) with this brief verbatim:

> Build an iOS Shortcut named "Log Meal to Health". It logs meal nutrition into Apple Health from a text payload.
>
> **Input (modular):** Accept the shortcut's input as text. If the input has no value, fall back to reading the Clipboard. Call the result `payload`. If `payload` is empty, show an alert: "No meal data found. Run this with a payload from Claude (passed in or copied to the clipboard)." and stop the shortcut.
>
> **Payload format:** newline-separated `key: value` lines. Optional keys `name` (meal label) and `date` (the word `now` or a date string). All other keys are nutrients (see list below).
>
> **Confirmation:** Use "Ask for Input" (type Text, allow multiple lines) with the default value set to `payload`, prompt "Review / edit before logging". Call the edited result `edited`.
>
> **Date:** From `edited`, extract the value of the `date` line if present. Use "Ask for Input" (type Date) with prompt "Meal date & time". Default the date field to the parsed `date` value if it is a real date; otherwise default to the Current Date. (If `date` is `now` or missing/unparseable, default to Current Date.) Call the result `logDate`.
>
> **Parse:** Split `edited` by new lines. For each nutrient key below, get its line, take the text after the first ": ", and trim it. Treat empty or non-numeric values as absent.
>
> **Logging:** For EACH of the 39 nutrient keys below, add a "Log Health Sample" action that logs the corresponding Health type with its fixed unit, using `logDate` as the sample's date. Guard each one inside an "If" so it only runs when that nutrient's parsed value is a non-empty number. Skip absent values.
>
> **Summary:** At the end show a notification titled with the `name` value (or "Meal logged" if no name) summarizing how many nutrients were logged.
>
> Nutrient key → Health type → unit:
> [paste the full Energy & macros / Vitamins / Minerals table from the Reference section above]

Note for the builder: the action identifier is `is.workflow.actions.health.quantity.log` (Log Health Sample); use the Nutrition category quantity sample types. Date parameter on that action sets the sample timestamp.

- [ ] **Step 2: Save the agent's output to the canonical path**

Save the returned unsigned XML to `shortcuts/Log Meal to Health.xml`.

---

## Task 2: Validate the plist

**Files:**
- Verify: `shortcuts/Log Meal to Health.xml`

- [ ] **Step 1: Verify it is a well-formed plist**

Run: `plutil -lint "shortcuts/Log Meal to Health.xml"`
Expected: `OK`

- [ ] **Step 2: Confirm all 39 Log Health Sample actions are present**

Run: `grep -c "health.quantity.log" "shortcuts/Log Meal to Health.xml"`
Expected: `39` (one logging action per nutrient). If lower, the builder collapsed or omitted nutrients — return to Task 1 Step 1 and require all 39.

- [ ] **Step 3: Confirm modular input + clipboard fallback exist**

Run: `grep -E -c "clipboard|Clipboard" "shortcuts/Log Meal to Health.xml"`
Expected: `>= 1` (clipboard fallback action present).

- [ ] **Step 4: Confirm the two Ask-for-Input prompts and the alert exist**

Run: `grep -E -c "ask|alert" "shortcuts/Log Meal to Health.xml"`
Expected: `>= 1` for each pattern; manually confirm a Text input, a Date input, and the empty-payload alert are all present. If any is missing, return to Task 1.

---

## Task 3: Sign / build the installable shortcut

**Files:**
- Create: `signed/Log Meal to Health.shortcut`

- [ ] **Step 1: Sign the validated XML to a `.shortcut`**

Use the `shortcuts-playground` signing path to produce `signed/Log Meal to Health.shortcut` from `shortcuts/Log Meal to Health.xml`. (On macOS this is `shortcuts sign -i <in.xml> -o <out.shortcut> -m anyone`, or the plugin's documented sign helper.)

- [ ] **Step 2: Verify the signed file was produced**

Run: `test -s "signed/Log Meal to Health.shortcut" && echo OK`
Expected: `OK`

---

## Task 4: Document usage in the README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add the usage section**

Append a section to `README.md`:

```markdown
## Log Meal to Health → Apple Health

Logs a meal's nutrition into Apple Health from a payload Claude produces after
analyzing a meal photo.

**How to use with Claude (iOS):**
1. Send Claude a photo of your meal.
2. Ask it to "estimate the nutrition and give me a Log Meal to Health payload".
3. Tap the run link it returns, or copy the payload text and run the shortcut.

**Tell Claude to emit this exact format** (one `key: value` per line, and to
**only include a nutrient when it is genuinely confident — omit, never guess**):

```
name: <meal name>
date: now
calories: <kcal>
protein: <g>
carbs: <g>
total_fat: <g>
... (any of the supported keys below)
```

Run link form:
`shortcuts://run-shortcut?name=Log%20Meal%20to%20Health&input=text=<URL-encoded payload>`

If no input is passed, the shortcut reads the payload from the clipboard.

**Supported keys** (unit in parentheses):
calories(kcal), carbs(g), protein(g), total_fat(g), saturated_fat(g),
monounsaturated_fat(g), polyunsaturated_fat(g), cholesterol(mg), sugar(g),
fiber(g), sodium(mg), water(mL), caffeine(mg), vitamin_a(mcg), thiamin(mg),
riboflavin(mg), niacin(mg), pantothenic_acid(mg), vitamin_b6(mg), biotin(mcg),
vitamin_b12(mcg), vitamin_c(mg), vitamin_d(mcg), vitamin_e(mg), vitamin_k(mcg),
folate(mcg), calcium(mg), chloride(mg), iron(mg), magnesium(mg), phosphorus(mg),
potassium(mg), zinc(mg), copper(mg), manganese(mg), molybdenum(mcg),
selenium(mcg), chromium(mcg), iodine(mcg).
```

- [ ] **Step 2: Verify the README renders the section**

Run: `grep -c "Log Meal to Health" README.md`
Expected: `>= 1`

---

## Task 5: Commit

- [ ] **Step 1: Commit all deliverables**

```bash
git add "shortcuts/Log Meal to Health.xml" "signed/Log Meal to Health.shortcut" README.md
git commit -m "Add Log Meal to Health shortcut"
```

(Per repo policy: no Claude co-author trailer.)

- [ ] **Step 2: Verify the commit**

Run: `git log --oneline -1`
Expected: shows "Add Log Meal to Health shortcut".

---

## Manual device verification (post-merge, user-run)

These cannot be automated; the user runs them on iPhone:

1. Install `signed/Log Meal to Health.shortcut`; grant Health write access on first run.
2. Run with a sample payload (e.g. `calories: 500\nprotein: 30\ncarbs: 40\ntotal_fat: 20`) on the clipboard → confirm the editable box and date picker appear, then check Apple Health → Browse → Nutrition shows the four entries at the chosen time.
3. Tap a `shortcuts://run-shortcut?...` link with a payload → confirm it pre-fills.
4. Run with empty input/clipboard → confirm the "No meal data found" alert.
