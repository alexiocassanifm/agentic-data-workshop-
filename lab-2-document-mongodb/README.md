# Lab 2 — Document Database on MongoDB

## Objective

In Lab 1 you took the Larkspur Flow corpus and modeled it relationally on
Supabase: separate tables, foreign keys, a schema the database enforces for
you. This lab takes the **same corpus** and asks a different question: what
does this data look like when there is no schema to enforce, and you get to
decide document boundaries instead of table boundaries?

This is not "Lab 1 again with a different tool." Here, the objective is:

- Decide **why** (or whether) a document model actually fits this corpus
  better than a relational one, entity by entity — not everywhere, not
  nowhere.
- Decide how to **structure documents**: what belongs inside a single
  document versus what belongs in its own collection.
- Learn the concrete rule for **embed vs. reference**: bounded, small,
  co-accessed data embeds well; unbounded, independently-growing data does
  not.
- See which queries become **natural** in a document model and which become
  **awkward** compared to the relational version you just built.
- Figure out **where an index is actually needed** once you are not relying
  on foreign keys and joins to keep things fast.

By the end you will have a MongoDB Atlas database populated from the same raw
files as Lab 1, a documented reasoning trail for the modeling choices you
made, and a concrete list of queries that are easier or harder than they were
on Supabase — the raw material the Bonus lab will turn into a scored
comparison.

> **Method note.** Keeping both Lab 1 and Lab 2 in this workshop is only
> justified because each one carries a declared, different design objective
> — Lab 1's is "why relational, and where the constraints matter"; this
> lab's is the list above. Without that distinction, Lab 2 would just be
> Lab 1 repeated on a second tool, which is low-value repetition, not a
> second lesson. With it, Lab 1 and Lab 2 together are exactly the
> comparison the Bonus lab builds on.

## Duration

Approximately 75 minutes:

- ~20 minutes: connection check + document model proposal and discussion
- ~10 minutes: the explicit decision point below
- ~25 minutes: pipeline build and population
- ~20 minutes: verification, natural-vs-awkward queries, index check

## Prerequisites

- **[`../corpus/`](../corpus/)** — the shared corpus and its
  [`README.md`](../corpus/README.md). You will point your agent at the same
  raw files as Lab 1 (CSVs, contract PDFs, support tickets, sales notes,
  usage-log JSON, onboarding emails) — not at Supabase's already-cleaned
  tables. Modeling the messiness yourself, again, is part of the point.
- **Lab 0 completed** for MongoDB specifically: an Atlas cluster exists, your
  connection details are configured, and your agent's MongoDB MCP connection
  is live. If you skipped straight here, ask your agent to run the
  `mcp-health-check` skill first and confirm it can see your MongoDB
  connection and an empty (or previously-populated, if you are re-running
  this lab) database.
- **[`../lab-1-relational-supabase/README.md`](../lab-1-relational-supabase/README.md)
  completed**, with its Supabase project still existing and reachable. This
  lab constantly asks "how did this compare to the relational version?" —
  but it does not depend on your agent remembering Lab 1's conversation.
  Your agent's Supabase MCP connection (already configured in Lab 0) is
  scoped to this project, not to any particular conversation, so whenever a
  comparison is needed it can and should inspect the live Supabase schema
  directly — whether this is a continuation of the same chat or a brand-new
  one.

## Guided steps

1. **Check the connection.** Ask your agent to run the `mcp-health-check`
   skill and confirm it can reach your MongoDB Atlas cluster and see the
   current (likely empty) list of collections.

   **Prompt:**

   ```
   Run the mcp-health-check skill and confirm you can reach my MongoDB
   Atlas cluster. Tell me what collections currently exist, if any.
   ```

2. **Ask your agent to re-read the corpus with a document lens.** Point it at
   `corpus/README.md` and the raw files, and ask it explicitly not to reuse
   the relational schema from Lab 1 as a template — the goal is to propose a
   document-shaped model from scratch, entity by entity, for the same
   underlying data.

   **Prompt:**

   ```
   Before proposing anything, re-read corpus/README.md and the raw
   corpus files - not the tables we built in Supabase. Don't use the
   Lab 1 relational schema as your starting point; think about document
   boundaries from scratch.
   ```

