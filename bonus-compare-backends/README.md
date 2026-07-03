# Bonus Lab: Supabase vs. MongoDB — A Reasoned Comparison

## Objective

Build a single self-contained HTML artifact that puts Supabase (Lab 1) and
MongoDB (Lab 2) side by side for Larkspur Software's actual use case on
this corpus, and end with an argued choice — not a pro/con list pulled
from the general internet, but a decision grounded in what you personally
saw happen across both labs: what the schema looked like in each case,
which queries came naturally, where the agent struggled, and how each
backend handled this corpus's messy `customers.csv` and nested
`usage-logs`.

This is the lab that closes the workshop's data-modeling thread. It is
the twin of the pgvector-vs-Qdrant comparison built inside Lab 3:
together, the two comparisons teach the same decision method on two
different axes — here, data modeling; there, where to put the vectors.

> **Hard rule, stated once and enforced throughout:** you must write down
> your evaluation criteria and their weights *before* the agent shows
> you, or you look at, any scored comparison. Criteria fixed beforehand
> are an evaluation; criteria fixed afterward are a rationalization of a
> choice you already made. See "Explicit decision point" below — the
> agent, through the `backend-compare` skill, is built to enforce this
> and should refuse to proceed to scoring without it.

## Duration

45–60 minutes. This is the synthesis lab that closes the workshop.

## Prerequisites

- **Lab 1 completed** — a relational schema populated on Supabase, with a
  passing `pipeline-verify` check. See the Lab 1 README (Supabase),
  `../lab-1-relational-supabase/README.md`.
- **Lab 2 completed** — collections modeled and populated on MongoDB,
  with a passing `pipeline-verify` check. See the Lab 2 README (MongoDB),
  `../lab-2-document-mongodb/README.md`.
- The shared corpus: [`../corpus/README.md`](../corpus/README.md). You
  don't need to re-read it in full, but you'll want it on hand — the
  criteria you set below should be grounded in what this corpus actually
  contains (the messy `customers.csv`, the nested `usage-logs`, the
  `customer_id` thread running through everything).
- Both the Supabase and MongoDB MCP connections still configured and
  reachable. If it's been a while since Lab 1 or Lab 2, re-run the
  `mcp-health-check` skill (or ask your agent directly) before starting.
- Something to write on — physical notepad or a scratch note your agent
  keeps for you — because in this lab **you** produce the criteria and
  weights, not the agent.

## Guided steps

1. Ask your agent to check that both backends are live and hold what you
   left them with: run the `mcp-health-check` skill, or ask directly for
   a list of Supabase tables (with row counts) and MongoDB collections
   (with document counts). Confirm this matches what you expect from Lab
   1 and Lab 2 before doing anything else.

   **Prompt:**

   ```
   Check that my Supabase and MongoDB connections are still live. Run
   the mcp-health-check skill, then list every Supabase table with its
   row count and every MongoDB collection with its document count.
   ```

