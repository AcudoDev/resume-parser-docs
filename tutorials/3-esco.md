# Map candidates to your skill graph using ESCO URIs

Free-text skills don't join. One resume lists "Python". Another lists
"python". A third "Python (programming)". Your database treats them
as three different things.

ESCO (the European Commission's skill taxonomy) gives every skill a
canonical URI, available in English and French. The Resume Parser API
normalizes every detected skill against ESCO, so you get a stable
join key out of the box.

This page shows two ways to turn that into a working skill graph:
Postgres if your stack is relational, Neo4j if it's graph. Use
whichever fits.

Prereqs: a working `/parse` call (see the **Quickstart** tutorial on
this listing if you don't have one yet), Python 3.9+, and one of:
Postgres 14+ with `psycopg`, or Neo4j 5+ with the official driver.

## What the response actually contains

Every skill in `resume.skills` ships with up to four ESCO fields:

```json
{
  "name": "Python",
  "type": "hard",
  "esco_uri": "http://data.europa.eu/esco/skill/ccd0a1d9-afda-43d9-b901-96344886e14e",
  "esco_label": "Python (computer programming)"
}
```

| Field | What it is |
|---|---|
| `name` | The string as it appeared on the resume |
| `type` | One of `hard`, `soft`, `language`, `tool`, `certification`, `unknown` |
| `esco_uri` | The canonical ESCO URI. Your join key. |
| `esco_label` | Standardized label (English by default, bilingual lookup available) |

For skills with no ESCO match (proprietary tools, slang, very new
tech), `esco_uri` and `esco_label` are `null`. You can still store
them. They just won't join to other candidates' skill graphs.

## Pattern 1: Postgres skill graph

The clean schema is two tables: a `skills` table keyed by ESCO URI
(populated lazily as you encounter new skills), and a
`candidate_skills` join table.

```sql
CREATE TABLE skills (
    esco_uri    TEXT PRIMARY KEY,
    esco_label  TEXT NOT NULL,
    skill_type  TEXT NOT NULL
);

CREATE TABLE candidate_skills (
    candidate_id  UUID NOT NULL,
    esco_uri      TEXT NOT NULL REFERENCES skills(esco_uri),
    raw_name      TEXT NOT NULL,    -- what was on the resume
    confidence    REAL NOT NULL,    -- copied from resume.confidence.skills.value
    PRIMARY KEY (candidate_id, esco_uri)
);
```

The Python that ingests one parse:

```python
import psycopg

def ingest_skills(conn, candidate_id: str, parse_response: dict) -> None:
    resume = parse_response["resume"]
    skill_conf = resume["confidence"]["skills"]["value"]

    with conn.cursor() as cur:
        for skill in resume["skills"]:
            if not skill.get("esco_uri"):
                continue  # skip unmapped skills, or stash them elsewhere

            # Upsert the skill master record (idempotent: same URI, same row)
            cur.execute(
                """
                INSERT INTO skills (esco_uri, esco_label, skill_type)
                VALUES (%s, %s, %s)
                ON CONFLICT (esco_uri) DO NOTHING
                """,
                (skill["esco_uri"], skill["esco_label"], skill["type"]),
            )

            # Insert the candidate -> skill edge
            cur.execute(
                """
                INSERT INTO candidate_skills (candidate_id, esco_uri, raw_name, confidence)
                VALUES (%s, %s, %s, %s)
                ON CONFLICT (candidate_id, esco_uri) DO UPDATE
                  SET raw_name = EXCLUDED.raw_name,
                      confidence = EXCLUDED.confidence
                """,
                (candidate_id, skill["esco_uri"], skill["name"], skill_conf),
            )
    conn.commit()
```

Now you can answer "find all Python engineers in France" with a real
join:

```sql
SELECT DISTINCT c.candidate_id, c.name
FROM candidates c
JOIN candidate_skills cs ON cs.candidate_id = c.candidate_id
WHERE cs.esco_uri = 'http://data.europa.eu/esco/skill/ccd0a1d9-afda-43d9-b901-96344886e14e'
  AND c.country = 'FR'
  AND cs.confidence >= 0.7;
```

No fuzzy matching, no `LOWER(skill) LIKE '%python%'`. The URI is the
source of truth.

## Pattern 2: Neo4j skill graph

If you're building anything skill-graph-shaped (skill similarity,
job-to-candidate recommendations, career-path traversal), a graph DB
is the natural fit. Nodes are candidates and skills. Edges are "has
skill" relationships.

```python
from neo4j import GraphDatabase

def ingest_skills_neo4j(driver, candidate_id: str, parse_response: dict) -> None:
    skills = [s for s in parse_response["resume"]["skills"] if s.get("esco_uri")]

    with driver.session() as session:
        session.run(
            """
            UNWIND $skills AS skill
            MERGE (s:Skill {esco_uri: skill.esco_uri})
              ON CREATE SET s.label = skill.esco_label, s.type = skill.type
            MERGE (c:Candidate {id: $candidate_id})
            MERGE (c)-[r:HAS_SKILL]->(s)
              ON CREATE SET r.raw_name = skill.name
            """,
            candidate_id=candidate_id,
            skills=skills,
        )
```

You can then ask Cypher questions like "who else in our candidate
pool has overlapping skills with candidate X?":

```cypher
MATCH (c1:Candidate {id: $candidate_id})-[:HAS_SKILL]->(s:Skill)<-[:HAS_SKILL]-(c2:Candidate)
WHERE c1 <> c2
WITH c2, count(s) AS shared_skills
WHERE shared_skills >= 5
RETURN c2.id, shared_skills
ORDER BY shared_skills DESC
LIMIT 20
```

## Bilingual labels (FR resumes)

When you parse a French resume, the same `esco_uri` comes back. The
URI is language-agnostic. The `esco_label` will be the French label
("Python (programmation informatique)") when the source resume was
detected as French.

So your skill graph is the same whether you parse English or French
resumes. A "Python" engineer from Paris and a "Python" engineer from
London land on the same node. You only handle language at the
display layer (which label to show the user), not at the data layer.

If you want both labels regardless of source language, the ESCO API
itself (free, public) exposes them via the URI. The Resume Parser
returns whichever matches the resume's language to keep responses
compact.

## Handling unmapped skills

Some skills won't have an ESCO match: proprietary tools, very new
frameworks, made-up titles. Two reasonable strategies:

1. **Store them separately**, keyed by lowercase name. You can batch-promote them to ESCO later if and when ESCO adds them.
2. **Drop them**. If your downstream is skill-graph-based, an unmappable skill is noise.

Look at the proportion in your first 1,000 parses to decide. Across
our sample resumes, around 5 to 8% of skills are unmapped, mostly
proprietary stacks ("our internal X framework") that wouldn't be
useful in a graph anyway.

## What to do next

Two related tutorials on this listing are useful follow-ups:

- **Filter resumes for human review with confidence scores**: the `confidence.skills.value` field tells you which parses to trust before ingesting.
- **Add resume upload to a Next.js application**: wire this into a candidate-facing form.

The full schema for every field on `skills[]` is in the developer
guide on this listing's About tab.
