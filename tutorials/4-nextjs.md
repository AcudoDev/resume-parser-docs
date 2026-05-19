# Add resume upload to a Next.js application

A candidate-facing resume upload in Next.js 15: file picker,
server-side parse, result display. The one rule: never expose your
RapidAPI key to the browser. Calls go through a Next.js Route Handler
that holds the key as a server-only environment variable.

What you'll end up with is an `/upload` page that accepts a PDF or
DOCX, parses it via the API, and renders the candidate's name,
skills, and confidence scores.

Prereqs: Node 18+, a fresh Next.js 15 project (`npx
create-next-app@latest`), and a RapidAPI key (see the **Quickstart**
tutorial on this listing if you don't have one yet).

## 1. Project setup

If you don't already have a Next.js 15 app:

```bash
npx create-next-app@latest resume-upload-demo \
  --typescript --app --tailwind --eslint --no-src-dir
cd resume-upload-demo
```

Add the RapidAPI key to `.env.local` (this file is gitignored by
default in Next.js, keep it that way):

```env
RAPIDAPI_KEY=your_key_here
RAPIDAPI_HOST=resume-parser20.p.rapidapi.com
```

The name does **not** start with `NEXT_PUBLIC_`, which is what keeps
it server-only. Anything prefixed `NEXT_PUBLIC_` gets bundled into
the client. You don't want to ship your API key to every visitor's
browser.

## 2. The server-side Route Handler

Create `app/api/parse/route.ts`. This is the proxy that receives the
file from the browser and forwards it to RapidAPI with your secret
key attached.

```typescript
// app/api/parse/route.ts
import { NextRequest, NextResponse } from "next/server";

export const runtime = "nodejs";          // need full Node for FormData -> fetch
export const maxDuration = 30;            // Vercel default is 10s, bump to 30 for the parse

export async function POST(request: NextRequest) {
  const incoming = await request.formData();
  const file = incoming.get("file");

  if (!(file instanceof File)) {
    return NextResponse.json(
      { detail: "Missing 'file' field in form data" },
      { status: 400 },
    );
  }

  if (file.size > 10 * 1024 * 1024) {
    return NextResponse.json(
      { detail: "File over 10 MB" },
      { status: 413 },
    );
  }

  // Re-build the form for the upstream call.
  const upstream = new FormData();
  upstream.append("file", file, file.name);

  const resp = await fetch(
    `https://${process.env.RAPIDAPI_HOST}/parse`,
    {
      method: "POST",
      headers: {
        "X-RapidAPI-Key": process.env.RAPIDAPI_KEY!,
        "X-RapidAPI-Host": process.env.RAPIDAPI_HOST!,
      },
      body: upstream,
    },
  );

  const body = await resp.json();
  return NextResponse.json(body, { status: resp.status });
}
```

A few non-obvious things in here:

- `runtime = "nodejs"` is needed because the Edge runtime's `fetch` doesn't accept a Node `FormData` with file parts in all hosting environments. Node runtime is the safe default for file forwarding.
- `maxDuration = 30` because parses take ~15s p50 / ~22s p95. Vercel's default 10s timeout kills the request mid-flight on longer parses.
- We don't stream the body back. The response is small (a few KB of JSON), and you almost certainly want to apply your own business logic before forwarding (filtering, audit logging, etc).
- The `413` pre-check shortcuts before the upstream call. Saves you one billed API request for files that would fail anyway.

## 3. The client-side upload form

Create `app/upload/page.tsx`. This is a client component (note the
`"use client"` at the top) so it can manage upload state.

```tsx
// app/upload/page.tsx
"use client";

import { useState } from "react";

type ParseResult = {
  resume: {
    identity: { full_name: string; title?: string };
    contact: { email?: string };
    skills: { name: string; esco_uri?: string; type: string }[];
    confidence: Record<string, { value: number; reason: string | null }>;
    detected_language: string;
  };
  metadata: { request_id: string; latency_ms: number };
};

