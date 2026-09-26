# Vocab Generation Rules

Use this doc when generating vocab for this repo. No rewriting needed — just follow it.

## Target Files

`word-banks/<domain>/themes.json` (single source of truth for theme order) plus `word-banks/<domain>/<difficulty>.json` where `domain` is from `domains.json` (`technology`, `business`, `academics`, `medical`, `finance`, `law`, `science`) and `difficulty` is `easy` | `medium` | `hard`. Served via `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/<domain>/...`.

## Hard Requirements (API rejects anything else)

1. **Theme order lives in exactly one place: `word-banks/<domain>/themes.json`.** Difficulty files contain only `words` — no `themes` key, no duplication. The API reads rotation order from `themes.json` and looks up each theme in the difficulty files.
2. `words` keys must match `themes.json` exactly — every theme present in every difficulty file, no extras, no missing keys.
3. `word`: lowercase, no period. No duplicate `word` (case-insensitive) within a file, or across the three files of a domain.
4. `meaning`: one line, 6–14 words, spoken style, lowercase start, no period, no example sentence.
5. No extra fields. `example`/`hinglish`/`pos` are generated in-app.

## Schema (must follow exactly)

`word-banks/<domain>/themes.json`:

```json
{ "themes": ["theme_one", "theme_two"] }
```

`word-banks/<domain>/<difficulty>.json`:

```json
{
  "words": {
    "theme_one": [
      { "word": "login", "meaning": "to sign in to your computer or account" }
    ],
    "theme_two": [ ... ]
  }
}
```

- `themes`: snake_case, situation-based (e.g. `starting_your_workday`), order = teaching order. Fixed once published — never reorder, never insert mid-array, only append.
- `words`: keys must match `themes.json` exactly.

## How the App Serves

- One theme per **7-day block** (days 1–7 = `themes[0]`, days 8–14 = `themes[1]`, …), looping back after the last theme.
- Each day serves exactly **5 words** from the current theme, split by user level:

| Level | easy | medium | hard |
| ----- | ---- | ------ | ---- |
| beginner | 2 | 2 | 1 |
| intermediate | 1 | 2 | 2 |
| advanced | 0 | 2 | 3 |

- Within a bucket the app takes a window per day and **shifts it forward each full rotation**, so every word in a bucket is eventually served and appended words get picked up. Nothing outside the bucket is ever served — oversized buckets are not variety, they are waste (see sizing).
- Word `difficulty` in-app always comes from the filename. Never encode difficulty anywhere else.

## Sizing

- **Standard: 35 words per theme per difficulty file.** Minimums for a repeat-free week: **14 easy / 14 medium / 21 hard** per theme (7 days × max daily take). Below the minimum, words repeat within the same week.
- Counts are **per file**. A full domain drop = 3 files × N themes (e.g. 8 themes = 105 words per file, 315 per domain).
- If asked for `N words` (per file): generate `ceil(N/35)` themes. Example: 280 words → 8 themes.
- If asked for `N days`: daily take is fixed at 5/day by the level mix above. **Themes = `ceil(N/7)`**, each theme holding the 35-word standard. Example: 50 days → 8 themes (covers 56 days; app uses the first 50).
- If asked for `domain` + `difficulty` without count: generate 1 theme (35 words) **and** append that theme slug to `themes.json` plus matching `words` entries in the other two files (rule 2 — every theme must exist in every file).
- If asked for `N themes`: generate `N × 35` words in the target file, plus the `themes.json` append and matching `words` keys in the other two files.
- Append to existing file: add to `themes` array and `words` object, never overwrite.

## Principles

- Goal is **English communication**, not textbook definitions. Every word must be sayable in a real conversation tomorrow.
- Theme = situation (e.g. `in_a_video_call`, `explaining_a_bug`), not dictionary chapter.

## Difficulty

- `easy` (A1–A2): survive — daily, concrete, high-frequency (e.g. `login`, `restart`).
- `medium` (B1–B2): collaborate — explain, update, feedback (e.g. `reproduce`, `priority`).
- `hard` (C1–C2): lead & persuade — negotiate, mitigate risk (e.g. `trade-off`, `mitigate`).

Difficulty controls word complexity only, never daily count. Daily serving is fixed by the level mix above.

## Meaning & Word Rules

- Spoken, not dictionary: `to sign in to your computer or account` not `a temporary storage area...`
- 6–14 words, start with `to ...` (verb) or `a ...`/`the ...` (noun).
- Frequency > rarity, Indian workplace context, mix verbs/nouns, no proper nouns/abbreviations, no duplicates.

## Generation Task

When user says:
- `generate for 50 days` → ask which `domain`, then themes = `ceil(N/7)` added to **all three** difficulty files (35 words each per theme).
- `generate for <domain>/<difficulty> [for N words/days/themes]` → generate exactly that many themes for that file, plus the `themes.json` append and matching `words` keys in the other two files. `difficulty` is always one of `easy` | `medium` | `hard`.

Output valid JSON only.

## Validation (run before push)

```bash
# 1. Valid JSON
python -m json.tool word-banks/<domain>/themes.json > /dev/null
python -m json.tool word-banks/<domain>/<difficulty>.json > /dev/null

# 2. words keys match themes.json exactly in every file (must print nothing = pass)
for f in easy medium hard; do
  diff <(jq -c '.themes' word-banks/<domain>/themes.json) \
       <(jq -c --sort-keys '.words | keys' word-banks/<domain>/$f.json) && echo "$f KEYS_OK"
done

# 3. Per-theme counts (expect 35 each; min 14 easy / 14 medium / 21 hard)
jq -r '.themes[]' word-banks/<domain>/themes.json | while read t; do
  echo "$t: $(jq --arg t "$t" '.words[$t] | length' word-banks/<domain>/<difficulty>.json)"
done

# 4. No duplicate words across the domain (must print nothing = pass)
jq -r '.words[][] | .word' word-banks/<domain>/{easy,medium,hard}.json | tr '[:upper:]' '[:lower:]' | sort | uniq -d
```
