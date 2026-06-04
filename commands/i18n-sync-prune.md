# /i18n-sync-prune

**Remove orphaned translation keys from all locale files.**

Runs `i18n-prune` across every detected locale. Orphaned keys are those that exist in a locale file but have no corresponding key in the base locale.

**Safety: shows a full preview and requires explicit `YES` confirmation before modifying any file.**

## Usage

```
/i18n-sync-prune
```

No arguments. Operates on all non-base locales.

## What it does

1. Detects translation file format and locale files.
2. Computes orphaned keys for every non-base locale.
3. Prints a full preview: locale, file path, key path, current value.
4. Waits for `YES` confirmation.
5. Creates timestamped backup copies of every file to be modified.
6. Removes the orphaned keys and writes the updated files.
7. Prints a summary: files modified, keys removed, backup locations.

## Example output

```
i18n-sync Prune Preview
========================

Orphaned keys found across 2 locales:

--- fr (locales/fr.json) — 2 keys to remove ---
  home.hero.subtitle_old    "Construisez plus vite"
  auth.login.remember_me    "Se souvenir de moi"

--- zh (locales/zh/common.json) — 4 keys to remove ---
  v1.dashboard.header       "仪表板"
  v1.dashboard.sidebar      "侧边栏"
  v1.dashboard.footer       "页脚"
  v1.settings.old_theme     "旧主题"

Total: 6 keys across 2 files.

Backups will be saved with suffix: .bak.2024-01-15T143022

Type YES to proceed, anything else to cancel:
> YES

Backup created: locales/fr.json.bak.2024-01-15T143022
Backup created: locales/zh/common.json.bak.2024-01-15T143022

Processing fr...   removed 2 keys   ✓
Processing zh...   removed 4 keys   ✓

Prune complete. 6 keys removed from 2 files.
Run /i18n-sync to confirm locales are in sync.

To undo:
  cp locales/fr.json.bak.2024-01-15T143022 locales/fr.json
  cp locales/zh/common.json.bak.2024-01-15T143022 locales/zh/common.json
```

## Related commands

| Command | Purpose |
|---|---|
| `/i18n-sync` | Full scan to identify orphaned keys first |
| `/i18n-sync-scan [locale]` | Inspect a specific locale before pruning |
| `/i18n-sync-generate [locale]` | Fill missing keys with stubs |
