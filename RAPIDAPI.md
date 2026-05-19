# Resume Parser API — Developer Guide

Fast multilingual resume parser that turns PDF / DOCX / image / text
resumes into a structured `ResumeV1` JSON, including ESCO skill URIs
and per-section confidence scores.

- **Languages**: English, French
- **Formats**: PDF, DOCX, image (PNG/JPG with OCR fallback), plain text
- **Latency**: ~15s p50, ~22s p95 per resume
- **Output**: structured JSON validated against `ResumeV1` schema (versioned)

## Authentication

All requests are authenticated by RapidAPI gateway via the standard
header:

```
X-RapidAPI-Key: <your-key>
X-RapidAPI-Host: resume-parser20.p.rapidapi.com
```

If you call the underlying service directly (bypassing RapidAPI), include
`X-RapidAPI-User` to attribute requests in your audit log:

```
X-RapidAPI-User: alice@example.com
```

## Endpoints

### POST `/parse` — synchronous

Best for interactive use. Blocks until the parse completes (~15s p50).

**Request**: `multipart/form-data` with a single `file` field.

**cURL**:
```bash
curl -X POST "https://resume-parser20.p.rapidapi.com/parse" \
  -H "X-RapidAPI-Key: <your-key>" \
  -H "X-RapidAPI-Host: resume-parser20.p.rapidapi.com" \
  -F "file=@/path/to/jane_doe.pdf"
```

**Python**:
```python
import requests

resp = requests.post(
    "https://resume-parser20.p.rapidapi.com/parse",
    headers={
        "X-RapidAPI-Key": "<your-key>",
        "X-RapidAPI-Host": "resume-parser20.p.rapidapi.com",
    },
    files={"file": ("jane_doe.pdf", open("jane_doe.pdf", "rb"), "application/pdf")},
    timeout=60,
)
data = resp.json()
print(data["resume"]["identity"]["full_name"])
print(f"{data['resume']['skills'][0]['name']} → {data['resume']['skills'][0]['esco_uri']}")
```

**JavaScript (fetch)**:
```javascript
const form = new FormData();
form.append("file", fileInput.files[0]);

const r = await fetch("https://resume-parser20.p.rapidapi.com/parse", {
  method: "POST",
  headers: {
    "X-RapidAPI-Key": "<your-key>",
    "X-RapidAPI-Host": "resume-parser20.p.rapidapi.com",
  },
  body: form,
});
const data = await r.json();
console.log(data.resume.identity.full_name);
```

