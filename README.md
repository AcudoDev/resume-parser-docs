# Resume Parser API — Public Documentation

This repository contains the **public-facing legal and developer
documentation** for the Resume Parser API.

The implementation source code lives in a private repository.

## What's here

| File | Purpose |
|---|---|
| [`RAPIDAPI.md`](RAPIDAPI.md) | Developer guide: authentication, endpoints, request/response examples in cURL / Python / JavaScript, error codes, limits. |
| [`PRIVACY.md`](PRIVACY.md) | GDPR-compliant privacy policy. Describes how we process resume content and what metadata we retain. |
| [`TERMS.md`](TERMS.md) | Terms of Service. Acceptable use, IP rights, limitation of liability, governing law. |

## What is the Resume Parser API?

A fast multilingual resume parser (English / French) that turns
PDF / DOCX / image / text resumes into a structured JSON schema
(`ResumeV1`), with skills normalized to the European Commission's
**ESCO taxonomy** and per-section **confidence scores**.

- ~15 s p50 latency, ~22 s p95
- Three endpoints: synchronous, async-with-polling, batch
- ESCO URIs on every detected skill
- Scanned PDFs supported natively (vision pipeline)
- EU-hosted infrastructure (Amsterdam)

## Get the API

Listed on the RapidAPI marketplace:
**https://rapidapi.com/<your-listing>/api/resume-parser-api**

A Free tier is available (50 calls / month, no card required) so you
can evaluate the API before subscribing to a paid plan.

## Contact

`solutions@slifer.fr`
