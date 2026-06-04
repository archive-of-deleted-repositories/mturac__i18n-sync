# /i18n-sync

**Full scan of all locales.**

Runs `i18n-scan` across every detected locale in the project and prints a consolidated report.

## Usage

```
/i18n-sync
```

No arguments required. Detects locale files automatically.

## What it does

1. Auto-detects translation file format (JSON nested/flat, YAML, PO, TypeScript const).
2. Finds all locale directories and files (`locales/`, `i18n/`, `translations/`, etc.).
3. Identifies the base locale (default: `en`).
4. Parses every locale file, extracts dot-notation keys.
5. For each non-base locale, computes:
   - **Missing** keys (in base, not in locale)
   - **Extra** keys (in locale, not in base — orphaned)
   - **Duplicate values** (multiple keys with the same translated string)
   - **Untranslated** keys (locale value identical to base value)
6. Prints a summary table and detailed lists.

## Example output

```
i18n-sync Scan Report
Base locale: en (142 keys)
Scanned: fr, de, es, zh

┌─────────┬─────────┬───────┬────────────┬──────────────┐
│ Locale  │ Missing │ Extra │ Duplicates │ Untranslated │
├─────────┼─────────┼───────┼────────────┼──────────────┤
│ fr      │ 3       │ 1     │ 2          │ 0            │
│ de      │ 12      │ 0     │ 0          │ 4            │
│ es      │ 0       │ 0     │ 1          │ 1            │
│ zh      │ 28      │ 2     │ 0          │ 0            │
└─────────┴─────────┴───────┴────────────┴──────────────┘

Action suggestions:
  /i18n-sync-generate de   — fill 12 missing keys with TODO stubs
  /i18n-sync-prune         — remove 3 orphaned keys (fr: 1, zh: 2)
```

## Related commands

| Command | Purpose |
|---|---|
| `/i18n-sync-scan [locale]` | Scan a single locale |
| `/i18n-sync-generate [locale]` | Generate stubs for missing keys |
| `/i18n-sync-prune` | Remove orphaned keys |
