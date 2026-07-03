# Lab 1 — Relational Modeling on Supabase

## Objective

Turn part of the shared corpus into a working relational database on
Supabase — entirely by directing your coding agent in natural language,
never by writing or debugging code yourself. Supabase happens to be a
rigid, schema-first relational store (Postgres underneath); the point of
this lab is not to learn Postgres, it's to practice directing an agent to
model, build, and populate a backend you don't already know how to
administer.

The real work here is not accepting whatever schema the agent proposes
first. It's reading that proposal critically and deciding: which entities
and relationships actually exist in this data, where a foreign key or a
`NOT NULL` constraint earns its keep, and where the messiness in the
source files needs a modeling decision from you rather than a silent fix
from the agent.

By the end of this lab you will have a relational schema you can defend,
live tables in Supabase built from that schema, a pipeline that populated
those tables from the raw corpus files, and a way to check that what
landed in the database actually matches what was in the source.

## Duration

Approximately 90 minutes. Designed to run live with a facilitator in a
group setting, but every step below is self-contained enough to repeat
solo afterward using this same README.

## Prerequisites

- Lab 0 (Setup) completed: a Supabase project exists, its MCP connection
  is configured in your agent, and your agent has already confirmed it
  can see the (empty) project. See
  [`../lab-0-setup/README.md`](../lab-0-setup/README.md).
- The shared corpus, described in full in
  [`../corpus/README.md`](../corpus/README.md). This lab uses:
  - `corpus/customers-export/` — `customers.csv` (180 rows),
    `contacts.csv` (350 rows), `subscriptions.csv` (200 rows). Read the
    corpus README's "What's messy, and what's clean" section before you
    start — the messiness is confined entirely to `customers.csv` and is
    deliberate.
  - `corpus/contracts/` — 50 PDF contracts, one per eligible customer,
    named `CTR-####_<company-slug>.pdf`.
  - Optional, if you want to push the discussion further: the free-text
    sources (`support-tickets/`, `sales-notes/`, `emails/`) can be
    mentioned as things a relational schema would model awkwardly, as a
    preview of the paradigm contrast coming in later labs. Not required
    for this lab's pipeline.
- No prior Supabase or SQL knowledge is required. Everything below is
  done by describing what you want to your agent, not by writing queries
  or code by hand.

## Guided steps

Work through these by asking your agent to perform them. You should not
need to open a code editor, write SQL, or touch the Supabase UI except to
verify results at the end.

1. **Ground the agent in the source data.** Ask your agent to read the
   corpus README, then inspect `customers.csv`, `contacts.csv`,
   `subscriptions.csv`, and a sample of a few files from `contracts/`, and
   summarize back to you: what fields exist in each file, which fields
   look messy or inconsistent, and how the files relate to each other
   through `customer_id`.

   **Prompt:**

   ```
   Read corpus/README.md, then look at
   corpus/customers-export/customers.csv,
   corpus/customers-export/contacts.csv,
   corpus/customers-export/subscriptions.csv, and a sample of a few
   files from corpus/contracts/. Summarize back to me: what fields
   exist in each file, which fields look messy or inconsistent, and how
   the files relate to each other through customer_id.
   ```

2. **Ask for a relational schema proposal.** Ask your agent to propose a
   relational schema for this data: entities, fields, data types, primary
   keys, foreign keys, and which fields should be nullable versus
   required. If you're running Claude Code from the repository root,
   this is what the `schema-proposal` skill structures for you
   automatically; on other tools, ask directly for the same shape of
   answer (see "Using Codex CLI or opencode instead" below). A useful
   proposal names at least one alternative structure it considered and
   rejected, and lists open questions it wants you to answer before it
   builds anything.

   **Prompt:**

   ```
   Propose a relational schema for the customer, contact, subscription,
   and contract data in the corpus - entities, fields, data types,
   primary keys, foreign keys, and which fields should be nullable
   versus required. Include at least one alternative structure you
   considered and rejected, and list the open questions you want me to
   answer before you build anything.
   ```

3. **Ask the agent to argue for the paradigm, not just the schema.**
   Before touching constraints, ask it to state explicitly why a
   relational model fits this particular slice of the corpus: every
   entity here has a clear, stable identity (`customer_id`,
   `contact_id`, `subscription_id`, `contract_id`), the relationships
   between them are fixed and well-defined (one customer to many
   contacts, many subscriptions over time, at most one contract), and
   referential integrity genuinely matters — a subscription or contract
   pointing at a customer that doesn't exist is a real data error, not
   an edge case to tolerate. Decide whether you agree, and hold on to
   that reasoning: it's what you'll cite later when comparing this
   backend against MongoDB in the closing artifact.

   **Prompt:**

   ```
   Before we touch constraints, argue explicitly for why a relational
   model fits this slice of the corpus - the identity of each entity,
   the fixed relationships between them, and where referential
   integrity actually matters here. I want to decide whether I agree
   before we go further.
   ```

