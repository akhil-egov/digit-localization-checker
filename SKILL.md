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

## DIGIT Casing Rules — English

### Rule 1 — Button Text & CTAs → Title Case
Capitalize first letter of every word **except** articles (a, an, the) and conjunctions (and, but, or, nor, for, so, yet) unless they are the first word.

**Signals a string is a CTA/button:** short (1–5 words), imperative verb, or code key contains `ACTION_`, `BTN_`, `BUTTON_`, `SUBMIT`, `CANCEL`, `SAVE`, `CREATE`, `ADD_`, `DELETE_`, `EDIT_`, `CONFIRM`, or similar action patterns.

✅ `Submit Feedback` | `Create Account` | `Add Member` | `Save Beneficiary`
❌ `submit feedback` | `Submit feedback` | `add member`

### Rule 2 — Body Text, Descriptions, Help Text, Error Messages, Validations → Sentence Case
Capitalize only the **first letter** of the string. Proper nouns and text inside quotes retain their capitalisation.

**Signals:** longer phrasing, contains "please", "must", "cannot", "failed", "successfully", or code key contains `_error`, `_message`, `_description`, `_hint`, `_alert`, `_info`, `_helpText`, `_tooltip`, `_mandatory`, `DESCRIPTION`, `ERROR`, `SUCCESS`, `FAILED`.

✅ `Please enter a valid email address`
✅ `Administration failed`
❌ `Please Enter A Valid Email Address`
❌ `administration failed` (missing first-letter cap)

### Rule 3 — Headings, Labels, Captions → Sentence Case
First letter capitalised, rest lowercase unless proper noun.

**Signals:** code key contains `TITLE`, `HEADER`, `HEADING`, `_label_`, `_LABEL`, `CAPTION`.

✅ `Beneficiary details` | `Date of birth`
❌ `Beneficiary Details` | `Date Of Birth`

---

## DIGIT Casing Rules — French

French does **not** use Title Case. All string types in French use Sentence Case (capitalize first word only). Proper nouns retain capitalisation.

**Flag for review** any French string where multiple words are capitalised — this likely mirrors English incorrectly.

✅ `Soumettre` | `Ajouter un membre` | `Échec de l'administration`
⚠️ `Ajouter Un Membre` → flag as `FRENCH_TITLE_CASE_REVIEW`

---

## DIGIT Casing Rules — Portuguese

Same as French: Sentence Case for all string types. Flag any string where multiple words are capitalised.

✅ `Salvar` | `Adicionar membro` | `Falha na administração`
⚠️ `Salvar Beneficiário` → flag as `PORTUGUESE_TITLE_CASE_REVIEW`

---

## Classification Logic

When a string_key is not obvious, infer type from these signals (in priority order):
1. **Key pattern** — strongest signal (see rules above)
2. **String length** — 1–4 words strongly suggests CTA; 8+ words strongly suggests body/error
3. **Verb form** — imperative verb at start (Submit, Add, Create) → CTA; descriptive/passive → body
4. **Punctuation** — ends with `.` → body text; no punctuation → CTA or heading

If type cannot be determined confidently, apply Sentence Case and note `violation_type` as `CASING_REVIEW_NEEDED`.

---

## Workflow

### Step 1 — Identify Input Type and Detect Columns

| Input | Action |
|---|---|
| `.xlsx` file | Read all sheets; detect language columns by scanning headers for `english`, `french`, `portuguese` (case-insensitive). Process each detected language column. |
| `.csv`, `.tsv` | Read rows; detect key + language columns the same way. |
| `.json`, `.strings`, `.xml` | Parse key-value pairs; infer language from filename or ask user. |
| Pasted text | Ask user to clarify string type (CTA / body / heading) if not obvious from keys. |

### Step 2 — Classify Each String

For each string:
- Determine language from column
- Determine string type (CTA, body/error, heading) using Classification Logic above
- Apply the corresponding casing rule

### Step 3 — Detect Violations

A violation exists when the actual casing differs from the required casing.

Record every violation with:
- `string_key`: the code/key identifier
- `original`: the string as-is
- `corrected`: the fixed version
- `violation_type`: one of the values below
- `language`: `EN`, `FR`, or `PT`

**Violation Types:**

| violation_type | Meaning |
|---|---|
| `CTA_NOT_TITLE_CASE` | Button/CTA text should be Title Case |
| `BODY_NOT_SENTENCE_CASE` | Body/error/description should be Sentence Case |
| `HEADING_NOT_SENTENCE_CASE` | Heading/label should be Sentence Case |
| `FRENCH_TITLE_CASE_REVIEW` | French string uses Title Case — flag for human review |
| `PORTUGUESE_TITLE_CASE_REVIEW` | Portuguese string uses Title Case — flag for human review |
| `LEADING_TRAILING_SPACE` | Whitespace before or after the string |
| `CASING_REVIEW_NEEDED` | Type could not be confidently determined |

### Step 4 — Produce Outputs

**For .xlsx input (primary use case):**
Use Python + openpyxl to:
- **Sheet 1… N — original sheet names**: Exact copy of input structure, with all detected language columns corrected in-place. Preserve the `Code` column and any other columns (e.g. Module, Locale) unchanged.
- **Final sheet — "Violations Report"**: Table with columns: `string_key` | `original` | `corrected` | `violation_type` | `language` | `sheet`. One row per violation.

**For pasted text / other formats:**
- Produce a markdown violations table
- Then provide the corrected text block

### Step 5 — Summary

After producing outputs, provide a short plain-language summary:
- Total strings checked (by language)
- Total violations found
- Breakdown by `violation_type`
- Any French or Portuguese strings flagged for human review

---

## Important Notes

- **Do not change meaning** — only fix casing, never reword.
- **Preserve proper nouns** — names of countries, products, organisations retain their capitalisation.
- **Preserve content inside quotes** — text within quotation marks keeps original casing.
- **Single-word strings** — compliant in both Title Case and Sentence Case; flag only if ALL CAPS.
- **French/Portuguese review flag** — corrections in these languages should be flagged with a note that a native speaker should confirm, since grammar rules (e.g. noun capitalisation) may require exceptions.
- **Empty cells** — skip; do not flag as violations.
- **Whitespace-only cells** — skip.

---

## Example

**Input row (from any file format):**
```
Code: ADD_MEMBER
English: Add member
French: Ajouter Un Membre
Portuguese: Adicionar Membro
```

**Output (corrected):**
```
Code: ADD_MEMBER
English: Add Member
French: Ajouter Un Membre  ← unchanged in corrected sheet, flagged for review
Portuguese: Adicionar Membro  ← unchanged in corrected sheet, flagged for review
```

**Violations report rows:**
```
ADD_MEMBER | Add member | Add Member | CTA_NOT_TITLE_CASE | EN
ADD_MEMBER | Ajouter Un Membre | Ajouter un membre | FRENCH_TITLE_CASE_REVIEW | FR
ADD_MEMBER | Adicionar Membro | Adicionar membro | PORTUGUESE_TITLE_CASE_REVIEW | PT
```

---

## Error Handling

- If no `Code` column is found, use row number as `string_key`.
- If a sheet is empty or has fewer than 2 rows, skip it and note it in the summary.
- If column detection is ambiguous, list the headers you see and ask the user to confirm the mapping before proceeding.