export default function UploadPage() {
  const [file, setFile] = useState<File | null>(null);
  const [busy, setBusy] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [result, setResult] = useState<ParseResult | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!file) return;
    setBusy(true);
    setError(null);
    setResult(null);

    const form = new FormData();
    form.append("file", file);

    const resp = await fetch("/api/parse", { method: "POST", body: form });
    const body = await resp.json();
    setBusy(false);

    if (!resp.ok) {
      setError(body.detail ?? "Parse failed");
      return;
    }
    setResult(body);
  }

  return (
    <main className="mx-auto max-w-2xl p-8">
      <h1 className="text-2xl font-bold mb-6">Resume upload</h1>

      <form onSubmit={handleSubmit} className="space-y-4">
        <input
          type="file"
          accept=".pdf,.docx,.png,.jpg,.txt"
          onChange={(e) => setFile(e.target.files?.[0] ?? null)}
          disabled={busy}
        />
        <button
          type="submit"
          disabled={!file || busy}
          className="rounded bg-teal-600 px-4 py-2 text-white disabled:opacity-50"
        >
          {busy ? "Parsing..." : "Parse resume"}
        </button>
      </form>

      {error && <p className="mt-4 text-red-600">Error: {error}</p>}

      {result && (
        <section className="mt-8 space-y-4">
          <h2 className="text-xl font-semibold">{result.resume.identity.full_name}</h2>
          {result.resume.identity.title && <p>{result.resume.identity.title}</p>}
          {result.resume.contact.email && <p>{result.resume.contact.email}</p>}

          <div>
            <h3 className="font-semibold mt-4">Top skills</h3>
            <ul className="list-disc list-inside">
              {result.resume.skills.slice(0, 8).map((s) => (
                <li key={s.name}>
                  {s.name} {s.esco_uri && <span className="text-xs text-slate-500">(ESCO)</span>}
                </li>
              ))}
            </ul>
          </div>

          <div>
            <h3 className="font-semibold mt-4">Confidence per section</h3>
            <ul>
              {Object.entries(result.resume.confidence).map(([section, c]) => (
                <li key={section}>
                  {section}: <strong>{c.value.toFixed(2)}</strong>
                  {c.reason && <span className="text-sm text-slate-600"> ({c.reason})</span>}
                </li>
              ))}
            </ul>
          </div>

          <p className="text-xs text-slate-500">
            request_id: {result.metadata.request_id} ({result.metadata.latency_ms} ms)
          </p>
        </section>
      )}
    </main>
  );
}
```

Run `npm run dev`, open <http://localhost:3000/upload>, pick a PDF
resume, submit. After ~15 seconds you should see the candidate's
name, top skills, and confidence scores rendered.

## 4. Handle the slow-call UX

A 15-second wait without feedback feels broken. Two upgrades worth
doing before shipping:

- **Disable the submit button** while busy (already in the snippet above) and change the label to "Parsing...".
- **Add a progress message** that rotates ("Extracting text...", "Matching skills to ESCO...", "Validating..."). It's UI theater, but the wait feels intentional instead of frozen.

```tsx
const [stage, setStage] = useState("");
useEffect(() => {
  if (!busy) return;
  const stages = ["Extracting text...", "Matching skills to ESCO...", "Validating..."];
  let i = 0;
  const t = setInterval(() => {
    setStage(stages[i % stages.length]);
    i += 1;
  }, 5000);
  return () => clearInterval(t);
}, [busy]);
```

## 5. Production hardening

Before pushing to production, three things to add:

1. **Rate-limit your Route Handler.** Otherwise a single misbehaving client can burn through your monthly quota. Simplest fix: wrap the handler in a per-IP token-bucket (`@upstash/ratelimit` if you're on Vercel + Upstash, `next-rate-limit` for self-hosted).

2. **Cache by file hash.** If a user uploads the same PDF twice (different sessions, same file), the parse is deterministic. Hash the file bytes (SHA-256) on the server, check a Redis or Vercel KV cache, and only call `/parse` on a miss. Saves quota and gives instant responses for duplicates.

3. **Audit-log the `request_id`.** Every parse comes back with a `metadata.request_id` UUID. Log it alongside your own internal ID. If you ever open a support ticket, that's how we look up your specific call.

```typescript
console.log({
  internal_id: candidateId,
  request_id: body.metadata?.request_id,
  status: resp.status,
  latency_ms: body.metadata?.latency_ms,
});
```

## What to do next

Two related tutorials on this listing build on the upload form:

- **Filter resumes for human review with confidence scores**: apply the bucket pattern to your Route Handler so the right parses go to your review queue.
- **Map candidates to your skill graph using ESCO URIs**: store the parsed skills in Postgres or Neo4j so they're queryable across candidates.

For the full reference (limits, error codes, every field), see the
developer guide on this listing's About tab.
