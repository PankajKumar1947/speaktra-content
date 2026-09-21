# speaktra-content

Content repository for **Speaktra** — domain-based word banks and metadata served as static JSON.

All JSON files are hosted directly from GitHub via the **jsDelivr CDN**.

## Repository Structure

```
.
├── domains.json                  # List of all domains (id, name, description)
└── word-banks/
    ├── technology/
    │   ├── beginner.json
    │   ├── intermediate.json
    │   └── advanced.json
    ├── business/{beginner,intermediate,advanced}.json
    ├── student/{beginner,intermediate,advanced}.json
    ├── medical/{beginner,intermediate,advanced}.json
    ├── finance/{beginner,intermediate,advanced}.json
    ├── law/{beginner,intermediate,advanced}.json
    ├── education/{beginner,intermediate,advanced}.json
    └── science/{beginner,intermediate,advanced}.json
```

## Hosting — jsDelivr CDN (No Extra Deployment Needed)

Since your data is already in GitHub, use **jsDelivr** to serve it globally:

**Base URL pattern:**

```
https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/<path>
```

### Examples

| File | jsDelivr URL |
|------|--------------|
| `domains.json` | `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/domains.json` |
| `technology/beginner` | `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/technology/beginner.json` |
| `technology/intermediate` | `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/technology/intermediate.json` |
| `business/beginner` | `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/business/beginner.json` |
| Any domain/level | `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/<domain>/<level>.json` |

> Replace `PankajKumar1947/speaktra-content` if you fork the repo, and `@main` with a tag like `@v1.0.1` for version pinning (see below).

### Usage in App

```js
// Fetch all domains
const domains = await fetch(
  'https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/domains.json'
).then(r => r.json());

// Fetch word bank for a domain + level
async function getWordBank(domain, level) {
  const url = `https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/${domain}/${level}.json`;
  const res = await fetch(url);
  if (!res.ok) throw new Error(`Failed to fetch ${domain}/${level}`);
  return res.json();
}

// Example
const words = await getWordBank('technology', 'beginner');
```

### Why jsDelivr vs `raw.githubusercontent.com`?

| Feature | `raw.githubusercontent.com` | `cdn.jsdelivr.net/gh` |
|---------|-----------------------------|------------------------|
| Rate limits | 60 req/hour (unauthenticated) | **Unlimited** |
| Caching | Short `max-age`, no edge cache | Global edge cache (Cloudflare + Fastly) |
| Bandwidth | Limited | Unlimited, free |
| Latency | Single origin | Worldwide edge nodes |
| Purge control | No | `purge.jsdelivr.net` API + auto-purge on push |

Do **not** use `raw.githubusercontent.com` in production.

## Cache Invalidation / Purge

jsDelivr caches files aggressively. When you push new JSON to `main`, the CDN needs to be purged.

### 1. Automatic purge (configured)

This repo includes a GitHub Action at [`.github/workflows/purge-jsdelivr.yml`](.github/workflows/purge-jsdelivr.yml) that runs on every push to `main` that changes `**.json` files:

- Detects changed JSON files via `git diff`
- Calls `https://purge.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/<file>` for each file
- Falls back to purging all known JSON files if diff is unavailable (first push / force push)
- Runs with retries and logs results

No manual step needed — just `git push`.

### 2. Manual purge

Purge a single file:

```bash
curl https://purge.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/word-banks/technology/beginner.json
curl https://purge.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/domains.json
```

Purge all word banks (example script):

```bash
for f in word-banks/*/*.json domains.json; do
  echo "Purging $f..."
  curl -s "https://purge.jsdelivr.net/gh/PankajKumar1947/speaktra-content@main/$f" | cat
  echo
done
```

You can also trigger it manually from **Actions → Purge jsDelivr Cache → Run workflow** in the GitHub UI.

### 3. Version pinning with tags (instant, no purge wait)

For production stability, create a Git tag/release and pin the CDN URL to it — new tags are always fresh, no purge needed:

```bash
git tag v1.0.1
git push origin v1.0.1
# Then create a Release on GitHub from the tag (optional but recommended)
```

Use in app:

```
https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@v1.0.1/word-banks/technology/beginner.json
https://cdn.jsdelivr.net/gh/PankajKumar1947/speaktra-content@v1.0.1/domains.json
```

Recommended flow:
- `latest / development` → `@main` (auto-purged)
- `production` → `@v1.x.x` (immutable, no cache issues)

## Contributing

1. Edit or add JSON under `word-banks/<domain>/<level>.json` or `domains.json`.
2. Validate JSON is well-formed (`jq empty <file>` or `python -m json.tool`).
3. Commit and push to `main` — CDN cache is purged automatically.
4. For breaking/content milestones, create a tagged release for version pinning.
