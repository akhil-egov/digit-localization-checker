---
name: digit-localization-checker
description: >
  Validates English, French, and Portuguese UI text strings against the DIGIT Content
  Standards & Guidelines 2.0. Use whenever a user asks to "check localization",
  "validate UI strings", "review content standards", "audit casing", "check DIGIT
  guidelines", or shares a localization file (.xlsx, .csv, .strings, .xml, .json).
  Also triggers when the user pastes text strings and asks for compliance review,
  or mentions "button text", "CTA", "error messages", "heading casing",
  "DIGIT standards", or "localization violations". Always use this skill when
  auditing DIGIT platform UI strings — even if the user just says "check the strings
  in this file".
  Outputs a corrected file + a violations report table with columns:
  string_key, original, corrected, violation_type, language.
---

# DIGIT Localization Checker

## Purpose

Audit UI text strings for compliance with **DIGIT Content Standards & Guidelines 2.0** and produce:
1. A **corrected output** — same format as input, violations fixed in-place
2. A **violations report** — every issue found with: `string_key`, `original`, `corrected`, `violation_type`, `language`

---

## IMPORTANT — Read the standards before doing anything else

**Before evaluating any string, read `references/DIGIT_Content_Standards.md` in full.**

Derive ALL casing rules exclusively from the `## Rule:` sections in that file:
- Each rule defines a `Convention:` (Title Case or Sentence Case), the `Key patterns:` that signal which strings it applies to, and the `Violation type:` to record.
- Do not apply any casing rule not defined in that file.
- If a string's content type has no matching rule, record it as `UNCATEGORIZED` rather than guessing.
- If a rule's `Flag behavior:` is `Flag for review`, do not auto-correct — report the original string unchanged and note it in the violations report.
- If a rule's `Flag behavior:` is `Auto-correct`, apply the convention and record the corrected form.

The standards file is the single source of truth. When it changes, your behaviour changes automatically.

---

## Language Column Detection

DIGIT localization files use varying column name conventions across teams and versions. **Always detect language columns dynamically** by scanning the header row for these keywords (case-insensitive, anywhere in the column name):

| Keyword | Column language |
|---|---|
| `english` | English |
| `french` | French |
| `portuguese` | Portuguese |

Examples of column names this covers: `English Messages`, `DIGIT_LOC_MESSAGE_HEADER_ENGLISH`, `English`, `French Messages`, `DIGIT_LOC_MESSAGE_HEADER_FRENCH`, `Portuguese Messages`, `DIGIT_LOC_MESSAGE_HEADER_PORTUGUESE`.

The `Code` column is always named `Code` (exact match, first column).

If no language column is found at all, tell the user the column names you see and ask them to identify which column contains which language.

---

## Classification Logic

After reading the standards file, classify each string using the `Key patterns:` from the matching `## Rule:` block. Use this priority order when a string could match more than one rule:

1. **Key pattern match** — check the string's `Code` key against the `Key patterns:` in every rule; the first match wins
2. **String length** — 1–4 words with no terminal punctuation suggests a CTA; 8+ words suggests body/error
3. **Verb form** — imperative verb at start (e.g. Submit, Add, Create) suggests CTA; passive or declarative suggests body
4. **Terminal punctuation** — ends with `.` suggests body text; no punctuation suggests CTA or heading

If classification is still ambiguous after all four signals, record the violation type as `CASING_REVIEW_NEEDED`.

---

## Workflow

### Step 1 — Read the standards

Read `references/DIGIT_Content_Standards.md` in full. Build your understanding of every `## Rule:` block before touching any strings.

### Step 2 — Identify input type and detect columns

| Input | Action |
|---|---|
| `.xlsx` file | Read all sheets; detect language columns by scanning headers for `english`, `french`, `portuguese` (case-insensitive). Process each detected language column. |
| `.csv`, `.tsv` | Read rows; detect key + language columns the same way. |
| `.json`, `.strings`, `.xml` | Parse key-value pairs; infer language from filename or ask user. |
| Pasted text | Ask user to clarify string type if not obvious from keys. |

### Step 3 — Classify and check each string

For each string:
- Determine language from column
- Classify the string type using Classification Logic above and the `Key patterns:` from the standards
- Look up the required `Convention:` from the matching rule
- Compare actual casing to required casing

### Step 4 — Record violations

A violation exists when the actual casing differs from the required convention, or when a `Flag for review` rule matches.

Record every violation with:
- `string_key`: the code/key identifier
- `original`: the string as-is
- `corrected`: the fixed version (or the original unchanged if `Flag for review`)
- `violation_type`: the `Violation type:` from the matching rule in the standards file
- `language`: `EN`, `FR`, or `PT`

**Universal violation types** (not language/rule-specific):

| violation_type | Meaning |
|---|---|
| `LEADING_TRAILING_SPACE` | Whitespace before or after the string |
| `CASING_REVIEW_NEEDED` | String type could not be confidently determined |
| `UNCATEGORIZED` | No matching rule found in the standards file |

### Step 5 — Produce outputs

**For .xlsx input (primary use case):**
Use Python + openpyxl to:
- **Sheet 1… N — original sheet names**: Exact copy of input structure, with all detected language columns corrected in-place. Preserve the `Code` column and any other columns (e.g. Module, Locale) unchanged.
- **Final sheet — "Violations Report"**: Table with columns: `string_key` | `original` | `corrected` | `violation_type` | `language` | `sheet`. One row per violation.

**For pasted text / other formats:**
- Produce a markdown violations table
- Then provide the corrected text block

### Step 6 — Summary

After producing outputs, provide a short plain-language summary:
- Total strings checked (by language)
- Total violations found
- Breakdown by `violation_type`
- Any strings flagged for human review (from `Flag for review` rules)

---

## Important Notes

- **Do not change meaning** — only fix casing, never reword.
- **Preserve proper nouns** — names of countries, products, organisations retain their capitalisation.
- **Preserve content inside quotes** — text within quotation marks keeps original casing.
- **Single-word strings** — apply the rule from the standards file; the only universal exception is ALL CAPS single words, which are always a violation regardless of rule.
- **Empty cells** — skip; do not flag as violations.
- **Whitespace-only cells** — skip.

---

## Example

See `assets/example-localizations.xlsx` for a sample input file covering all rule types.

**Example input row:**
```
Code: ADD_MEMBER
English: Add member
French: Ajouter Un Membre
Portuguese: Adicionar Membro
```

**Expected output (corrected):**
```
English: Add Member          ← CTA_NOT_TITLE_CASE auto-corrected
French: Ajouter Un Membre    ← unchanged; FRENCH_TITLE_CASE_REVIEW flagged for human review
Portuguese: Adicionar Membro ← unchanged; PORTUGUESE_TITLE_CASE_REVIEW flagged for human review
```

---

## Error Handling

- If no `Code` column is found, use row number as `string_key`.
- If a sheet is empty or has fewer than 2 rows, skip it and note it in the summary.
- If column detection is ambiguous, list the headers you see and ask the user to confirm the mapping before proceeding.
- If `references/DIGIT_Content_Standards.md` cannot be read, stop and tell the user — do not fall back to built-in rules.
