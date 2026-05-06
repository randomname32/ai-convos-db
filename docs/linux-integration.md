---
summary: "Linux support: Chrome and Firefox cookie extraction paths and decryption."
read_when:
  - Running on Linux
  - Debugging cookie extraction on Linux
  - Understanding platform differences
---

# Linux Support

`convos` works on Linux for all local sync operations and web fetching via Chrome or Firefox.

## Platform Detection

`PLATFORM = sys.platform` is set once at module load (`darwin`, `linux`, `win32`).
All platform branches key off this single constant.

## Chrome Cookies

**Paths checked (in order):**
- `~/.config/google-chrome/{profile}/Cookies`
- `~/.config/chromium/{profile}/Cookies`

**Decryption key:** `PBKDF2(b"peanuts", b"saltysalt", iterations=1, keylen=16)`

This is Chrome's standard fallback key when no desktop keyring (GNOME Keyring,
KWallet) is present. It works in headless environments and most desktop setups.

Encrypted cookie values start with `v10` and use AES-128-CBC with a zero IV.

## Firefox Cookies

**Paths checked (in order):**
- `~/.mozilla/firefox/` (native install)
- `~/snap/firefox/common/.mozilla/firefox/` (Snap)
- `~/.var/app/org.mozilla.firefox/.mozilla/firefox/` (Flatpak)

Firefox cookies are stored unencrypted in `cookies.sqlite` (SQLite).

When Firefox is running, the database is in WAL mode. The reader copies both
`cookies.sqlite` and `cookies.sqlite-wal` to a temp directory before opening,
so reads succeed without interfering with the running browser.

## TLS Impersonation

Both ChatGPT and Perplexity sit behind Cloudflare. The `cf_clearance` cookie
is bound to the browser's TLS fingerprint (JA3/JA4), so requests must use
`curl-cffi` to impersonate the browser's TLS stack.

Mapping: `firefox` -> `firefox133`, `chrome` -> `chrome136`, `safari` -> `safari17_0`.
`curl-cffi` is a required dependency and is installed automatically.

## Usage

```bash
convos sync                      # tries safari, chrome, firefox in order
convos doctor                    # shows cookie status per browser
CONVOS_CHATGPT_BROWSER=firefox convos sync
CONVOS_CLAUDE_BROWSER=chrome convos sync
```
