# Vocab Generation Rules

Use this doc when a user asks a coding agent to generate vocab for this repo. No rewriting needed — just follow it.

## Target Files

`word-banks/<domain>/<difficulty>.json` where `domain` is from `domains.json:1` (`technology`, `business`, `academics`, `medical`, `finance`, `law`, `science`) and `difficulty` is `easy` | `medium` | `hard`. Served via `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/<domain>/<difficulty>.json`.

> Rename note: old `beginner` → `easy`, `intermediate` → `medium`, `advanced` → `hard`. Existing files must be renamed, not duplicated.

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

- **1 theme = 35 words, ordered by teaching order.** Content repo does NOT fix words/day — the app decides daily serving count per difficulty at runtime (e.g. `easy: 5/day`, `medium: 3/day`, `hard: 2/day`, or any mix).
- App slices `words[theme]` sequentially: Day 1 = first `daily_count` words, Day 2 = next `daily_count`, etc., continuing across themes.
- If asked for `N words`: generate `ceil(N/35)` themes. Example: 280 words → 8 themes.
- If asked for `N days`: ask for `daily_count` (default `5` if not specified), then total words = `N × daily_count`, themes = `ceil(total/35)`. Example: 50 days × 5/day = 250 words → 8 themes (280 words, app uses first 250).
- If asked for `domain` + `difficulty` without count: generate 1 theme (35 words) for that `domain`/`difficulty`. If asked for `N themes`: generate `N × 35` words.
- Append to existing file: add to `themes` array and `words` object, never overwrite. No duplicate `word` across file.

## Principles

- Goal is **English communication**, not textbook definitions. Every word must be sayable in a real conversation tomorrow.
- Theme = situation (e.g. `in_a_video_call`, `explaining_a_bug`), not dictionary chapter.

## Difficulty (replaces Levels)

- `easy` (old `beginner`, A1–A2): survive — daily, concrete, high-frequency (e.g. `login`, `restart`).
- `medium` (old `intermediate`, B1–B2): collaborate — explain, update, feedback (e.g. `reproduce`, `priority`).
- `hard` (old `advanced`, C1–C2): lead & persuade — negotiate, mitigate risk (e.g. `trade-off`, `mitigate`).

Difficulty controls word complexity only, never daily count. Daily count is an app-side decision per user plan.

## Meaning & Word Rules

- Spoken, not dictionary: `to sign in to your computer or account` not `a temporary storage area...`
- 6–14 words, start with `to ...` (verb) or `a ...`/`the ...` (noun).
- Frequency > rarity, Indian workplace context, mix verbs/nouns, no proper nouns/abbreviations, no duplicates.

## Generation Task

When user says:
- `generate for 50 days` → ask which `domain`/`difficulty` and `daily_count` (default 5/day), then themes = `ceil(N × daily_count / 35)`.
- `generate for <domain>/<difficulty> [for N words/days/themes]` → generate exactly that many themes for that file. `difficulty` must be `easy` | `medium` | `hard`, never `beginner`/`intermediate`/`advanced`.

Output valid JSON only. Validate with `python -m json.tool word-banks/<domain>/<difficulty>.json` and `jq empty <file>`.
