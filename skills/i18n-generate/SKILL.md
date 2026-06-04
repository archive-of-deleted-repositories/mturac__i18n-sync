---
name: i18n-generate
description: Generate missing translation placeholder stubs for a target locale. Produces a patch file with TODO markers ready for translators.
version: 0.1.0
author: mturac
tags: [i18n, translations, generate, stubs]
---

# i18n-generate

Generates placeholder entries for every key missing in a target locale. Does not overwrite existing translations — only adds what is absent.

## Step 1 — Identify missing keys

Run the same detection and parsing logic as `i18n-scan` (Steps 1–4) to build the full list of missing keys for the specified locale.

Display the list before generating anything:

```
Locale: de
Missing keys (12):

  home.hero.cta           (en: "Get started")
  home.hero.subtitle      (en: "Build faster with confidence")
  dashboard.empty.label   (en: "No items found")
  dashboard.empty.action  (en: "Create your first item")
  auth.forgot.link        (en: "Forgot password?")
  auth.forgot.success     (en: "Check your email")
  errors.network.timeout  (en: "Request timed out")
  errors.network.retry    (en: "Retry connection")
  errors.403.title        (en: "Access denied")
  errors.403.body         (en: "You don't have permission to view this page.")
  settings.theme.label    (en: "Appearance")
  settings.theme.auto     (en: "System default")

Generating stubs for de...
```

## Step 2 — Generate patch content with TODO markers

For each missing key, produce a stub value in this format:

```
"TODO: <base locale value>"
```

Example:
```json
{
  "home": {
    "hero": {
      "cta": "TODO: Get started",
      "subtitle": "TODO: Build faster with confidence"
    }
  },
  "dashboard": {
    "empty": {
      "label": "TODO: No items found",
      "action": "TODO: Create your first item"
    }
  }
}
```

For YAML:
```yaml
home:
  hero:
    cta: "TODO: Get started"
    subtitle: "TODO: Build faster with confidence"
```

For PO files:
```po
#, fuzzy
msgid "home.hero.cta"
msgstr "TODO: Get started"
```

For TypeScript const:
```ts
// TODO: translate the following keys
"home.hero.cta": "TODO: Get started",
```

## Step 3 — Output patch file

Write the patch to a separate file — never directly overwrite the locale file:

```
Patch file written: locales/de.patch.json

Apply with:
  i18n-sync-apply de  (not yet implemented — merge manually or with jq)

Manual merge (JSON):
  jq -s '.[0] * .[1]' locales/de.json locales/de.patch.json > locales/de.merged.json
  mv locales/de.merged.json locales/de.json
```

If the target locale file does not exist yet, generate the full file (not a patch) and write it directly to the expected path:

```
New file created: locales/de.json
All 142 keys populated with TODO markers.
```

## Step 4 — Show exact structure to add

For every missing key, also print the minimal JSON fragment that must be inserted into the existing locale file, respecting the correct nesting position:

```
Add to locales/de.json under "home" > "hero":
  "cta": "TODO: Get started",
  "subtitle": "TODO: Build faster with confidence"

Add to locales/de.json under "errors" > "network":
  "timeout": "TODO: Request timed out",
  "retry": "TODO: Retry connection"
```

This lets developers apply changes manually with precision even without tooling support.