3. **Ask your agent to run the `schema-proposal` skill for a document
   database.** Make sure the request is explicit that the output should be
   document-shaped: candidate collections, which fields nest inside a parent
   document versus which live in their own collection referenced by
   `customer_id`, at least one rejected alternative with the reasoning
   spelled out, and open questions for you to decide — not just a straight
   translation of five or six SQL tables into five or six collections.

   **Prompt:**

   ```
   Run the schema-proposal skill for a MongoDB document model of this
   corpus. I want candidate collections, an explicit embed-vs-reference
   call for each entity with your reasoning, at least one alternative
   you considered and rejected, and a list of open questions for me to
   decide - not a one-to-one translation of the Supabase tables into
   collections.
   ```

4. **Ask your agent to hold its proposal up against Lab 1's schema, table by
   table.** First, have it inspect the live Supabase project directly
   through its MCP connection — listing tables, columns, constraints, and
   foreign keys — rather than relying on anything it might recall from
   earlier in the conversation. Then, for each table it finds, have it
   state plainly: does this become an embedded field, a field inside a
   bigger document, or does it stay its own collection? And why, in one
   sentence per entity.

   **Prompt:**

   ```
   First, inspect the live Supabase project via your MCP connection - list
   its tables, columns, constraints, and foreign keys. Don't rely on
   anything you might recall from earlier in this conversation. Then go
   through every table you found one by one. For each, tell me plainly:
   does this become an embedded field, a field inside a bigger document,
   or does it stay its own collection here - and why, in one sentence per
   entity.
   ```

5. **Ask your agent to specifically address the usage-log files.** Each
   `usage-<customer_id>.json` is already nested (a `daily_usage` array, each
   day nesting an array of per-feature records). Ask it to lay out, side by
   side, what keeping that nesting as-is inside a document looks like versus
   flattening it into one document per day (or per day-per-feature) in its
   own collection — with the tradeoffs of each, not just a recommendation.

   **Prompt:**

   ```
   Look specifically at the usage-log files (usage-<customer_id>.json).
   Lay out, side by side, what keeping the existing nested daily_usage
   structure looks like inside one document per customer, versus
   flattening it into one document per day (or per day-per-feature) in
   its own collection. Give me the tradeoffs of each - not just a
   recommendation.
   ```

6. **Make the decision point below before moving on.**

7. **Ask your agent to build the population pipeline** once you've decided
   the document boundaries: read the raw corpus files, apply the same kind
   of cleanup `customers.csv` needs (mixed date formats, blank
   `industry`/`company_size_band`/`signup_date` values, inconsistent country
   casing, the two mojibake company names), and load everything into the
   collections you agreed on. Ask it to store dates as proper date values,
   not as plain strings — this matters more here than it did in Lab 1,
   since there is no column type to force it.

   **Prompt:**

   ```
   Build the population pipeline for the document model we just
   decided on. Read the raw corpus files, apply the same cleanup
   customers.csv needs (mixed date formats, blank industry /
   company_size_band / signup_date values, inconsistent country casing,
   and the two mojibake company names), and load everything into the
   collections we agreed on. Store dates as proper date values, not as
   plain strings.
   ```

8. **Ask your agent to create the indexes the chosen model actually needs**,
   and to explain, for each one, what query it makes possible or fast that
   would otherwise require scanning a whole collection.

   **Prompt:**

   ```
   Create the indexes this model actually needs. For each one, explain
   what query it makes possible or fast that would otherwise require
   scanning a whole collection.
   ```

9. **Ask your agent to run the `pipeline-verify` skill** to compare corpus
   counts and samples against what actually landed in MongoDB, and to
   spot-check a handful of documents field by field.

   **Prompt:**

   ```
   Run the pipeline-verify skill to compare the corpus counts and
   samples against what actually landed in MongoDB, and spot-check a
   handful of documents field by field.
   ```

