# Tutorials

Long-form walkthroughs covering the most common ways teams integrate
the Resume Parser API. Each tutorial is self-contained and assumes
only a working RapidAPI key.

These tutorials are mirrored on the RapidAPI Hub listing under the
**Tutorials** tab. The Markdown in this directory is the source of
truth — edit here, then copy-paste into the RapidAPI editor.

## Index

| # | Tutorial | Audience | What you'll learn |
|---|---|---|---|
| 1 | [Get your first parse in 5 minutes](./1-quickstart.md) | Anyone discovering the API | cURL / Python / Node snippets, response schema, common errors |
| 2 | [Filter resumes for human review with confidence scores](./2-confidence.md) | ATS / recruitment platforms | The 3-bucket review pattern, threshold tuning, reviewer UX |
| 3 | [Map candidates to your skill graph using ESCO URIs](./3-esco.md) | HR-tech, talent platforms | Postgres skill graph, Neo4j skill graph, bilingual labels |
| 4 | [Add resume upload to a Next.js application](./4-nextjs.md) | Indie builders, MVP devs | Server-side Route Handler, client form, prod hardening |

## Banner images

Each tutorial has an accompanying banner image (1200×675 PNG)
displayed at the top of the tutorial page on the RapidAPI listing.

The banners live in [`banners/`](./banners/) and are uploaded
directly via the RapidAPI Studio editor (the "image" field accepts
file uploads — no need to host them at a public URL).

The PNGs in this directory are the source of truth for what was
uploaded. If you need to swap a banner, edit the PNG here, push to
this repo, then re-upload through the editor.

## Adding a new tutorial

1. Pick the next number in sequence (e.g. `5-…`).
2. Create `N-slug.md` with the same structure as the existing ones:
   hook → prerequisites → numbered steps → "What to do next" links.
3. Drop the banner PNG into `banners/banner-N-slug.png` (1200×675,
   16:9 — see `scripts/rapidapi/generate_spotlights.py` in the
   private repo for the generator).
4. Add a row to the table above.
5. In the RapidAPI Studio editor (Tutorials tab → New tutorial),
   paste the title, upload the banner PNG from `banners/`, and
   paste the Markdown content. Save as draft, preview, then publish.

## Cross-tutorial links

When linking between tutorials in the Markdown source, use relative
paths (`./2-confidence.md`) so the links resolve correctly on
GitHub. When you paste into the RapidAPI editor, swap those relative
paths for the canonical RapidAPI tutorial URLs (which you'll know
once each tutorial is published — they follow the pattern
`https://rapidapi.com/AcudoDev/api/resume-parser20/tutorials/<slug>`).
