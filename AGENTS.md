# Speaktra Content — Agent Guide

This repo is static JSON for Speaktra English communication app. Content is served via `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/<domain>/<difficulty>.json` (`README.md:29`).

## Before generating vocab

- Read `docs/VOCAB_GENERATION_RULES.md` — single source of truth for schema, sizing (1 theme = 35 words, app decides daily count), difficulty, and word rules.
- Check `domains.json:1` for valid `domain` (`technology`, `business`, `academics`, `medical`, `finance`, `law`, `science`) and `difficulty` (`easy` | `medium` | `hard`).
- Check existing `word-banks/<domain>/<difficulty>.json` to append (never overwrite) and avoid duplicates.

## After generating

- Validate: `python -m json.tool word-banks/<domain>/<difficulty>.json` and `jq empty <file>`.
- Do not auto-commit/push. Preview the commit message and ask for confirmation before committing. After push to `main`, `.github/workflows/purge-jsdelivr.yml:1` auto-purges CDN.