10. **Ask your agent to write and run 4–5 queries that test the
    natural-vs-awkward comparison** directly: for example, "give me
    everything about one customer in a single read" (a strength of the
    document model) and "count open tickets grouped by industry" (which
    needs an aggregation across collections, closer to the join you wrote in
    Lab 1). Ask it to note, for each, whether it was simpler or more
    convoluted to express than the equivalent in Supabase.

    **Prompt:**

    ```
    Write and run 4-5 queries that test the natural-vs-awkward
    comparison directly - for example, give me everything about one
    customer in a single read, and count open tickets grouped by
    industry. For each one, tell me whether it was simpler or more
    convoluted to express than the equivalent query in Supabase. If
    you're not certain what the equivalent Supabase query would look
    like, check the live Supabase schema via your MCP connection first,
    or reason about the equivalent query structurally from that schema -
    don't guess based on a conversation that may not exist.
    ```

## Explicit decision point

Your agent will propose a document model. Before it builds anything, you
need to decide two things yourself — don't just approve the first draft.

**1. Where does the document boundary sit?**

- **Option A — one rich customer document.** A `customers` collection where
  each document embeds its 1–2 contacts and its subscription/contract
  details (all bounded, small, and almost always read together with the
  customer), while support tickets, sales notes, onboarding emails, and
  usage logs live in their own collections, each carrying a `customer_id`
  field back to the parent.
- **Option B — collection-per-entity.** A close mirror of Lab 1's table set
  (`customers`, `contacts`, `subscriptions`, `tickets`, `sales_notes`,
  `emails`, `usage_logs`), each its own collection, referencing
  `customer_id` everywhere, with almost nothing embedded.

There is a real tradeoff, not a "correct" answer: Option A gives you fast,
single-read access to a whole customer profile at the cost of documents that
grow over time and some duplicated data if you ever need to look a contact
up independently. Option B keeps every collection lean and independently
scalable but pushes you back toward doing the same kind of application-side
joins MongoDB has no built-in enforcement for. Pick one, and be able to say
why — in terms of this corpus, not database theory in the abstract.

**2. What happens to the nested usage logs?**

