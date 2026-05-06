# Changelog

## 0.3.0
- Add Linux support: Chrome cookie extraction via `~/.config/google-chrome` and `~/.config/chromium`, decryption key via PBKDF2 fallback.
- Add Firefox cookie support on Linux (native, Snap, Flatpak) and macOS; WAL-safe reads while browser is running.
- Add curl-cffi (required dependency) for browser TLS impersonation, fixing Cloudflare bot detection that blocked ChatGPT, Claude, and Perplexity web sync.
- Add Perplexity AI web conversation sync via `POST /rest/thread/list_ask_threads`.
- Add `doctor` checks for Firefox cookies and Perplexity.
- Fix SQL injection in `convos export` (source filter and output path now use parameterized queries).
- Fix SQLite URI construction in Chrome cookie reader (path percent-encoded via `urllib.parse.quote`).
- Raise line budget from 1000 to 1500.

## 0.2.0
- Add opt-in hybrid semantic search with `convos query`: BM25 + local embeddings + Qwen3 reranking.
- Add `convos embed` to backfill embeddings without fetching new web conversations.
- Preserve existing embeddings during sync unless message content changes.
- Document the `[hybrid]` extra and hybrid search database schema.

## 0.1.3
- Auto-discover Chrome profiles for ChatGPT sync.

## 0.1.2
- Fix ChatGPT web sync for workspace accounts.
- Add optional parse error logging and Chrome profile selection.

## 0.1.1
- Sync output: per-service updated/new convo counts, totals, and timings with -v.
- Fix Claude no-op sync to avoid full re-fetch when unchanged.
- Improve local sync to only reparse changed Codex/Claude Code sessions.
- Add repo-local UV cache wrapper and install script cache default.
- README install command and headings cleanup.
