# Trademark247 API

**Fast, no-friction access to US trademark + UDRP/WIPO domain-dispute data — built to be integrated by scripts, bots, and AI agents in minutes.**

Tell your favorite AI agent (e.g. Claude or ChatGPT) to visit this URL and learn how to integrate
the API into your workflow. All the data is available through a single JSON endpoint — pull
trademark and domain-dispute records straight into your own dataset. No SDK, no OAuth, no scraping,
no HTML to parse.

---

## Why this exists

- **Private by design — we keep no access logs.** Your IP address and your search terms are
  never recorded — for API calls or normal site use. We run no analytics, build no profiles, and
  never tie requests to your identity or sell/share activity. The only usage data we keep is an
  aggregate hit count per API key (for fair-use limits) — never your query, never who you are. Your
  searches are yours.
- **Built for automation & AI.** Clean, predictable JSON over HTTPS with a single header for
  auth. Drop it into a script, a data pipeline, or whatever AI agent you're using — it
  just works. (These docs live on GitHub on purpose: openly readable by bots and humans alike, with
  nothing gating access.)
- **Fast.** Typically responds in tens of milliseconds.
- **Two datasets, one call.** Search **trademarks** and **UDRP/WIPO domain decisions** together,
  or either one on its own.

> **Roadmap:** In addition to this search/detail API, you'll soon be able to **download the entire
> dataset** for fully local, offline access — pick whichever fits your workflow. *(Not yet available;
> [open an issue](../../issues) if you want early access.)*

---

## Base URL

```
https://trademark247.com
```

## Get an API key

Keys are fixed strings issued to you. **To request one, email `api@trademark247.com`** (or open an
issue on this repo). There's no signup form and no dashboard — you get a key, you use it.

## Authentication

Send your key in a header on every request:

```
X-API-Key: <your key>
```

Requests without a valid key return `401`.

---

## Endpoint

### `GET /api/v1/search`

| Param  | Required | Values                          | Notes |
|--------|----------|---------------------------------|-------|
| `type` | yes      | `all` · `trademark` · `udrp`    | Which dataset(s) to search. |
| `q`    | yes      | any string                      | Your search term. Cleaned automatically the same way the site search box does (trademarks: space-insensitive; domains: reduced to `a–z0–9`). |
| `live` | no       | `1`                             | Trademarks only — return just **live** marks (excludes dead/abandoned/expired). Ignored for UDRP. |

Returns the total match **count** for each dataset plus the **first 25 results**.

- `type=all` → `trademark_count` + `trademark_results` **and** `udrp_count` + `udrp_results`
- `type=trademark` → `trademark_count` + `trademark_results`
- `type=udrp` → `udrp_count` + `udrp_results`

---

## Examples

**curl**

```bash
curl -s -H "X-API-Key: YOUR_KEY" \
  "https://trademark247.com/api/v1/search?type=all&q=nike"
```

Search only live trademarks, term with spaces (let curl encode it):

```bash
curl -s -G "https://trademark247.com/api/v1/search" \
  -H "X-API-Key: YOUR_KEY" \
  --data-urlencode "type=trademark" \
  --data-urlencode "q=coca cola" \
  --data-urlencode "live=1"
```

**Python**

```python
import requests

r = requests.get(
    "https://trademark247.com/api/v1/search",
    headers={"X-API-Key": "YOUR_KEY"},
    params={"type": "all", "q": "nike"},
)
data = r.json()
print(data["trademark_count"], data["udrp_count"])
```

**Node**

```js
const r = await fetch(
  "https://trademark247.com/api/v1/search?type=udrp&q=nike",
  { headers: { "X-API-Key": "YOUR_KEY" } }
);
const data = await r.json();
```

---

## Response

```json
{
  "status": "ok",
  "type": "all",
  "query": "nike",
  "live": false,
  "trademark_count": 554,
  "trademark_results": [
    {
      "serial": "99607511",
      "mark": "NIKE",
      "owner_name": "NIKE, INC.",
      "status_name": "Published for Opposition",
      "is_alive": true,
      "file_date": "2026-01-21",
      "gs_description": "Jewelry",
      "image_thumb_url": "https://tsdr.uspto.gov/img/99607511/thumbnail"
    }
  ],
  "udrp_count": 251,
  "udrp_results": [
    {
      "domain": "nike-licensing.com",
      "status": "Transferred",
      "case_number": "D2025-5189",
      "ruleset": "WIPO",
      "case_name": "NIKE, Inc. v Robert Choe",
      "commence_date": null,
      "decision_date": "2026-02-02",
      "source": "wipoint",
      "source_id": 80814,
      "case_url": "https://www.wipo.int/amc/en/domains/search/case.jsp?case_id=80814"
    }
  ]
}
```

### Trademark result fields

| Field | Description |
|-------|-------------|
| `serial` | USPTO serial number |
| `mark` | Mark text |
| `owner_name` | Current owner |
| `status_name` | USPTO status (e.g. *Registered*, *Abandoned*) |
| `is_alive` | `true` if the mark is live |
| `file_date` | Filing date (`YYYY-MM-DD`) |
| `gs_description` | Goods & services (truncated) |
| `image_thumb_url` | Mark image thumbnail (USPTO TSDR) |

### UDRP result fields

| Field | Description |
|-------|-------------|
| `domain` | Disputed domain |
| `status` | Outcome — *Transferred, Cancelled, Withdrawn, Claim Denied, Split Decision, Pending, …* |
| `case_number` | Provider case number (e.g. `D2025-5189`) |
| `ruleset` | `WIPO`, `UDRP`, `URS`, … |
| `case_name` | *Complainant v Respondent* |
| `commence_date` / `decision_date` | Dates (`YYYY-MM-DD`, may be `null`) |
| `source` / `source_id` | Provider + provider-internal id |
| `case_url` | Link to the full decision (when available) |

---

## Status codes

| Code | Meaning |
|------|---------|
| `200` | OK |
| `400` | Missing `q` or invalid `type` |
| `401` | Missing or invalid API key |
| `500` | Server error |

Error bodies are JSON: `{ "status": "error", "message": "..." }`.

---

## Data & freshness

- **Trademarks:** ~14M US marks, synchronized **daily** from the USPTO.
- **UDRP/WIPO:** domain-dispute decisions, updated **daily** from WIPO.

## Privacy

We don't track you, and we don't keep a history of what you search — for API calls or normal site
use. **We keep no web-server access logs**: your IP address and search terms are never recorded. We
run no usage analytics, build no profiles, and never tie requests to your identity or sell/share your
activity. The only usage data we keep is an aggregate hit count per API key, used for fair-use limits
— it never includes your query. (We retain only minimal server error diagnostics needed to keep the
service running.)

## License

Released under the [MIT License](LICENSE) — permissive; use it however you like.
