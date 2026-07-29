# AGENTS.md

## Cursor Cloud specific instructions

### What this repository is

This is a **data-only repository**, not an application. It stores Chinese provincial
electricity time-of-use (TOU) pricing period data (峰谷分时电价) as JSON.

- There is **no application code, no services, no dependencies, and no build/test/lint tooling**.
- The `main` branch is essentially a stub (`README.md` only).
- The actual data lives on **feature branches**, one per region/month, named
  `feat/parse-{region}_电价标准_{YYYY-MM}` (e.g. `feat/parse-江苏_电价标准_2026-07`).
- Each data branch contains exactly `{YYYY-MM}.json` plus a `README.md`.

### Data schema (non-obvious)

Each JSON file maps `user-category -> { "MM": "<slot string>" }`. The slot string encodes
the TOU classification of each half-hour of the day, so a full day is **48 characters**
(24h × 2 slots/hour). Some regions/months use an empty string when no TOU data exists.
Valid characters:

| Char | Meaning              |
|------|----------------------|
| 谷   | off-peak (valley)    |
| 平   | normal / flat        |
| 峰   | peak                 |
| 尖   | super-peak (sharp)   |

### Environment / dev setup

- **No install/update step is required.** There are no package managers or lockfiles.
- Pre-installed tooling in this environment is sufficient to work with the data:
  `git`, `python3`, `jq`, `node`.

### Working with / validating data

Do not switch off your working branch just to read data — inspect other branches with
`git show`:

```bash
# Read a data file from a data branch without checking it out
git show origin/feat/parse-江苏_电价标准_2026-07:2026-07.json | python3 -m json.tool

# Validate JSON + slot-string schema (48 chars, only 谷/平/峰/尖)
git show origin/feat/parse-江苏_电价标准_2026-07:2026-07.json | python3 -c '
import json, sys
data = json.load(sys.stdin)
valid = set("谷平峰尖")
for cat, months in data.items():
    for m, s in months.items():
        assert not (set(s) - valid), (cat, m, set(s) - valid)
        print(cat, m, "len", len(s))
'
```
