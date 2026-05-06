---
summary: "Perplexity AI integration: API endpoints and cookie auth."
read_when:
  - Fetching Perplexity conversations
  - Debugging Perplexity API issues
  - Understanding Perplexity data structure
---

# Perplexity Integration

Fetches threads from perplexity.ai using browser cookies.

## Web API Fetching

### Endpoints

**List threads (paginated):**
```
POST https://www.perplexity.ai/rest/thread/list_ask_threads
Body: {"offset": 0, "limit": 20}
```

Response (list):
```json
[
  {
    "uuid": "thread-uuid",
    "title": "Thread title",
    "query_str": "User query text",
    "first_answer": "{\"answer\": \"AI answer text\"}",
    "display_model": "pplx_pro",
    "last_query_datetime": "2024-01-01T00:00:00",
    "query_count": 1,
    "has_next_page": true,
    "total_threads": 99
  }
]
```

Paginate by incrementing `offset` by 20 until `has_next_page` is false.

Note: `GET /rest/thread/list_recent?offset=N` ignores the offset parameter
and always returns the same 20 threads. Use POST `list_ask_threads` instead.

**Get thread detail (multi-turn threads):**
```
GET https://www.perplexity.ai/rest/thread/{uuid}
```

Response:
```json
{
  "entries": [
    {
      "uuid": "entry-uuid",
      "query_str": "User query",
      "display_model": "pplx_pro",
      "last_query_datetime": "2024-01-01T00:00:00",
      "text": "[{\"step_type\": \"INITIAL_QUERY\", ...}, {\"step_type\": \"FINAL\", \"content\": {\"answer\": \"...\"}}]"
    }
  ],
  "thread_metadata": {
    "title": "Thread title",
    "created_at": "2024-01-01T00:00:00",
    "updated_at": "2024-01-01T00:00:00"
  },
  "has_next_page": false,
  "next_cursor": null
}
```

### Answer Parsing

The `text` field in each entry is a JSON-encoded array of steps. The AI answer
is in the `FINAL` step, and `content.answer` is itself a JSON-encoded string:

```python
steps = json.loads(entry["text"])
final = next(s for s in steps if s["step_type"] == "FINAL")
ans = json.loads(final["content"]["answer"]).get("answer", "")
```

For single-turn threads (`query_count == 1`), `first_answer` in the list
response contains the same JSON-encoded answer string, avoiding an extra request.

### Cookie Requirements

Valid session cookies from perplexity.ai:
- `__Secure-next-auth.session-token`
- `cf_clearance` (Cloudflare challenge token)

### TLS Requirement

Perplexity sits behind Cloudflare. `cf_clearance` is bound to the browser's
TLS fingerprint, so requests must use `curl-cffi` with browser impersonation.
Install `curl-cffi` (`pip install curl-cffi`) or install the full package.

## Usage

```bash
convos sync                      # auto-detects Firefox cookies on Linux
CONVOS_PERPLEXITY_BROWSER=firefox convos sync
```

## Troubleshooting

**403 Forbidden:**
- Cookies may be expired - log into perplexity.ai in your browser

**No threads fetched:**
- Check `convos doctor` shows `perplexity.ai=yes` for your browser
