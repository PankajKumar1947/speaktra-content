# Vocab Generation Rules

Use this doc when a user asks a coding agent to generate vocab for this repo. No rewriting needed — just follow it.

## Target Files

`word-banks/<domain>/<level>.json` where `domain` is from `domains.json:1` (`technology`, `business`, `academics`, `medical`, `finance`, `law`, `science`) and `level` is `beginner` | `intermediate` | `advanced`. Served via `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/<domain>/<level>.json`.

## Schema (must follow exactly)

```json
{
  "themes": ["theme_one"],
  "words": {
    "theme_one": [
      { "word": "login", "meaning": "to sign in to your computer or account" }
    ]
  }
}
```

- `themes`: snake_case, situation-based (e.g. `starting_your_workday`), order = teaching order.
- `words`: keys must match `themes`.
- `word`: lowercase, no period.
- `meaning`: one line, 6–14 words, spoken style, lowercase start, no period, no example sentence.
- No extra fields. `example`/`hinglish`/`pos` are generated in app.

## Sizing

- **1 day = 5 words. 1 theme = 1 week = 35 words (7 days × 5).** App slices `words[theme][0:5]` = Day 1.
- If asked for `N days`: generate `ceil(N/7)` themes = `ceil(N/7) × 35` words, then app uses first `N×5` words. Example: 50 days → 8 themes (280 words).
- If asked for `domain` + `level` without days: generate 1 theme (35 words) for that `domain`/`level`. If asked for `N themes`: generate `N × 35` words.
- Append to existing file: add to `themes` array and `words` object, never overwrite. No duplicate `word` across file.

## Principles

- Goal is **English communication**, not textbook definitions. Every word must be sayable in a real conversation tomorrow.
- Theme = situation (e.g. `in_a_video_call`, `explaining_a_bug`), not dictionary chapter.

## Levels

- `beginner` (A1–A2): survive — daily, concrete, high-frequency (e.g. `login`, `restart`).
- `intermediate` (B1–B2): collaborate — explain, update, feedback (e.g. `reproduce`, `priority`).
- `advanced` (C1–C2): lead & persuade — negotiate, mitigate risk (e.g. `trade-off`, `mitigate`).

## Meaning & Word Rules

- Spoken, not dictionary: `to sign in to your computer or account` not `a temporary storage area...`
- 6–14 words, start with `to ...` (verb) or `a ...`/`the ...` (noun).
- Frequency > rarity, Indian workplace context, mix verbs/nouns, no proper nouns/abbreviations, no duplicates.

## Generation Task

When user says:
- `generate for 50 days` → calculate themes = `ceil(50/7)=8`, distribute words across domains/levels as requested (or ask which domain/level), generate per schema.
- `generate for <domain>/<level> [for N days/weeks/themes]` → generate exactly that many themes for that file.

Output valid JSON only. Validate with `python -m json.tool word-banks/<domain>/<level>.json` and `jq empty <file>`.
