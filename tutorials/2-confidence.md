# Filter resumes for human review with confidence scores

LLM-powered parsers sometimes fabricate. They might invent a phone
number that "looks plausible" or assign a skill that's only loosely
implied by the resume. The Resume Parser API ships **per-section
confidence scores** so you can route the suspicious parses to a human
queue while auto-processing the rest.

This tutorial shows the pattern most ATS / recruitment platforms use
to turn that signal into an actionable workflow.

**You'll need**: a working `/parse` call (see the
[Quickstart](./1-quickstart.md)) and Python 3.9+.

## What the scores actually mean

Every parse returns 8 confidence scores under `resume.confidence`,
one per section:

```json
"confidence": {
  "identity":         { "value": 1.0,  "reason": null },
  "contact":          { "value": 0.85, "reason": null },
  "summary":          { "value": 1.0,  "reason": null },
  "work_experiences": { "value": 0.65, "reason": "End date for the 2019 role inferred — not in source" },
  "educations":       { "value": 1.0,  "reason": null },
  "skills":           { "value": 0.92, "reason": null },
  "languages":        { "value": 1.0,  "reason": null },
  "certifications":   { "value": 1.0,  "reason": null }
}
```

The scale is the same across sections:

| Range | What it means |
|---|---|
| **1.0** | Section is verifiable in source text — or correctly empty |
| **0.7 – 0.9** | Minor unverifiable details (dates approximate, location guessed) |
| **0.4 – 0.6** | Significant unverifiable claims |
| **0.0 – 0.3** | Section largely fabricated or missed |

When a section drops below **0.7**, the `reason` field contains a
human-readable explanation of what couldn't be verified.

> **Why per-section, not per-resume?** Confidence is naturally
> uneven. The identity is often perfectly extracted while the
> work-experience section has a fuzzy date — averaging into a single
> score would hide that. With 8 scores, you can route on the exact
> field that matters to your downstream workflow.

## The 3-bucket review pattern

The common pattern is: **auto-accept the high-confidence parses,
queue the medium ones, reject the lowest**.

```python
import requests

def classify(parse_response: dict) -> str:
    """Bucket a parse as 'auto', 'review', or 'reject'."""
    conf = parse_response["resume"]["confidence"]
    scores = [section["value"] for section in conf.values()]

    if min(scores) >= 0.85:
        return "auto"           # all sections solid → auto-accept
    if min(scores) >= 0.50:
        return "review"         # at least one shaky section → queue for human
    return "reject"             # something is badly broken → ask candidate to re-upload

def collect_review_reasons(parse_response: dict) -> list[str]:
    """Surface the reason strings for any sub-0.7 section.

    These are the talking points your reviewers see in the queue UI.
    """
    return [
        f"{name}: {section['reason']}"
        for name, section in parse_response["resume"]["confidence"].items()
        if section["value"] < 0.7 and section["reason"]
    ]
```

You'd then wire it into your intake flow:

```python
resp = requests.post(
    "https://resume-parser20.p.rapidapi.com/parse",
    headers={"X-RapidAPI-Key": API_KEY, "X-RapidAPI-Host": HOST},
    files={"file": (filename, file_bytes, "application/pdf")},
    timeout=60,
)
data = resp.json()

bucket = classify(data)
match bucket:
    case "auto":
        ingest_to_pipeline(data["resume"], candidate_id=candidate_id)
    case "review":
        reasons = collect_review_reasons(data)
        enqueue_for_review(data["resume"], candidate_id=candidate_id, reasons=reasons)
    case "reject":
        notify_candidate_reupload(candidate_id=candidate_id)
```

## Tuning the thresholds

The defaults above (0.85 / 0.50) are a reasonable starting point but
the right values depend on your tolerance vs your team's review
bandwidth:

- **Strict (0.95 / 0.75)**: high accuracy, high review load. Good for
  regulated industries (finance, healthcare) where a misparse
  carries cost.
- **Loose (0.75 / 0.40)**: fewer humans involved, occasional bad
  data slips through. Good for high-volume top-of-funnel where you'd
  rather re-verify later than slow down intake.

Watch the distribution of `min(scores)` across your first 1,000
parses — that histogram tells you where to put the cuts. If 80% of
parses already sit above 0.85, your bottleneck is the 20% — set the
"review" cutoff to where adding more reviewers actually moves the
needle.

## What to show your reviewers

The `reason` field on each sub-0.7 section is written specifically
to be reviewer-friendly. Example reasons you'll see in production:

- `"End date for the 2019 role inferred — not in source"`
- `"Phone number formatting suggests E.164 but country code unverified"`
- `"3 skills present in source but couldn't be mapped to ESCO"`

Surface those next to the candidate's resume PDF in your review UI
and the reviewer can resolve most cases in under 30 seconds.

## What to do next

- **[Map candidates to your skill graph using ESCO URIs](./3-esco.md)** — the next layer after intake quality is intake structure.
- **[Add resume upload to a Next.js application](./4-nextjs.md)** — wire this into a candidate-facing form.

For the canonical reference on every confidence field, see the
[developer guide](../RAPIDAPI.md#confidence-scores).
