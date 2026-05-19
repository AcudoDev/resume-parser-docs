# Get your first parse in 5 minutes

Five minutes from API key to a parsed resume. The `/parse` endpoint
takes a file and returns JSON. That's the whole interface. This page
covers three steps: get a key, send a request, read the response.

You need a RapidAPI account (free) and one of: `curl`, Python 3.9+,
or Node 18+.

## 1. Subscribe to a plan

The **Basic** tier is free forever. 50 calls per month, no credit
card. Click "Subscribe to Test" on the
[listing page](https://rapidapi.com/AcudoDev/api/resume-parser20)
and pick Basic.

Once subscribed, your `X-RapidAPI-Key` shows in the upper right of
any code snippet on the listing. Copy it.

## 2. Send your first request

Three equivalent snippets below. Each uploads a local PDF and prints
the parsed JSON. Pick whichever language you're on.

### cURL

```bash
curl -X POST "https://resume-parser20.p.rapidapi.com/parse" \
  -H "X-RapidAPI-Key: YOUR_KEY_HERE" \
  -H "X-RapidAPI-Host: resume-parser20.p.rapidapi.com" \
  -F "file=@./jane_doe.pdf"
```

### Python

```python
import requests

with open("jane_doe.pdf", "rb") as f:
    resp = requests.post(
        "https://resume-parser20.p.rapidapi.com/parse",
        headers={
            "X-RapidAPI-Key": "YOUR_KEY_HERE",
            "X-RapidAPI-Host": "resume-parser20.p.rapidapi.com",
        },
        files={"file": ("jane_doe.pdf", f, "application/pdf")},
        timeout=60,
    )

resp.raise_for_status()
data = resp.json()
print(data["resume"]["identity"]["full_name"])
print(data["resume"]["skills"][:3])
```

### JavaScript (Node)

```javascript
import fs from "node:fs";

const form = new FormData();
form.append("file", new Blob([fs.readFileSync("jane_doe.pdf")]), "jane_doe.pdf");

const r = await fetch("https://resume-parser20.p.rapidapi.com/parse", {
  method: "POST",
  headers: {
    "X-RapidAPI-Key": "YOUR_KEY_HERE",
    "X-RapidAPI-Host": "resume-parser20.p.rapidapi.com",
  },
  body: form,
});

const data = await r.json();
console.log(data.resume.identity.full_name);
console.log(data.resume.skills.slice(0, 3));
```

Expect **~15 seconds** on a text-native PDF. That's the LLM pipeline
running extract, ESCO-normalize and validate in series. Scanned PDFs
go through the vision model. Long resumes push p95 to ~22 seconds.

## 3. Read the response

A successful parse returns one JSON object with two top-level keys:
`resume` (the structured data) and `metadata` (request info).

```json
{
  "resume": {
    "schema_version": "1.0",
    "identity": { "full_name": "Jane Doe", "title": "Senior Engineer" },
    "contact": { "email": "jane@example.com", "city": "San Francisco" },
    "summary": "10+ years building distributed systems...",
    "work_experiences": [ { "company": "Acme Corp", "title": "...", ... } ],
    "educations": [ ... ],
    "skills": [
      {
        "name": "Python",
        "type": "hard",
        "esco_uri": "http://data.europa.eu/esco/skill/...",
        "esco_label": "Python (computer programming)"
      }
    ],
    "languages": [ { "name": "English", "iso_code": "en", "proficiency": "native" } ],
    "certifications": [],
    "detected_language": "en",
    "confidence": {
      "identity":         { "value": 1.0,  "reason": null },
      "skills":           { "value": 0.92, "reason": null },
      "work_experiences": { "value": 0.95, "reason": null },
      ...
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

The fields most consumers touch first:

| Path | What it is |
|---|---|
| `resume.identity.full_name` | Candidate name |
| `resume.contact.email` | Primary email if found |
| `resume.skills[].esco_uri` | ESCO canonical URI per skill (joinable across systems) |
| `resume.confidence.<section>.value` | 0.0 to 1.0 score per section, with a `reason` string when below 0.7 |
| `resume.detected_language` | `"en"` or `"fr"` |
| `metadata.request_id` | UUID to give us if you ever open a support ticket |

## 4. Handle the common errors

Errors come back as JSON with a `detail` field. The five you'll
hit most often:

| HTTP | Meaning | What to do |
|---:|---|---|
| **413** | File over 10 MB | Compress the PDF, or check the request isn't double-encoded |
| **415** | Unsupported MIME type | Convert to PDF, DOCX, image, or text |
| **422** | Document is empty, too short, or unreadable | Send a text-native PDF. The OCR fallback only runs on images |
| **429** | Rate limit hit on your plan | Back off, or upgrade to a higher tier |
| **504** | Pipeline stage exceeded its budget | Retry once. If it persists, the file likely has an extraction issue |

Always check `response.status_code` (or `response.ok`) before calling
`.json()`. The body of an error response is still valid JSON, but it
won't have the `resume` key.

## What to do next

Two related tutorials on this listing:

- **Filter resumes for human review with confidence scores**: the differentiator for HR-tech teams.
- **Map candidates to your skill graph using ESCO URIs**: turn the `esco_uri` field into a relational join key.

For the full reference (every field, every error code, every limit),
see the developer guide on this listing's About tab.