4. **Stop here and work through the decision point below before letting
   the agent proceed.** Do not tell it to create anything until you've
   made the calls in the next section yourself.

5. **Ask your agent to create the tables in Supabase**, matching the
   schema you've now signed off on, including every constraint (foreign
   keys, `NOT NULL`, uniqueness) you decided on in the previous step.

   **Prompt:**

   ```
   Create the tables in Supabase matching the schema I signed off on,
   including every constraint we decided on: the foreign keys, NOT
   NULL columns, and uniqueness rules.
   ```

6. **Ask your agent to build the population pipeline**: reading
   `customers.csv`, `contacts.csv`, and `subscriptions.csv`, extracting
   the relevant fields from each contract PDF in `contracts/`, and
   inserting all of it into the tables it just created. Ask it to state,
   before it runs anything at scale, exactly how it plans to handle the
   messy fields in `customers.csv` (mixed date formats, blanks,
   inconsistent country values) — this should match the decisions you
   made below, not a fresh set of choices it invents on the spot.

   **Prompt:**

   ```
   Build the pipeline that reads customers.csv, contacts.csv, and
   subscriptions.csv, extracts the relevant fields from each contract
   PDF in corpus/contracts/, and inserts all of it into the tables you
   just created. Before running it at scale, tell me exactly how you
   plan to handle the messy fields in customers.csv - the mixed date
   formats, the blanks, and the inconsistent country values - so I can
   confirm it matches what we decided.
   ```

7. **Ask your agent to run the pipeline** and report row counts per table
   as it completes.

   **Prompt:**

   ```
   Run the population pipeline and report the row count for each
   table as it completes.
   ```

8. **Ask your agent to verify what actually landed** against the source
   corpus. In Claude Code this is what the `pipeline-verify` skill is
   for; on other tools, ask directly for the same comparison (row counts
   against the source files, a field-by-field spot check on a handful of
   records, a clear pass/fail with concrete numbers rather than a vague
   "looks good").

   **Prompt:**

   ```
   Compare what actually landed in the Supabase tables against the
   source corpus: row counts against the source CSV and PDF counts, a
   field-by-field spot check on a handful of records, and a clear pass
   or fail with concrete numbers rather than a vague 'looks good'.
   ```

## Explicit decision point

Before any table gets created, treat the agent's first schema proposal as
a draft to interrogate, not a plan to approve. At minimum, decide the
following yourself:

- **Table boundaries.** Does the proposal give you four tables —
  customers, contacts, subscriptions, contracts — connected by
  `customer_id`? Or has the agent folded contract or subscription fields
  directly into the customers table? Decide whether that consolidation
  is justified, or whether it collapses a one-to-many relationship (a
  customer can have several subscription rows over time, tracking
  upgrades and downgrades) into something that can no longer represent
  history.
- **Foreign keys.** Does every child table (`contacts`, `subscriptions`,
  `contracts`) declare an actual, database-enforced foreign key back to
  `customers.customer_id` — not just a same-named column that happens to
  line up? Decide what should happen on delete. Restrict is the sensible
  default here: there's no legitimate reason to delete a customer and
  silently orphan or cascade away their billing history.
- **Nullability.** `customers.csv` has genuine blanks in `industry`,
  `company_size_band`, and one `signup_date`. Decide which columns should
  be nullable (almost certainly those) versus which must never be —
  `customer_id`, `plan_tier`, and `country` are never blank anywhere in
  the source, so decide whether to enforce that with `NOT NULL` at the
  database level rather than trust that every future load respects it.
- **The messy fields, specifically.** `signup_date` arrives in at least
  three formats (a majority ISO, a minority `MM/DD/YYYY`, and two
  spelled-out dates); `country` has inconsistent casing and abbreviations
  on a minority of rows; two company names carry a mojibake-style
  encoding artifact. For each of these, decide whether to normalize at
  load time (parse every date to one type, canonicalize country values)
  or to store the raw value alongside a cleaned one. Either is
  defensible. Leaving it for the agent to guess row by row, with no
  stated rule, is not.
- **Uniqueness.** Should `contacts.email` be constrained as unique?
  Should more than one `active` subscription row ever exist for the same
  customer at the same time? Decide what a duplicate would even mean
  here before asking the agent to prevent it.
