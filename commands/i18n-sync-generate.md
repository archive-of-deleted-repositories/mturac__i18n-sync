# /i18n-sync-generate

**Generate missing translation key stubs for a locale.**

Runs `i18n-generate` for the specified locale, creating placeholder entries with `TODO:` markers for every missing key.

## Usage

```
/i18n-sync-generate [locale]
/i18n-sync-generate de
/i18n-sync-generate fr
/i18n-sync-generate zh-Hans
```

`[locale]` is required. Run `/i18n-sync` first to identify which locales need stubs.

## What it does

1. Detects translation file format and locale files.
2. Computes missing keys for the target locale (same algorithm as `/i18n-sync-scan`).
3. Displays the list of missing keys with base locale values.
4. Generates a patch file containing only the missing keys, with values set to `TODO: <base value>`.
5. If the locale file does not yet exist, creates the full file with all keys stubbed.
6. Prints the exact JSON/YAML fragments to insert manually if preferred.

## Behavior

- **Never overwrites** existing translations.
- **Writes a patch file** (e.g. `locales/de.patch.json`) — not the locale file itself.
- If the locale file does not exist, writes the full file directly.
- Merge command is printed for easy application.

## Example output

```
Locale: de — 12 missing keys

  home.hero.cta           en: "Get started"
  home.hero.subtitle      en: "Build faster with confidence"
  dashboard.empty.label   en: "No items found"
  ...

Patch written: locales/de.patch.json

Preview (first 3 keys):
{
  "home": {
    "hero": {
      "cta": "TODO: Get started",
      "subtitle": "TODO: Build faster with confidence"
    }
  },
  "dashboard": {
    "empty": {
      "label": "TODO: No items found"
    }
  }
}

Merge command:
  jq -s '.[0] * .[1]' locales/de.json locales/de.patch.json > locales/de.merged.json
  mv locales/de.merged.json locales/de.json
```

## Related commands

| Command | Purpose |
|---|---|
| `/i18n-sync` | Full scan first to see what needs generating |
| `/i18n-sync-scan de` | Detailed scan for a single locale |
| `/i18n-sync-prune` | Remove orphaned keys |