2. Before discussing how Lab 1 or Lab 2 actually went, ask your agent to
   help you **brainstorm — not score — a longlist of dimensions** that
   matter for Larkspur Flow's real use case: things like looking up a
   customer's full history across support, sales, and billing; keeping
   billing and contracts consistent; handling the messy signup dates and
   inconsistent countries in `customers.csv`; modeling the nested,
   per-day, per-feature `usage-logs`; how much a query for a common
   business question (e.g., "active Enterprise customers in the EU with
   an open SSO-related ticket") costs to write in each backend; and how
   much friction the agent itself hit while building each pipeline.
   Treat this as a menu to choose from, not a verdict on either backend.

   **Prompt:**

   ```
   Help me brainstorm a longlist of dimensions for comparing Supabase
   and MongoDB on Larkspur Flow's use case - things like looking up a
   customer's full history across support, sales, and billing; keeping
   billing and contracts consistent; handling the messy signup dates
   and inconsistent countries in customers.csv; modeling the nested,
   per-day, per-feature usage-logs; how expensive a common business
   question is to query in each backend; and how much friction you
   personally hit while building each pipeline. Just brainstorm - don't
   score or rank anything yet.
   ```

3. **Stop here and complete the decision point below before continuing.**
   You must pick your final criteria, assign weights, and write them down
   yourself before the agent touches any Lab 1/Lab 2 evidence.

4. Only once your criteria and weights are written down, ask your agent
   to put together a **Lab 1 evidence brief**: the schema as it actually
   exists on Supabase right now (tables, keys, relationships), the
   rejected alternative and reasoning recorded by the `schema-proposal`
   skill during Lab 1, one or two queries that were genuinely natural to
   write, one that was awkward or required a workaround, and any
   friction or `pipeline-verify` results you saw at the time.

   **Prompt:**

   ```
   Put together a Lab 1 evidence brief for Supabase: the schema as it
   actually exists right now (tables, keys, relationships), the
   rejected alternative and reasoning the schema-proposal skill
   recorded during Lab 1, one or two queries that were genuinely
   natural to write, one that was awkward, and any pipeline-verify
   results from that lab.
   ```

5. Ask your agent to put together the matching **Lab 2 evidence brief**
   for MongoDB: the collections and document shapes as they exist right
   now, the embed-vs-reference calls that were actually made (and why),
   including the rejected alternative from Lab 2's `schema-proposal`
   output, natural vs. awkward queries, and `pipeline-verify` results.

   **Prompt:**

   ```
   Put together the matching Lab 2 evidence brief for MongoDB: the
   collections and document shapes as they exist right now, the
   embed-vs-reference calls that were actually made and why (including
   the rejected alternative from Lab 2's schema-proposal output),
   natural vs. awkward queries, and pipeline-verify results.
   ```

6. Ask your agent to invoke the `backend-compare` skill, passing it your
   written criteria and weights plus both evidence briefs, and have it
   build one self-contained HTML artifact that scores each backend
   against each criterion, cites the specific Lab 1/Lab 2 observation
   behind every single score, and ends with an explicit recommendation.
   (On Codex CLI or opencode, follow the equivalent steps in "Multi-tool
   notes" below.)

   **Prompt:**

   ```
   Using the criteria and weights note we saved earlier, invoke the
   backend-compare skill together with both evidence briefs. Build one
   self-contained HTML artifact that scores each backend against each
   criterion, cites the specific Lab 1 or Lab 2 observation behind
   every score, and ends with an explicit recommendation.
   ```

7. Ask your agent to save the artifact — for example as
   `comparison.html` — inside this lab's folder, and to open it so you
   can read it end to end.

   **Prompt:**

   ```
   Save the comparison artifact as comparison.html inside this lab's
   folder, and open it so I can read it.
   ```

8. Review it critically before calling it done. For every scored
   criterion, check that the citation points to something specific from
   Lab 1 or Lab 2 (a table name, an actual query, a concrete error, a
   `pipeline-verify` number) — not a generic statement that could have
   come from a blog post. Send anything vague back to the agent and ask
   it to replace the citation with a specific observation, or to admit it
   doesn't have one.

   **Prompt (if a citation looks vague):**

   ```
   The citation for <criterion> isn't specific enough - replace it
   with an exact table name, query, or pipeline-verify number from Lab
   1 or Lab 2, or tell me you don't have one.
   ```

## Explicit decision point: fix your criteria and weights before any scoring happens

By this point you've already run Lab 1 and Lab 2 — you can't un-see the
two schemas. That's fine: the discipline this lab teaches isn't "look at
this with no prior knowledge," it's **"commit to what matters, and how
much, before you let anything be scored against it."** If you fix
criteria after seeing which backend scores better, you're rationalizing
a decision you've already made. If you fix them first, you're actually
evaluating.

Concretely:

- From the longlist in step 2, **choose 4 to 6 final criteria yourself**.
  Add ones the agent didn't suggest if you think they matter more for
  this use case (for example: how well each handled the two mojibake
  company names, or how easy each schema would be for a support agent to
  query without help). Drop ones you don't care about, even if the agent
  proposed them.
- **Assign each criterion a numeric weight, summing to a fixed total**
  (100 works well). Vague labels like "High / Medium / Low" don't
  produce a single comparable score — insist on numbers.
- Ask your agent to write your criteria and weights into a short note
  (plain text or Markdown) **before it is allowed to look at or discuss
  either evidence brief in a scored way**.
- Give your agent this instruction explicitly: *"Do not produce or show
  me any score, ranking, or comparison until I've given you my written
  criteria and weights. If I ask you to compare before that, refuse and
  ask me to finish this step first."* This is exactly what the
  `backend-compare` skill enforces automatically — but it's worth saying
  out loud once yourself, so you feel the constraint rather than just
  comply with it.
- The requirement is that *you* decide the criteria and weights, not that
  you approve a list the agent hands you. If the agent proposes weights,
  change at least one before accepting it.

## Verification

- Open `comparison.html` in a browser and confirm every scored row cites
  a **specific** Lab 1 or Lab 2 observation — a schema decision, an
  actual query, a concrete friction point, a `pipeline-verify` number —
  never a generic claim you could find written about these two products
  in general.
- Ask your agent to report two counts: the number of criteria you set,
  and the number of citations in the artifact. Every criterion scored
  against every backend needs its own citation — the counts should
  reconcile one-to-one, with no cell left uncited.
- Ask your agent to flag if the same single observation was reused as
  the citation for more than one or two rows — that usually means the
  evidence brief for one backend was too shallow, not that the backends
  are actually tied everywhere.
- As a light sanity check, ask your agent to re-run `mcp-health-check`
  (or list tables/collections directly) on both Supabase and MongoDB one
  more time, to confirm the data being cited hasn't drifted since Lab 1
  and Lab 2. Optionally, glance at Supabase Studio and the MongoDB Atlas
  UI yourself to confirm the tables and collections being cited still
  look the way the artifact describes them.
- Read the final recommendation paragraph on its own. Does it follow from
  the scored table above it, or does it smuggle in a new argument that
  never appeared as a criterion? If the latter, either add that as a
  criterion and rescore, or have the agent remove it from the conclusion.

## Known pitfalls

1. **Fixing criteria after peeking.** The single biggest risk in this
   lab. If you notice yourself wanting to see "how MongoDB did" before
   you've finished writing your weights, stop and finish the decision
   point first.
2. **Vague citations dressed as evidence** — "MongoDB was more
   flexible" without pointing to the actual document shape, query, or
   number behind that claim.
3. **Reusing a generic Postgres-vs-MongoDB take from the internet**
   instead of what this corpus and this agent actually produced in Lab 1
   and Lab 2. The whole point of the exercise is that the right answer is
   corpus- and use-case-dependent, not universal.
4. **Weights that don't add up to anything comparable** — three
   criteria rated "High" and two rated "Medium" gives no way to compute
   a single score. Insist on numbers that sum to a fixed total.
5. **One criterion silently dominating** because its weight was chosen
   after glancing at which backend would win on it.
6. **Treating the messy `customers.csv` or the nested `usage-logs` as
   solved problems** rather than as genuine comparison points. Both labs
   had to make a real modeling decision about them — that decision is
   exactly the kind of evidence this lab should cite.
7. **Stale evidence.** If Lab 1 or Lab 2 happened in an earlier session,
   the agent's recollection of what it built may be approximate. Always
   have it re-inspect the live tables and collections rather than recall
   from an earlier conversation.
8. **Uneven evidence depth.** If one backend gets a detailed evidence
   brief and the other gets a shallow one, every score is biased before
   scoring even starts. Insist on the same depth of inspection for both.

## Multi-tool notes

### Using Codex CLI or opencode instead

Both Codex CLI and opencode now support the same open Agent Skills format
(SKILL.md files) Claude Code uses. opencode scans `.claude/skills/`
directly, so it already sees this repo's `backend-compare` skill: invoke it
by name exactly as in the steps above ("Run the backend-compare skill...").
Codex CLI supports skills too, but scans `.agents/skills/` instead, a path
this repo doesn't use — so on Codex CLI specifically, there's no
`backend-compare` command to invoke directly. Reach the same outcome by
asking your agent to do the following steps, in order:

1. Confirm live connections to Supabase and MongoDB by listing tables
   (with row counts) and collections (with document counts).
2. Brainstorm a longlist of use-case-specific dimensions, without scoring
   anything yet.
3. Stop and have you personally pick and weight the final criteria; ask
   the agent to write them into a plain-text or Markdown criteria note.
   Instruct it not to proceed to any scoring until that note exists.
4. Gather a Lab 1 evidence brief from the live Supabase schema and data.
5. Gather a Lab 2 evidence brief from the live MongoDB collections and
   data.
6. Give this explicit instruction: *"Score each backend against my
   criteria file. Every score must cite one of the observations in the
   evidence briefs. If you can't cite one, don't score that cell — ask
   me instead."*
7. Have it assemble the criteria, both evidence briefs, the scored table,
   and a reasoned recommendation into one self-contained HTML file,
   saved inside this lab's folder.
8. Open the file yourself and verify it against the "Verification"
   section above — the citation coverage check applies exactly the same
   way regardless of which agent built the file.