Given the 14-features-per-day, day-per-customer shape of the usage JSON,
decide: keep each customer's usage log as one nested document (fast to read
whole, but a document that keeps growing and is awkward to query "which
customers used feature X yesterday" against), or flatten it into one document
per day (or per day-per-feature) in its own collection (larger and more
uniform, better suited to aggregation queries, but no longer "one file in,
one document out"). Whichever you pick, be ready to justify it against a
concrete query you actually plan to run.

Write your decision and your one-paragraph reasoning down somewhere
retrievable (a scratch note is fine) — the Bonus lab will ask you to cite it.

## Verification

**In the MongoDB Atlas UI:**

- Open the Collections tab for your database and confirm the collection
  list matches the model you decided on — no stray or duplicate collections
  from a re-run.
- Check the document count per collection against the corpus: 180 customer
  documents (or 180 embedded-contact-bearing customer documents if you chose
  Option A), 500 tickets, 150 sales notes, 124 emails, and 177 usage-log
  documents — not 180. The gap is the handful of very recently signed-up
  customers who don't have usage history yet; if your count is 177, that's
  correct, not a bug. If you chose Option B, also confirm 350 contact
  documents and 200 subscription documents in their own collections.
- Open one customer document directly in the UI and read it. If you embedded
  contacts or subscription data, confirm it's actually nested inside, not a
  reference you forgot to resolve.
- Open one usage-log document and check whether the nested `daily_usage`
  structure is intact (Option "keep nested") or absent because it was
  flattened into its own collection (Option "flatten") — whichever you
  decided.
- Open the Indexes tab for each referenced collection (tickets, sales notes,
  emails, and usage logs if kept separate) and confirm a `customer_id` index
  exists on each.

**Via your agent:**

- Ask it to count documents per collection and compare against the numbers
  above.
- Ask it to fetch one customer by `customer_id` together with everything
  related to them (contacts, tickets, notes, emails, usage) and confirm the
  result is complete and correctly joined by `customer_id`, regardless of
  whether that data is embedded or referenced.
- Ask it to explain (using the query planner) whether the "everything for
  this customer" query is using the `customer_id` index or scanning full
  collections. If it's scanning, that's the index you're missing.
- Ask it to run the aggregation query from step 10 above (tickets by
  industry) and sanity-check the totals against a quick count from the raw
  corpus files.

## Known pitfalls

- **Dates stored as strings.** MongoDB won't stop you from saving
  `signup_date` as the literal text that was in `customers.csv`. Left
  unconverted, you lose range queries, correct sorting, and any date
  arithmetic later. Ask your agent to confirm it converted every date field
  to an actual date type during the pipeline, not just during display.
- **Unbounded embedding.** Embedding tickets, sales notes, emails, or full
  usage history directly inside a customer document looks convenient at
  small scale, but these collections are meant to keep growing per customer
  in a real system. Beyond the practical problems (documents that get slower
  to read and rewrite as they grow, and MongoDB's hard 16 MB document size
  limit), it also means adding one new ticket requires rewriting the whole
  customer document. This is the concrete reason those four stay their own
  collections in a sound design.
- **Missing the `customer_id` index.** Skip it, and the exact query the
  document model is supposed to make easy — "give me everything about this
  customer" — falls back to scanning full collections. Check this with the
  query planner, not just by assuming the agent added it.
- **Duplication drift.** If a contact or subscription value ends up both
  embedded in a customer document and separately stored in its own
  collection, the two will eventually disagree after an update touches only
  one of them. Decide which copy is the source of truth, or avoid the
  duplication altogether.
- **Inconsistent field shapes within a collection.** Nothing enforces that
  every document in a collection has the same fields or types. If your agent
  is inconsistent — say, `mrr_usd` stored as a number in some subscription
  documents and as a string in others — aggregations will silently
  under-count instead of erroring. Spot-check field types, not just field
  names, during verification.
- **The 177-vs-180 usage-log gap.** A handful of very recently signed-up
  customers have no usage-log file at all. Any query or aggregation that
  assumes every customer has one (an inner-style lookup, or an average that
  doesn't account for missing documents) will quietly drop or skew results.
  Ask your agent to handle the missing case explicitly rather than assume
  full coverage.
- **Re-running the pipeline isn't automatically safe.** Unlike a SQL primary
  key, nothing stops MongoDB from inserting the same ticket or customer
  twice if the pipeline runs a second time. Ask your agent to make the load
  idempotent (upsert on the natural ID, or a unique index on it) before you
  re-run anything.

## Multi-tool notes

### Using Codex CLI or opencode instead

Both Codex CLI and opencode now support the same open Agent Skills format
(SKILL.md files) Claude Code uses. opencode scans `.claude/skills/`
directly, so it already sees this repo's `mcp-health-check`,
`schema-proposal`, and `pipeline-verify` skills: invoke them by name exactly
as in the steps above. Codex CLI supports skills too, but scans
`.agents/skills/` instead, a path this repo doesn't use — so on Codex CLI
specifically, every outcome in this lab is reachable by asking your agent
directly to do the same work:

- Instead of the `mcp-health-check` skill, ask your agent to check its
  MongoDB connection and list existing collections and document counts in
  plain language.
- Instead of the `schema-proposal` skill, ask your agent to produce, in
  writing, a document model proposal that includes: candidate collections,
  an explicit embed-vs-reference call for each entity with reasoning, at
  least one alternative it considered and rejected, and a list of open
  questions for you to decide before it builds anything.
- Instead of the `pipeline-verify` skill, ask your agent, after the pipeline
  runs, to compare corpus counts and samples against what actually landed in
  MongoDB, spot-check a handful of documents field by field, and report
  pass/fail with concrete numbers, exactly as described in the Verification
  section above.
- If either tool's MongoDB connection is set up differently (their own MCP
  configuration, or a direct driver connection the agent manages on your
  behalf), the outcome is the same — you are still never opening a code
  editor or writing a connection string by hand; you are asking the agent to
  connect and confirming it can see your database.
