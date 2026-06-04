---
name: i18n-prune
description: Safely remove orphaned translation keys that exist in locale files but not in the base locale. Always shows a preview before writing.
version: 0.1.0
author: mturac
tags: [i18n, translations, prune, cleanup]
---

# i18n-prune

Removes stale translation keys from all non-base locale files. A key is considered orphaned if it exists in a locale file but has no corresponding key in the base locale.

**Safety guarantee: no file is modified until the user explicitly confirms.**

## Step 1 — Detect orphaned keys

Run the full detection and parsing pipeline (same as `i18n-scan` Steps 1–5).

For each non-base locale, compute:

```
orphaned(locale) = localeKeys - baseKeys
```

## Step 2 — Show preview (mandatory before any write)

Display a full preview of every key that would be removed, grouped by locale and file:

```
i18n-sync Prune Preview
========================

Orphaned keys found across 3 locales:

--- fr (locales/fr.json) — 2 keys to remove ---
  home.hero.subtitle_old    "Construisez plus vite"
  auth.login.remember_me    "Se souvenir de moi"

--- de (locales/de.json) — 1 key to remove ---
  deprecated.banner.text    "Dieses Feature wurde entfernt"

--- zh (locales/zh/common.json) — 4 keys to remove ---
  v1.dashboard.header       "仪表板"
  v1.dashboard.sidebar      "侧边栏"
  v1.dashboard.footer       "页脚"
  v1.settings.old_theme     "旧主题"

Total: 7 keys across 3 files.

IMPORTANT: These keys will be permanently deleted from locale files.
Type YES to proceed, or anything else to cancel:
```

## Step 3 — Require explicit confirmation

Wait for the user to type `YES` (case-sensitive) before proceeding. Any other input cancels the operation with the message:

```
Prune cancelled. No files were modified.
```

## Step 4 — Create backups before writing

Before modifying any file, write a timestamped backup copy:

```
Backup created: locales/fr.json.bak.2024-01-15T143022
Backup created: locales/de.json.bak.2024-01-15T143022
Backup created: locales/zh/common.json.bak.2024-01-15T143022
```

Format: `<original-path>.bak.<ISO8601-without-colons>`

## Step 5 — Remove orphaned keys and write files

For each locale file:

1. Parse the current file content.
2. Remove the identified orphaned keys from the parsed structure. For nested JSON/YAML, remove only the specific leaf key (do not remove parent nodes unless they become empty).
3. If a parent node becomes empty after removal, remove the empty parent as well (clean up).
4. Serialize back to the original format, preserving indentation (2 spaces for JSON by default; match existing file if detectable).
5. Write the updated file.

Report progress:

```
Processing fr (locales/fr.json)...
  Removed: home.hero.subtitle_old
  Removed: auth.login.remember_me
  Written: locales/fr.json

Processing de (locales/de.json)...
  Removed: deprecated.banner.text
  Written: locales/de.json

Processing zh (locales/zh/common.json)...
  Removed: v1.dashboard.header
  Removed: v1.dashboard.sidebar
  Removed: v1.dashboard.footer
  Removed: v1.settings.old_theme
  Written: locales/zh/common.json
```

## Step 6 — Final summary

```
Prune complete.
  Files modified: 3
  Keys removed: 7
  Backups saved with suffix .bak.2024-01-15T143022

Run /i18n-sync to verify the locales are now in sync.
```

## Undo instructions

If the user wants to revert:

```
Restore backups:
  cp locales/fr.json.bak.2024-01-15T143022 locales/fr.json
  cp locales/de.json.bak.2024-01-15T143022 locales/de.json
  cp locales/zh/common.json.bak.2024-01-15T143022 locales/zh/common.json
```
