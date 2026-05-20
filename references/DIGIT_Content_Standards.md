# DIGIT Content Standards & Guidelines 2.0 — Casing Rules

This file is the **single source of truth** for all casing rules enforced by the
digit-localization-checker skill. Edit this file when the DIGIT Content Standards change.
The skill reads and applies these rules at runtime — no changes to SKILL.md are needed.

---

## How to edit this file

Each rule block has the form:

```
## Rule: [content type name]
- Convention: [Title Case | Sentence Case]
- Applies to: [plain-English description of which strings match this rule]
- Key patterns: [comma-separated code key substrings that signal this type]
- Exceptions: [any exceptions, or "None"]
- Languages: [English | French | Portuguese | All]
- Flag behavior: [Auto-correct | Flag for review]
- Violation type: [SNAKE_CASE identifier used in the violations report]
```

To add a new rule, copy a block and fill in all fields. To change a convention,
edit the `Convention:` line. The skill will apply the updated rule on the next run.

---

## Rule: CTA and button text

- Convention: Title Case
- Applies to: Interactive controls the user taps or clicks to take an action — buttons, submit controls, links used as actions, confirmation prompts
- Key patterns: `ACTION_`, `BTN_`, `BUTTON_`, `SUBMIT`, `CANCEL`, `SAVE`, `CREATE`, `ADD_`, `DELETE_`, `EDIT_`, `CONFIRM`, `_ACTION`, `_BTN`, `ACTION_BUTTON`
- Exceptions: Articles (a, an, the) and coordinating conjunctions (and, but, or, nor, for, so, yet) are lowercase unless they are the first word
- Languages: English
- Flag behavior: Auto-correct
- Violation type: `CTA_NOT_TITLE_CASE`

---

## Rule: Body text, descriptions, and help text

- Convention: Sentence Case
- Applies to: Explanatory copy, instructional text, placeholder text, tooltip content, help text shown beneath fields, onboarding descriptions
- Key patterns: `_description`, `_DESCRIPTION`, `DESCRIPTION`, `_hint`, `_helpText`, `_HELPTEXT`, `_tooltip`, `_TOOLTIP`, `_info`, `_INFO`, `_placeholder`, `SCREEN_DESCRIPTION`
- Exceptions: Proper nouns retain their capitalisation. Text inside quotation marks is preserved as-is.
- Languages: English
- Flag behavior: Auto-correct
- Violation type: `BODY_NOT_SENTENCE_CASE`

---

## Rule: Error messages and validation text

- Convention: Sentence Case
- Applies to: Inline validation errors, form-level errors, system alerts, success/failure notifications, mandatory-field messages
- Key patterns: `_error`, `_ERROR`, `ERROR`, `_message`, `_MESSAGE`, `_alert`, `_ALERT`, `_mandatory`, `_MANDATORY`, `SUCCESS`, `FAILED`, `_validation`
- Exceptions: Proper nouns retain their capitalisation.
- Languages: English
- Flag behavior: Auto-correct
- Violation type: `BODY_NOT_SENTENCE_CASE`

---

## Rule: Headings, labels, and captions

- Convention: Sentence Case
- Applies to: Screen titles, section headings, sub-headings, field labels, column headers, image captions, tab names
- Key patterns: `TITLE`, `HEADER`, `HEADING`, `_label_`, `_LABEL`, `_LABEL_`, `LABEL_`, `CAPTION`, `SCREEN_HEADING`, `SCREEN_TITLE`
- Exceptions: Named sections used as proper titles (e.g. a section always referred to as "Latest Updates") may retain Title Case — mark as `CASING_REVIEW_NEEDED` rather than auto-correcting.
- Languages: English
- Flag behavior: Auto-correct
- Violation type: `HEADING_NOT_SENTENCE_CASE`

---

## Rule: French — all string types

- Convention: Sentence Case
- Applies to: All French strings regardless of type (CTAs, headings, body, errors). French grammar does not use Title Case.
- Key patterns: (detected via language column, not key patterns)
- Exceptions: Proper nouns retain their capitalisation. Acronyms retain ALL CAPS. A native French speaker should confirm corrections before release.
- Languages: French
- Flag behavior: Flag for review
- Violation type: `FRENCH_TITLE_CASE_REVIEW`

---

## Rule: Portuguese — all string types

- Convention: Sentence Case
- Applies to: All Portuguese strings regardless of type. Portuguese grammar does not use Title Case.
- Key patterns: (detected via language column, not key patterns)
- Exceptions: Proper nouns retain their capitalisation. Acronyms retain ALL CAPS. A native Portuguese speaker should confirm corrections before release.
- Languages: Portuguese
- Flag behavior: Flag for review
- Violation type: `PORTUGUESE_TITLE_CASE_REVIEW`