**Response (200)**: see [Response schema](#response-schema-resumev1).

### Processing multiple resumes

Call `/parse` in parallel from your client. The server handles
concurrent requests up to your plan's rate limit.

```python
import concurrent.futures, requests

HEADERS = {
    "X-RapidAPI-Key": "<your-key>",
    "X-RapidAPI-Host": "resume-parser20.p.rapidapi.com",
}

def parse_one(path):
    with open(path, "rb") as f:
        r = requests.post(
            "https://resume-parser20.p.rapidapi.com/parse",
            headers=HEADERS,
            files={"file": (path, f, "application/pdf")},
            timeout=120,
        )
    return path, r.json()

paths = ["jane.pdf", "alice.docx", "bob.pdf"]
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as ex:
    for path, result in ex.map(parse_one, paths):
        print(path, "→", result["resume"]["identity"]["full_name"])
```

## Response schema (ResumeV1)

```json
{
  "resume": {
    "schema_version": "1.0",
    "identity": {
      "full_name": "Jane Doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "title": "Senior Software Engineer"
    },
    "contact": {
      "email": "jane.doe@example.com",
      "phone": "+1-415-555-0142",
      "linkedin_url": "https://linkedin.com/in/janedoe",
      "city": "San Francisco",
      "country": "US"
    },
    "summary": "10+ years building distributed systems...",
    "work_experiences": [
      {
        "company": "Acme Corp",
        "title": "Senior Software Engineer",
        "start_date": "2020-03",
        "end_date": "present",
        "duration_months": 50,
        "location": "San Francisco, CA",
        "description": "...",
        "achievements": ["Led team of 6 to ship payments microservice"]
      }
    ],
    "educations": [...],
    "skills": [
      {
        "name": "Python",
        "type": "hard",
        "esco_uri": "http://data.europa.eu/esco/skill/...",
        "esco_label": "Python (computer programming)",
        "proficiency_level": null
      }
    ],
    "languages": [
      { "name": "English", "iso_code": "en", "proficiency": "native" }
    ],
    "certifications": [...],
    "detected_language": "en",
    "confidence": {
      "identity":         { "value": 1.0,  "reason": null },
      "contact":          { "value": 1.0,  "reason": null },
      "summary":          { "value": 1.0,  "reason": null },
      "work_experiences": { "value": 0.95, "reason": null },
      "educations":       { "value": 1.0,  "reason": null },
      "skills":           { "value": 0.92, "reason": null },
      "languages":        { "value": 1.0,  "reason": null },
      "certifications":   { "value": 1.0,  "reason": null }
    }
  },
  "metadata": {
    "request_id": "f3e2c0c0-1234-4abc-9def-0987654321ab",
    "schema_version": "1.0",
    "latency_ms": 15234,
    "pipeline_version": "v1.0.0"
  }
}
```

### Confidence scores

Each of the 8 sections has a `value` between 0.0 and 1.0:
- **1.0**: section verifiable in source text (or correctly empty)
- **0.7-0.9**: minor unverifiable details (dates approximate, location guessed)
- **0.4-0.6**: significant unverifiable claims
- **0.0-0.3**: section largely fabricated or missed

When `value < 0.7`, the `reason` field describes what couldn't be verified.

### Skill types

`skills[].type` is one of:
- `hard` — technical skill (Python, Excel, Photoshop, ...)
- `soft` — interpersonal (Communication, Leadership, ...)
- `language` — natural language (English, French, ...)
- `tool` — specific software / platform (AWS, Salesforce, ...)
- `certification` — named cert (AWS Solutions Architect, PMP, ...)
- `unknown` — type couldn't be confidently determined

### Language proficiency

`languages[].proficiency` follows the [CEFR scale](https://www.coe.int/en/web/common-european-framework-reference-languages/level-descriptions):
`A1`, `A2`, `B1`, `B2`, `C1`, `C2`, or `native`. `null` when no
proficiency was mentioned in the resume.

## Response codes

| HTTP | Meaning | What to do |
|---:|---|---|
| 400 | Too many files in a batch (>25), or empty file list | Split / fix request |
| 413 | File over 10 MB | Compress or send a different file |
| 415 | Unsupported MIME type | Convert to PDF / DOCX / image / text |
| 422 | Document is empty, too short (<50 chars), or scanned PDF | Send a text-native PDF; OCR fallback for images only |
| 429 | Per-IP rate limit hit (60 req/min default) | Back off; check `Retry-After` header |
| 500 | Internal pipeline error | Retry with backoff; if persists, contact support |
| 504 | Pipeline stage exceeded its budget (90s extract / 60s validate) | Retry; check that the file is parseable |

The body of any error is JSON: `{"detail": "human-readable explanation"}`.

## Limits

- **File size**: 10 MB per file (uncompressed)
- **Rate**: varies by your RapidAPI plan; see the Pricing tab
- **Pipeline budgets**: 90s extract, 60s validate, 30s per ESCO Tier-3 disambig
- **Quota**: 1 call deducts 1 unit from your monthly plan quota

Your RapidAPI subscription tier sets the monthly quota and per-second
rate limit; see the Pricing tab on the listing for the current values.

## Operational endpoints

- **GET /health** — liveness probe (200 if process is up)
- **GET /readyz** — readiness probe (200 only if all deps healthy)
- **GET /benchmark** — public 24h rolling stats (success rate, p50/p95/p99 latency)
- **GET /docs** — interactive OpenAPI UI

## Versioning & breaking changes

The response is versioned via `resume.schema_version` (currently `"1.0"`)
and `metadata.pipeline_version` (currently `"v1.0.0"`).

Breaking changes to `ResumeV1` will ship as `ResumeV2` on a separate
endpoint path; the old shape continues to be served until deprecation
notice (90+ days). Patch-level changes to the pipeline (better prompts,
new ESCO data, etc.) bump `pipeline_version` only — schema stays stable.

## Support

- Source: https://github.com/your-org/resume-parser
- Issues: https://github.com/your-org/resume-parser/issues
- Email: support@example.com
