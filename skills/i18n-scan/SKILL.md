---
name: i18n-scan
description: Scan all locale translation files for missing, extra, duplicate, and untranslated keys. Outputs a per-locale report table.
version: 0.1.0
author: mturac
tags: [i18n, translations, localization, scan]
---

# i18n-scan

Audits every locale file in the project against the base locale (default: `en`) and produces a structured report.

## Step 1 — Auto-detect translation file format

Search the project for translation files. Supported formats:

| Format | Extensions | Detection rule |
|---|---|---|
| JSON nested | `.json` | Top-level values contain objects |
| JSON flat | `.json` | All top-level values are strings/numbers |
| YAML | `.yml`, `.yaml` | File extension |
| PO (GNU gettext) | `.po` | File starts with `msgid` blocks |
| TypeScript const | `.ts` | File contains `export const` + object literal |

Detection algorithm:
1. Glob `**/*.{json,yml,yaml,po,ts}` inside locale directories (see Step 2).
2. For `.json`: read the first file; if any value is a plain object, classify as JSON nested, else JSON flat.
3. For `.ts`: extract the exported const object literal using regex or AST walk.
4. If mixed formats are detected, warn the user and ask which format is canonical before continuing.

## Step 2 — Detect locale directories and files

Search in priority order:
1. Directories named exactly: `locales/`, `i18n/`, `translations/`, `lang/`, `messages/`, `locale/`
2. Files at any depth whose name matches a locale pattern: `en.json`, `fr-FR.yml`, `de.po`, `zh-Hans.ts`
3. Namespace structures: `locales/en/common.json`, `i18n/fr/messages.yml`

Build locale map:
```
{ "en": ["locales/en.json"], "fr": ["locales/fr.json"], "de": ["locales/de/common.json", "locales/de/errors.json"] }
```

Identify base locale:
1. Prefer file named `en.*`.
2. Fall back to `defaultLocale` / `fallbackLng` in `i18next.config.*`, `next-i18next.config.*`, or `astro.config.*`.
3. If unresolvable, prompt: "Which locale is the base/source locale? (e.g. en)"

## Step 3 — Parse and extract keys with dot notation

For each locale file, flatten nested keys to dot-notation strings:

```
Input:  { "home": { "title": "Welcome", "nav": { "about": "About" } } }
Output: { "home.title": "Welcome", "home.nav.about": "About" }
```

Flattening logic (pseudocode):
```
function flatten(obj, prefix=''):
  result = {}
  for key, value in obj:
    full = prefix ? prefix + '.' + key : key
    if value is plain object:
      merge result with flatten(value, full)
    else:
      result[full] = value
  return result
```

For PO files: map `msgid` → `msgstr` pairs. Treat `msgid` as the key, `msgstr` as the value. Skip entries where `msgstr` is empty (treat as missing).

Build final inventory:
```
Map<localeCode, Map<dotKey, translationValue>>
```

## Step 4 — Find MISSING keys

Keys present in the base locale but absent in a target locale.

```
missing(locale) = baseKeys - localeKeys
```

For each missing key, record:
- Key path (dot notation)
- Base locale value (for translator reference)

## Step 5 — Find EXTRA keys (orphaned)

Keys present in a target locale but not in the base locale.

```
extra(locale) = localeKeys - baseKeys
```

These are stale translations that no longer map to any source key. **Never auto-delete.** Only report.

## Step 6 — Find DUPLICATE values

Within a single locale, find keys that share the exact same translated value.

```
for each locale:
  valueMap = invert(keyMap)  # value -> [key, key, ...]
  duplicates = entries where len(keys) > 1
```

Flag these as possible accidental key reuse or copy-paste errors.

## Step 7 — Find UNTRANSLATED keys

Keys where the target locale value is **identical** to the base locale value (copy-pasted, not translated).

```
untranslated(locale, key) = localeValue[key] == baseValue[key]
```

Exclude keys that are intentionally the same (e.g. brand names, URLs). Flag but do not auto-fix.

## Step 8 — Output report table

Print a summary table per locale, then detailed lists.

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

--- fr: 3 Missing Keys ---
  home.hero.cta          (en: "Get started")
  dashboard.empty.label  (en: "No items found")
  errors.network.retry   (en: "Retry connection")

--- fr: 1 Extra Key ---
  home.hero.subtitle_old

--- fr: 2 Duplicate Values ---
  auth.login.submit == auth.signup.submit  ("Continue")

--- de: 4 Untranslated Keys ---
  footer.copyright    (value same as en: "© 2024 Acme Inc.")
  ...
```

## Step 9 — Guardrail

**NEVER auto-delete keys.** This skill is read-only. It only reports.

If the user asks to delete, respond:
> "Use `/i18n-sync-prune` which will show you exactly what will be removed and ask for confirmation before touching any file."
