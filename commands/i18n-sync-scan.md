# /i18n-sync-scan

**Scan a specific locale against the base locale.**

Runs `i18n-scan` for a single target locale and prints a detailed report of discrepancies.

## Usage

```
/i18n-sync-scan [locale]
/i18n-sync-scan fr
/i18n-sync-scan de
/i18n-sync-scan zh-Hans
```

If `[locale]` is omitted, lists all detected locales and prompts the user to pick one.

## What it does

1. Auto-detects translation file format and locale files (same as `/i18n-sync`).
2. Parses only the base locale and the specified target locale.
3. Reports:
   - **Missing keys** — with base locale values shown inline for translator context.
   - **Extra keys** — orphaned entries not present in base.
   - **Duplicate values** — keys sharing the same translated string.
   - **Untranslated keys** — values identical to base locale.
4. Shows file path(s) being scanned.

## Example output

```
Scanning locale: de
File: locales/de.json
Base: locales/en.json (142 keys) | Target: de (130 keys)

--- Missing (12) ---
  home.hero.cta           en: "Get started"
  home.hero.subtitle      en: "Build faster with confidence"
  dashboard.empty.label   en: "No items found"
  dashboard.empty.action  en: "Create your first item"
  auth.forgot.link        en: "Forgot password?"
  auth.forgot.success     en: "Check your email"
  errors.network.timeout  en: "Request timed out"
  errors.network.retry    en: "Retry connection"
  errors.403.title        en: "Access denied"
  errors.403.body         en: "You don't have permission to view this page."
  settings.theme.label    en: "Appearance"
  settings.theme.auto     en: "System default"

--- Extra (0) ---
  (none)

--- Duplicate values (0) ---
  (none)

--- Untranslated (4) ---
  footer.copyright        "© 2024 Acme Inc."
  meta.og.site_name       "Acme"
  nav.logo.alt            "Acme logo"
  common.loading          "Loading..."

Next step: /i18n-sync-generate de
```

## Related commands

| Command | Purpose |
|---|---|
| `/i18n-sync` | Scan all locales at once |
| `/i18n-sync-generate [locale]` | Generate stubs for missing keys |
| `/i18n-sync-prune` | Remove orphaned keys |
