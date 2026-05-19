# Privacy Policy — Resume Parser API

_Last updated: 2026-05-19_

> **Plain-language summary**: We process the resume documents you send us
> for the sole purpose of returning a structured JSON parse, then
> discard them. We do **not** store, train on, share, or sell your
> resume content. We do keep minimal request metadata (request ID,
> timestamp, byte size, success status) for billing and abuse
> prevention, never the resume content itself.

This Privacy Policy applies to the Resume Parser API ("the Service",
"we", "us"), accessible via RapidAPI at
`https://rapidapi.com/<your-listing>/api/resume-parser-api`. It is
designed to comply with the EU **General Data Protection Regulation
(GDPR, Regulation (EU) 2016/679)** and the UK GDPR.

## 1. Data Controller

The data controller is the operator of the Service. Contact:
**support@your-domain.com** (replace before launch).

## 2. What we process

When you call the Service, you submit a resume document. That document
may contain personal data within the meaning of GDPR Art. 4(1) — most
typically:

- Full name, gender (inferable), date of birth (when present)
- Contact details (email, phone, postal address)
- Profile photo (when present)
- Employment history, education, certifications
- Languages, skills, references
- Any other content the document author chose to include

We are the **data processor** for this content; you (the API caller)
are the data controller in most use cases. If you are processing
resumes on behalf of third parties (e.g. as part of recruitment
software), you remain responsible for having a lawful basis under
GDPR Art. 6 to share them with us.

## 3. What we do with the data

The resume content is held **in memory only** for the duration of a
single API call (typically 10–25 seconds, never more than 90 seconds
which is our hard pipeline budget). During that time we:

1. Extract text and (for scanned PDFs) render page images.
2. Send the text/images to our LLM provider for parsing — see §5 below.
3. Match detected skills against the public ESCO taxonomy
   (https://esco.ec.europa.eu).
4. Return the structured JSON to you.
5. Discard the resume document from memory.

**We do not store the resume content on disk, in a database, in
backups, or in any persistent medium.** No human reads it. We do not
train any machine learning model on your data.

## 4. What we *do* keep (metadata only)

For each API call we keep a single audit row containing:

- Anonymous request UUID
- Timestamp (UTC, millisecond precision)
- Caller's RapidAPI user identifier (provided by RapidAPI gateway)
- File size in bytes, MIME type
- Detected language code (e.g. `en`, `fr`)
- Counts (skills, work experiences) and average confidence score
- Latency in milliseconds
- Success flag and error code (if any)

Retention: **90 days**, then automatically deleted. Purpose:
abuse prevention, billing reconciliation, performance monitoring,
and the public 24-hour rolling stats endpoint (`/benchmark`). Legal
basis under GDPR Art. 6: **legitimate interest** (operating a
secure, reliable API service).

## 5. Sub-processors

We rely on the following sub-processors. None of them receive
identifying metadata that would let them re-identify a specific
candidate; they only see the resume content as part of the LLM call.

| Sub-processor | Purpose | Data shared | DPA status |
|---|---|---|---|
| **OpenRouter** (Anthropic-routed LLM provider) | Resume parsing (text and vision) | Resume content as plain text or rendered images | Sub-processor under their TOS; opt-out of training enabled |
| **OpenAI** (via OpenRouter or direct) | LLM inference + skill embedding generation | Resume content (for parsing); skill names only (for embeddings) | OpenAI Data Processing Addendum signed; zero-retention API tier requested |
| **Railway** (infrastructure host) | Container hosting, persistent volume | None (no resume content reaches Railway storage) | EU region (Amsterdam) used; DPA available on request |
| **RapidAPI** (gateway) | Authentication, rate limiting, billing | Request metadata only (your API key, timestamp, response size) | Their own privacy policy applies |

We have selected providers that contractually commit to:
- Not training models on our customers' data without explicit opt-in.
- Not retaining inputs beyond what is strictly necessary to serve the
  request (with the exception of standard 30-day abuse-monitoring logs
  on the LLM providers' side).
- Operating in jurisdictions with adequate data protection (EU/EEA, UK,
  or US under Data Privacy Framework).

## 6. International transfers

Our infrastructure runs in the **EU (Amsterdam, Netherlands)**.
LLM inference may transit through OpenRouter / OpenAI data centers in
the United States. These transfers are covered by:

- The EU–US Data Privacy Framework, and
- Standard Contractual Clauses (SCCs) where the framework does not apply.

If you need to keep all processing within the EU, contact us — we can
configure an EU-only LLM provider (e.g. Mistral) on request.

## 7. Your rights under GDPR

If your personal data is being processed via our Service, you have the
right to:

- **Access** the metadata we hold about you (Art. 15).
- **Rectification** of inaccurate metadata (Art. 16).
- **Erasure** of metadata before the 90-day retention window expires
  (Art. 17).
- **Restriction** of processing (Art. 18).
- **Object** to processing on legitimate-interest grounds (Art. 21).
- **Data portability** of your metadata in a machine-readable format
  (Art. 20).
- **Lodge a complaint** with your supervisory authority. For EU users
  that is your national Data Protection Authority; for UK users, the
  ICO.

To exercise any of these, email **support@your-domain.com** with your
RapidAPI user identifier and a description of your request. We respond
within 30 days as required by Art. 12.

## 8. Security measures

- TLS 1.2+ for all transport (RapidAPI gateway → our service → LLM
  provider).
- API keys held in environment variables, never in source code or logs.
- No customer data is ever written to disk.
- The audit log on persistent volume contains metadata only, not
  document content; access is restricted to the operator.
- Source code is private and access-controlled.
- Periodic dependency vulnerability scans via GitHub Actions and
  Dependabot.

## 9. Children's data

The Service is not directed at children. If a resume document submitted
to the Service relates to a minor, you (the API caller) are responsible
for having an appropriate legal basis and parental consent where required.

## 10. Changes

We will post any material changes to this policy at least 30 days
before they take effect, and notify active API users by email when
practicable.

## 11. Contact

Questions, GDPR requests, or DPA requests: **support@your-domain.com**
