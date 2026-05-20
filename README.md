# digit-localization-checker

A Claude skill that audits UI text strings for compliance with the **DIGIT Content Standards & Guidelines 2.0**.

It checks English, French, and Portuguese localization files and produces:
- A **corrected output file** — violations fixed in-place
- A **violations report** — every issue with `string_key`, `original`, `corrected`, `violation_type`, `language`

## What it checks

Rules are defined in [`references/DIGIT_Content_Standards.md`](references/DIGIT_Content_Standards.md). The current ruleset:

| Content type | Required casing | Applies to |
|---|---|---|
| Buttons / CTAs | Title Case | Keys with `ACTION_`, `BTN_`, `SAVE`, `CANCEL`, etc. |
| Body text, descriptions, help text | Sentence case | Keys with `_description`, `_hint`, `_helpText`, `_tooltip`, etc. |
| Error messages and validations | Sentence case | Keys with `_error`, `_message`, `ERROR`, `FAILED`, etc. |
| Headings, labels, captions | Sentence case | Keys with `HEADING`, `HEADER`, `TITLE`, `_label_`, `CAPTION`, etc. |
| All French strings | Sentence case | Title Case flagged for human review |
| All Portuguese strings | Sentence case | Title Case flagged for human review |

Column names are detected automatically — works with `English Messages`, `DIGIT_LOC_MESSAGE_HEADER_ENGLISH`, or any column whose name contains "english", "french", or "portuguese".

## Repo structure

```
digit-localization-checker/
├── SKILL.md                              # Skill logic — workflow, classification, output format
├── references/
│   └── DIGIT_Content_Standards.md       # ← All casing rules live here
└── assets/
    └── example-localizations.xlsx       # Sample input file for testing
```

**`SKILL.md` contains no hardcoded rules.** It reads `references/DIGIT_Content_Standards.md` at runtime and derives all conventions from the `## Rule:` sections there. When the standard changes, only the standards file needs updating.

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

## Updating the rules

**All casing rules live in [`references/DIGIT_Content_Standards.md`](references/DIGIT_Content_Standards.md) — not in `SKILL.md`.**

When the DIGIT Content Standards change:

1. Open `references/DIGIT_Content_Standards.md`
2. Find the relevant `## Rule:` block and update the `Convention:`, `Key patterns:`, or `Flag behavior:` line
3. To add a new rule, copy any existing block and fill in all fields
4. Open a PR — no changes to `SKILL.md` are needed

Each rule block looks like this:

```markdown
## Rule: CTA and button text
- Convention: Title Case
- Applies to: Interactive controls the user taps or clicks to take an action
- Key patterns: `ACTION_`, `BTN_`, `SUBMIT`, `SAVE`, `CANCEL`, ...
- Exceptions: Articles and conjunctions are lowercase unless first word
- Languages: English
- Flag behavior: Auto-correct
- Violation type: `CTA_NOT_TITLE_CASE`
```

To change a convention (e.g. headings switch from Sentence Case to Title Case), edit the `Convention:` line in the relevant block. The skill picks it up immediately on the next run.

### Standards doc

The underlying standard is the **DIGIT Content Standards & Guidelines 2.0** maintained by the eGovernments Foundation design team. The `references/DIGIT_Content_Standards.md` file in this repo should stay in sync with that document.