- **Contracts as a subset, not a placeholder.** Only customers on a
  non-trial Professional or Enterprise plan have a contract, so most
  customers will have no matching row in `contracts`. Decide whether
  that's modeled as a nullable relationship — no contract row simply
  means no contract, which is correct here — rather than something that
  needs a default or placeholder row to satisfy a constraint. Also
  decide whether `contracts.customer_id` itself should be constrained as
  unique: this corpus models one Master Subscription Agreement per
  eligible customer, so a second contract row for the same customer is
  most likely a data problem, not a legitimate case to allow.

Once you've made these calls, tell your agent explicitly what you decided
and why, and only then move on to building the tables.

## Verification

**In the Supabase UI:**

- Open the Table Editor and confirm all four tables exist with the
  columns, types, and constraints you decided on above.
- Check row counts directly: `customers` should show 180 rows, `contacts`
  350, `subscriptions` 200, and `contracts` at most 50 — matching however
  many PDFs your agent successfully parsed. Fewer than 50 is a signal to
  investigate, not a rounding error to wave off.
- Try inserting a row into `contacts` with a `customer_id` that doesn't
  exist in `customers`, directly in the Table Editor, and confirm it's
  rejected. That's your foreign key constraint actually working, not just
  matching column names.

**Via your agent:**

- Ask it to run a query that counts rows per table and compares those
  counts to the source CSV and PDF counts — the same comparison the
  `pipeline-verify` skill runs — and report a clear pass or fail with
  numbers for each table.
- Ask it to run a join across all four tables for a handful of customers
  on the Enterprise plan: customer, their contacts, their current
  subscription, and their contract terms. Confirm the results make
  sense — the contract's monthly fee should match that customer's
  `mrr_usd`, by design in this corpus.
- Ask it to show you a handful of the messy `customers.csv` rows (a blank
  `industry`, a non-ISO `signup_date`, an inconsistent `country` value)
  as they now look in the `customers` table, so you can confirm they
  were handled the way you decided above, not some other way.

## Known pitfalls

- **No enforced foreign keys.** It's easy for an agent to create tables
  with matching column names and call it a relationship. Matching names
  are not a constraint — ask your agent to show you the actual
  constraint definitions, or check them in the Supabase UI.
- **Everything typed as text.** Agents will sometimes default to `text`
  for dates and money fields to sidestep parsing errors. Push back:
  `signup_date` and the subscription dates should be a real date type,
  `mrr_usd` a numeric type.
- **Silent date-parsing failures.** With three formats in `signup_date`,
  a pipeline that assumes a single format will silently null out or
  misparse the others. Ask your agent to report how many rows it could
  not parse cleanly, rather than trusting a clean-looking row count.
- **Re-running the pipeline creates duplicates.** Without a uniqueness
  constraint or an explicit "clear before reload" step, running the
  population pipeline a second time — after fixing something — can
  double up rows instead of replacing them. Ask your agent to make the
  pipeline safe to re-run before you rely on it as your recovery path.
- **Matching contracts to customers by filename instead of by content.**
  Contract filenames encode a contract ID and a company-name slug
  (`CTR-0019_quarry-construction.pdf`), but the customer ID and figures
  that matter are inside the PDF text. Confirm your agent resolves each
  contract to a customer using the ID it reads from inside the document,
  not by fuzzy-matching the filename slug against company names.
- **Restricted write access.** If Lab 0 was set up with a read-only
  Supabase key, table creation and inserts will fail partway through.
  Confirm the MCP connection has permission to create tables and write
  rows before starting the pipeline.

## Multi-tool notes

### Using Codex CLI or opencode instead

Both Codex CLI and opencode now support the same open Agent Skills format
(SKILL.md files) Claude Code uses — this is no longer Claude-Code-specific.
opencode scans `.claude/skills/` directly, so it already sees this repo's
`schema-proposal` and `pipeline-verify` skills: invoke them by name exactly
as in the steps above ("Run the schema-proposal skill...", "Run the
pipeline-verify skill..."). Codex CLI supports skills too, but scans
`.agents/skills/` instead, a path this repo doesn't use — so on Codex CLI
specifically, ask for the same outcome directly:

- Instead of invoking `schema-proposal`, ask your agent explicitly for
  the same shape of answer: "Propose a relational schema for this data —
  entities, fields, types, keys, and relationships — and include at
  least one alternative structure you considered and rejected, plus a
  list of open questions for me to decide before you build anything."
- Instead of invoking `pipeline-verify`, ask: "Compare the row counts and
  a sample of records between the source corpus files and what's now in
  the Supabase tables. Report pass or fail with concrete numbers for
  each table, and show me a few records side by side."
- Confirm your Supabase MCP connection was configured for whichever tool
  you're using back in Lab 0 — Codex CLI and opencode each use their own
  configuration file format for MCP servers, and the connection step
  differs slightly between tools, but once it's configured every step in
  this lab works identically regardless of which one you're driving.
