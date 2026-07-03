# Agentic Coding for Data Specialists

A full-day, hands-on workshop for data scientists and data engineers who are
not developers. The point is not to learn a database. The point is to learn
to **direct an AI coding agent**, entirely in natural language, to access and
shape data in backends you don't already know how to operate.

Across the day, one shared fictional corpus — the customer data of
**Larkspur Software, Inc.** and its product **Larkspur Flow** (see
[`corpus/README.md`](corpus/README.md)) — is reused from start to finish.
What changes is not the data; it's the backend paradigm you point the agent
at:

- a rigid **relational** database (Supabase / Postgres)
- a schemaless **document** database (MongoDB)
- **semantic / vector search**, tried two ways on the same corpus (Qdrant
  native, and pgvector inside the same Postgres instance)

The path through the day is: shared corpus → agent-led extraction and schema
design → load into Supabase → load into MongoDB → semantic search with
Qdrant and pgvector → a dashboard over what you've built → a reasoned,
evidence-based comparison of the three approaches.

You will never open a code editor and never write or debug code by hand.
Every step is a prompt to an agent, reviewed and refined in plain language.
The transferable skill this workshop builds is not "Postgres" or "MongoDB"
or "Qdrant" — it's *choosing and designing* a data model for a given
problem, and directing an agent to build it well. That skill travels to
whatever backend you meet next; today's specific tools are just the
practice field.

## Table of contents

A facilitator runs this live with a group, start to finish, in one day. It
is also designed to be repeatable on your own afterward, at your own pace,
using this same README as your guide.

| # | Folder | What happens there | Suggested duration |
|---|---|---|---|
| — | [`corpus/`](corpus/README.md) | The shared data corpus used by every lab below | reference material |
| 0 | [`lab-0-setup/`](lab-0-setup/README.md) | Create the four backend accounts, connect your agent to each, and verify what it can see | ~75–90 min |
| 1 | [`lab-1-relational-supabase/`](lab-1-relational-supabase/README.md) | Agent-led schema design and data load into Supabase/Postgres — the rigid, relational paradigm | ~90 min |
| 2 | [`lab-2-document-mongodb/`](lab-2-document-mongodb/README.md) | The same corpus, remodeled and loaded into MongoDB — the schemaless, document paradigm | ~75 min |
| 3 | [`lab-3-vector-search/`](lab-3-vector-search/README.md) | Semantic search over the unstructured parts of the corpus, tried in Qdrant and in pgvector on the same Postgres database | ~90 min |
| 4 | [`lab-4-dashboard/`](lab-4-dashboard/README.md) | An agent-built dashboard drawing on the backends you've populated | ~60 min |
| Bonus | [`bonus-compare-backends/`](bonus-compare-backends/README.md) | A reasoned comparison artifact scoring the backends against criteria you define yourself | ~45–60 min |

This is a full day of workshop time — roughly six to seven hours of
hands-on work across the sessions above, plus breaks and lunch. Facilitators
should plan pacing accordingly; the per-lab durations are a guide, not a
strict clock.

## Prerequisites

**Four free-tier accounts**, all created and connected during Lab 0:

- **Supabase** — used in Lab 1 and again in Lab 3 (pgvector)
- **MongoDB Atlas** — used in Lab 2
- **Qdrant Cloud** — used in Lab 3
- **Voyage AI** — used to generate embeddings for Lab 3

No account needs to be created before the workshop starts; Lab 0 walks
through all four.

**An AI coding agent.** Claude Code is the primary supported tool for this
material, and every lab is written with it in mind, including five
supporting skills available automatically to anyone running Claude Code
from the repo root (see [`.claude/skills/`](.claude/skills)):

- **mcp-health-check** — checks every connected backend and reports, in
  plain language, what the agent can currently see
- **schema-proposal** — structures how the agent proposes a data model:
  candidate entities and fields, a relationships or embed-vs-reference
  recommendation, a rejected alternative, and open questions for you to
  decide
- **pipeline-verify** — after a load runs, compares the corpus against what
  actually landed in the destination store and reports pass/fail with
  concrete numbers
- **backend-compare** — builds the Bonus comparison artifact from your own
  criteria plus concrete observations from earlier labs
- **dashboard-build** — structures how the agent turns the student's chosen
  business questions into a verified, correctly-labeled Lab 4 dashboard

OpenAI Codex CLI and opencode are also supported. Both now support the same
open Agent Skills format (SKILL.md files) Claude Code uses, so this is no
longer a Claude-Code-only mechanism — but the two look for skills in
different places. opencode scans `.claude/skills/` directly, so it already
picks up every skill above with no extra setup. Codex CLI scans
`.agents/skills/` instead, a path this repo doesn't ship skills under, so on
Codex CLI each lab README spells out the equivalent as a set of steps to ask
your agent to perform directly, reaching the same outcome by direct
prompting instead.

No prior database experience, and no ability to read or write code, is
assumed anywhere in this material.

## How to use this repo

**In a facilitated session:** follow the labs in order, at the pace the
facilitator sets. Each lab builds on state left behind by the one before
it (the Supabase project from Lab 1, the MongoDB cluster from Lab 2, and so
on), so it's worth staying roughly in step with the group, especially
across the Lab 0 → Lab 1 → Lab 2 handoffs.

**On your own, at home:** every lab is self-contained and written to be
followed without a facilitator present. Start at [`lab-0-setup/`](lab-0-setup/README.md),
work through the labs in order, and use the suggested durations above to
plan your own sessions — the whole path does not need to be done in one
sitting.

## Possible future additions

The following are titles only, listed for future development — no content
exists yet for any of them.

- **Guided demo: PostgreSQL to MongoDB migration** — a walkthrough of
  moving data that already lives in a relational schema into a document
  model, using the Lab 1 and Lab 2 backends as source and destination.

Optional extensions:

- **Conversational analytics (natural-language-to-SQL)** on the Lab 1
  database — asking business questions in plain English and having the
  agent translate them into queries against Supabase.
- **Agentic data quality and cleaning** — directing an agent to find and
  resolve the messiness deliberately built into the corpus.
- **Ingestion from a public API with scheduling** — extending the corpus
  with live data pulled on a recurring schedule.
- **Constrained synthetic data generation** — generating additional
  corpus-like records that respect the existing ID scheme and relationships.
- **Automatic ML baseline with report** — a first-pass model and written
  report produced directly from one of the populated backends.
