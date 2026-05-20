# digit-localization-checker

A Claude skill that audits UI text strings for compliance with the **DIGIT Content Standards & Guidelines 2.0**.

It checks English, French, and Portuguese localization files and produces:
- A **corrected output file** — violations fixed in-place
- A **violations report** — every issue with `string_key`, `original`, `corrected`, `violation_type`, `language`

## What it checks

| Rule | String type | Required casing |
|---|---|---|
| Rule 1 | Buttons / CTAs (`ACTION_`, `BTN_`, `SAVE`, `CANCEL`, etc.) | Title Case |
| Rule 2 | Body text, error messages, descriptions, help text | Sentence case |
| Rule 3 | Headings, labels, captions | Sentence case |
| FR rule | All French strings | Sentence case — Title Case flagged for review |
| PT rule | All Portuguese strings | Sentence case — Title Case flagged for review |

Column names are detected automatically — works with `English Messages`, `DIGIT_LOC_MESSAGE_HEADER_ENGLISH`, or any column whose name contains "english", "french", or "portuguese".

## Installation

1. Download [`digit-localization-checker.zip`](../../archive/refs/heads/main.zip) (or clone this repo)
2. In Claude.ai → **Settings → Skills → Upload skill**
3. Upload the zip, or point it at this folder

To install via Claude Code CLI:
```
/plugin install https://github.com/akhil-egov/digit-localization-checker
```

## Usage

Just hand Claude a localization file or paste strings:

> "Check this file for DIGIT content standard violations: Localizations.xlsx"

> "Audit these UI strings for DIGIT compliance: `ADD_MEMBER | Add member`"

Claude will produce a corrected `.xlsx` and a violations report.

## Contributing rule changes

The casing rules live in `SKILL.md` under the **DIGIT Casing Rules** sections. To change a rule:

1. Edit `SKILL.md` directly
2. Update the relevant section (English Rule 1/2/3, French rule, or Portuguese rule)
3. If adding a new violation type, add it to the **Violation Types** table
4. Open a PR with a short description of what changed and why

### Key patterns to know

String type is inferred from the `Code` key first, then string length and verb form:

- `ACTION_`, `BTN_`, `BUTTON_`, `SUBMIT`, `SAVE`, `CANCEL`, `CREATE`, `ADD_`, `DELETE_`, `EDIT_` → CTA → Title Case
- `_error`, `_message`, `_description`, `_hint`, `_helpText`, `_tooltip`, `_mandatory`, `DESCRIPTION`, `ERROR`, `FAILED` → Body → Sentence case
- `TITLE`, `HEADER`, `HEADING`, `_label_`, `CAPTION` → Heading → Sentence case

To add a new key pattern, find the `Classification Logic` section in `SKILL.md` and add the pattern to the appropriate rule.

### Standards doc

The underlying standard is the **DIGIT Content Standards & Guidelines 2.0** maintained by the eGovernments Foundation design team. If the standard changes, update `SKILL.md` to match.
