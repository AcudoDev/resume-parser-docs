# Terms of Service — Resume Parser API

_Last updated: 2026-05-19_
_Effective date: upon first use of the Service_

> **Plain-language summary**: You can use our API to parse resumes for
> any lawful business purpose. You agree not to abuse the Service or
> use it to violate someone's privacy. We provide the API "as is" and
> bill via RapidAPI. Either side can stop the relationship anytime.

These Terms of Service ("Terms") govern your access to and use of the
Resume Parser API ("the Service", "we", "us", "our"), accessible
through the RapidAPI marketplace at
`https://rapidapi.com/<your-listing>/api/resume-parser-api`.

By calling the Service or subscribing to a pricing plan, you ("you",
"Customer") accept these Terms in full. If you do not accept them, do
not use the Service.

## 1. Service description

The Service is an HTTP API that accepts a resume document (PDF, DOCX,
image, or plain text) and returns a structured JSON representation of
its content, including identity, contact details, work history,
education, skills (normalized to the European Commission ESCO
taxonomy), languages, and certifications, alongside per-section
confidence scores.

The Service is provided through the RapidAPI marketplace gateway,
which handles authentication, request routing, rate limiting, and
billing on our behalf.

## 2. Account and authentication

You authenticate by sending a valid `X-RapidAPI-Key` header issued by
RapidAPI to your account. You are responsible for keeping that key
confidential. You are liable for all calls made with your key, whether
or not authorized by you. Notify RapidAPI immediately if you suspect a
compromise.

## 3. Pricing and billing

Pricing is set per tier on the RapidAPI listing page. RapidAPI
collects payment and remits the operator's share according to their
own marketplace terms. We do not directly bill you and do not store
your payment information. Refunds, chargebacks, and disputes are
handled per the RapidAPI Terms of Service.

Free-tier limits and per-tier quotas are listed on the RapidAPI page.
Exceeding your quota returns HTTP 429 until the next billing window;
upgrade in the RapidAPI dashboard to continue.

## 4. Acceptable use

You agree NOT to use the Service to:

- Parse resumes of individuals who have not consented to having their
  CV processed for the purpose you are pursuing.
- Build a profiling system that makes consequential automated decisions
  about a person (e.g., automated hiring) without an appropriate
  legal basis and human oversight (cf. GDPR Art. 22, EU AI Act).
- Conduct discrimination based on protected characteristics inferable
  from a resume (gender, age, ethnicity, religion, disability, etc.).
- Reverse-engineer, decompile, or attempt to derive the source code,
  prompts, or model weights underlying the Service.
- Resell or redistribute the Service as a standalone product (you may
  embed it in your own product where the API is one of many
  components).
- Submit content that you do not have the legal right to process, or
  that contains malware, illegal content, or copyrighted material
  beyond what is reasonable for the purpose of resume parsing.
- Knowingly attempt to overwhelm the Service (denial-of-service, abuse
  of free tier, etc.).
- Use the Service in a jurisdiction where doing so is illegal.

We may suspend or terminate your access for violations of this
section, with notice where possible.

## 5. Intellectual property

- **The Service, including the API, prompts, code, models, and ESCO
  index, is and remains our property.** Your subscription grants you
  a limited, non-exclusive, non-transferable license to use the
  Service strictly for the purposes described in these Terms.
- **Your input data (resumes) remains yours.** Our use of it is
  limited to the processing described in §1 of the Privacy Policy and
  in this document.
- **The output (parsed JSON) is yours to use.** You may freely store,
  display, modify, redistribute, or commercialize the output. We
  claim no copyright over the structured representation of facts.
- The ESCO taxonomy is published by the European Commission under the
  EU Open Data licence; their licence terms apply to redistribution
  of URIs and labels.

## 6. Data processing and privacy

Our processing of resume content is governed by the
[Privacy Policy](PRIVACY.md), which is incorporated into these Terms
by reference. Key points:

- Resume content is held in memory for the duration of one API call
  (typically 10-25 seconds, hard cap 90 seconds) then discarded.
- No resume content is ever written to disk, persisted, or used to
  train models.
- We retain anonymous metadata (call ID, timestamp, byte size,
  success status) for 90 days for billing and abuse prevention.
- For EU/UK users, we act as a GDPR data processor; you are the
  controller for the resumes you submit. A Data Processing Agreement
  template is available on request at `support@your-domain.com`.

## 7. Service availability

We target 99.5% monthly uptime but **do not commit to a contractual
SLA at the Free, Pro, Ultra tiers**. For enterprise commitments with
formal SLAs and dedicated support, contact `support@your-domain.com`.

Planned maintenance is announced 24 hours in advance via the
RapidAPI dashboard "API status" field. Emergency maintenance may be
performed without notice when required for security.

The Service depends on third-party providers (RapidAPI, OpenRouter,
OpenAI, Railway, ESCO). Outages of these dependencies may cascade
into Service unavailability; we are not liable for downtime caused
by sub-processor failures beyond our reasonable control.

## 8. Warranties and disclaimers

THE SERVICE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITHOUT WARRANTY
OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING WITHOUT LIMITATION
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, OR
NON-INFRINGEMENT.

Resume parsing is an inherently approximate task: the Service uses
large language models that may produce inaccurate, incomplete, or
inconsistent outputs. **Per-section confidence scores indicate the
parser's own assessment of reliability** — you are responsible for
deciding how to act on outputs with low confidence. The Service is
not a substitute for human review in high-stakes decisions (hiring,
credit, immigration, etc.).

## 9. Limitation of liability

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW:

- Our aggregate liability under these Terms is capped at the **greater
  of (a) the fees you paid us in the 12 months preceding the claim,
  or (b) one hundred (100) euros (€100)**.
- We are not liable for indirect, incidental, consequential,
  exemplary, or punitive damages (lost profits, lost data, business
  interruption, etc.) even if advised of the possibility.
- These limits do not apply to liability that cannot be excluded
  under applicable law (e.g., death/bodily injury caused by gross
  negligence, willful misconduct).

This allocation of risk reflects the price of the Service and is a
fundamental part of the bargain. If you require higher limits,
contact us for an enterprise contract.

## 10. Indemnification

You will defend and indemnify us against any third-party claim
arising from (a) your violation of these Terms, (b) your processing
of resumes without the necessary legal basis, or (c) the way your
product uses the Service's output in downstream decisions.

## 11. Termination

You may stop using the Service at any time by cancelling your
RapidAPI subscription. We may terminate or suspend your access:
- Immediately, for material breach of §4 (Acceptable use).
- With 30 days' notice, for any reason or no reason, refunded
  prorated to the unused portion of the current billing period.

On termination, sections 5, 8, 9, 10, 12 survive.

## 12. Governing law and disputes

These Terms are governed by the **laws of France**, without regard to
conflict-of-laws principles. Disputes are subject to the **exclusive
jurisdiction of the courts of Paris, France**, except that either
party may seek injunctive relief in any competent court to protect
intellectual property or confidentiality.

Consumer users in the EU retain the protections of mandatory consumer
law in their country of residence (e.g., GDPR rights, withdrawal
rights) which override any conflicting provision here.

## 13. Changes to these Terms

We will publish material changes at least 30 days before they take
effect, and notify active subscribers via the RapidAPI dashboard.
Continued use after the effective date constitutes acceptance.

## 14. Contact

For any question about these Terms:
**support@your-domain.com**
