# i18n-sync

A Claude Code plugin that keeps translation keys in sync across locales. Detects missing, extra, duplicate, and untranslated keys. Generates type-safe stubs for translators. Safely prunes orphaned entries.

## Install

```bash
claude plugin install i18n-sync
```

Or from local path (development):

```bash
claude plugin install /path/to/i18n-sync
```

## Commands

| Command | Description |
|---|---|
| `/i18n-sync` | Full scan of all detected locales |
| `/i18n-sync-scan [locale]` | Scan one locale vs. the base |
| `/i18n-sync-generate [locale]` | Generate TODO stubs for missing keys |
| `/i18n-sync-prune` | Remove orphaned keys (with confirmation) |

## Supported formats

- JSON (nested and flat)
- YAML (`.yml`, `.yaml`)
- PO / GNU gettext (`.po`)
- TypeScript `export const` objects

Detects locale files in `locales/`, `i18n/`, `translations/`, `lang/`, `messages/`.

## Example sync report

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

--- de: 12 Missing Keys ---
  home.hero.cta           en: "Get started"
  home.hero.subtitle      en: "Build faster with confidence"
  dashboard.empty.label   en: "No items found"
  ...

Action suggestions:
  /i18n-sync-generate de   — fill 12 missing keys with TODO stubs
  /i18n-sync-prune         — remove 3 orphaned keys (fr: 1, zh: 2)
```

## Generate stubs for missing keys

```
/i18n-sync-generate de

Patch written: locales/de.patch.json

{
  "home": {
    "hero": {
      "cta": "TODO: Get started",
      "subtitle": "TODO: Build faster with confidence"
    }
  }
}

Merge: jq -s '.[0] * .[1]' locales/de.json locales/de.patch.json > locales/de.merged.json
```

## Safety

- Prune always shows a preview and requires `YES` before writing.
- Backups are created before any file is modified (`.bak.<timestamp>`).
- Scan and generate are read-only (no file mutations).
