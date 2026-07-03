# Lab 3 — Semantic Search: Qdrant and pgvector

## Objective

In Lab 1 you designed and enforced a relational schema on Supabase. In Lab
2 you modeled the same corpus with no schema at all, in MongoDB. This lab
introduces a third paradigm, and it is not "the same corpus again, another
tool": there is no schema to design here. The free-text sources you've
mostly set aside so far — support tickets, sales notes, onboarding emails
— become embeddings, numeric vectors a database can compare for closeness
of *meaning* rather than exact value. A good query for "customers upset
about login problems" should surface tickets, notes, and emails that never
use those exact words.

Voyage AI plays two different roles in this lab, one required and one
optional, and it matters which is which. Postgres has no embedder built
in, so pgvector — the extension you'll use to turn your Lab 1 project into
a vector store — has no way to turn text into a vector on its own; calling
Voyage AI directly is the only way to get a vector into it at all. Qdrant
is different: its official MCP server embeds text with its own built-in
model the moment you ask it to store something, so Qdrant can be built,
loaded, and queried without Voyage ever entering the picture. Because of
that asymmetry, this lab does Qdrant first, start to finish, using its own
batteries-included embedding path — then does pgvector second, start to
finish, where Voyage isn't a refinement but the only way in. An optional
step at the end shows you how to make the two directly comparable on
identical embeddings, for anyone who wants that stricter test.

You'll run the same search questions in two architecturally different
places: Qdrant, a database built for nothing but vectors, reached through
its official, close-to-batteries-included MCP server; and pgvector, an
extension that turns the very same Postgres project from Lab 1 into a
vector store, where you control the embeddings, the index, and the query
directly. Neither is the automatically correct answer. By the end of this
lab you will have fully built and queried both, one after the other, and
be ready to argue, from what you actually observed, which one you would
put into production for this use case.

> **Method note.** Qdrant's official MCP server is close to
> batteries-included: point it at your cluster, and a single tool call
> embeds text and stores it, creating the collection for you if needed.
> pgvector is the opposite: you enable an extension, choose an index type,
> write the table, and construct the similarity query yourself, right
> alongside the relational data from Lab 1. Neither is "the way" to do
> semantic search — a dedicated vector database earns its place when
> vector search is a first-class, high-volume workload; extending the
> database you already operate earns its place when vectors are one more
> thing living next to data you already query relationally. Deciding
> which is true for Larkspur is the actual point of doing this lab twice.
> The written verdict you produce at the end is this lab's own
> comparison — a counterpart to the Bonus lab's Supabase-vs-MongoDB
> artifact, not an input to it.

## Duration

Approximately 90 minutes for the required Parts 1 through 3, plus roughly
15 more minutes if you do the optional bonus:

- ~25 min: **Part 1 — Qdrant first.** Connection check, picking a pilot
  batch, chunking and metadata, creating the collection and loading the
  pilot letting Qdrant embed for you, verifying it landed, scaling to the
  full 774 documents, and running the comparison queries against Qdrant
  alone.
- ~40 min: **Part 2 — pgvector second.** Connection and Voyage API key
  check, picking the embedding model, generating Voyage embeddings and
  loading the pilot into pgvector, verifying it landed, scaling to the
  full 774 documents, and running the same comparison queries against
  pgvector alone.
- ~25 min: **Part 3 — compare and decide.** Summarizing what you noticed
  operating each backend, then writing the recommendation.
- ~15 min (optional): **Bonus — true apples-to-apples.** Rewriting the
  Qdrant load with the same Voyage vectors used in pgvector and re-running
  both queries.

## Prerequisites

- **Lab 0 completed**, specifically: a live Supabase MCP connection
  pointing at your Lab 1 project, a live Qdrant Cloud MCP connection with
  write access (not read-only), and a Voyage AI API key your agent can
  use. See [`../lab-0-setup/README.md`](../lab-0-setup/README.md).
- **Lab 1 completed.** This lab writes pgvector data into the same
  Supabase/Postgres project you built in Lab 1 — the new vectors will
  live alongside the `customers`, `contacts`, and `subscriptions` tables
  you already created. See
  [`../lab-1-relational-supabase/README.md`](../lab-1-relational-supabase/README.md).
- **Lab 2 (MongoDB) is not required for this lab.** Nothing here depends
  on what you built there, though it's assumed you've done it as part of
  the same day.
- The shared corpus, described in full in
  [`../corpus/README.md`](../corpus/README.md). This lab uses only the
  free-text sources:
  - `corpus/support-tickets/` — 500 files, `TCK-#####.txt`
  - `corpus/sales-notes/` — 150 files, `SN-####.txt`
  - `corpus/emails/` — 124 files, `EML-####.txt`

  774 documents in total. The customer CSVs, contract PDFs, and usage-log
  JSON from Lab 1 and Lab 2 are not used in this lab.
- No prior experience with vector databases, embeddings, or SQL is
  required. Everything below is done by describing what you want to your
  agent.

## Guided steps

### Part 1 — Qdrant first

Qdrant goes first because, through its official MCP server, it needs
nothing from you but a connection and some text — no Voyage, no
dimensionality bookkeeping. Build it, load it, and query it end to end
before pgvector enters the picture at all.

1. **Check the Qdrant connection, and pick a pilot batch.** Ask your
   agent to run the `mcp-health-check` skill (or the manual equivalent)
   and confirm the Qdrant MCP connection points at your Qdrant Cloud
   cluster and has write access, not read-only. Then ask it to list the
   support tickets, sales notes, and emails folders with their counts
   (774 files combined), and propose a small pilot batch — for example
   15–20 tickets, 10 notes, and 8 emails — to prove the whole pipeline
   end to end before committing to embedding all 774.

   **Prompt:**

   ```
   Run the mcp-health-check skill to confirm Qdrant is connected and has
   write access, not read-only. Then list the files in
   corpus/support-tickets/, corpus/sales-notes/, and corpus/emails/ with
   their counts, and propose a small pilot batch — around 15-20 tickets,
   10 notes, and 8 emails — so we can test the whole pipeline before
   embedding all 774 documents.
   ```

2. **Settle chunking and metadata.** Ask your agent to confirm that each
   file is short enough to embed whole, as a single chunk, without
   splitting (flag anything unusually long), and to propose the small set
   of metadata fields to carry alongside every vector: source type,
   source ID, `customer_id`, and one or two useful fields per type
   (priority and status for tickets, account executive for notes, subject
   line for emails). These same fields become the columns you store
   alongside each vector in pgvector in Part 2, so it's worth settling
   them once, now.

   **Prompt:**

   ```
   Check whether each file in the pilot batch is short enough to embed
   as a single chunk without splitting it, and flag anything unusually
   long. Then propose the metadata fields to store alongside every
   vector: source type, source ID, customer_id, and one or two extra
   fields per type — priority and status for tickets, account executive
   for notes, subject line for emails.
   ```

3. **Create the Qdrant collection and load the pilot batch**, letting
   Qdrant's own built-in embedder turn the raw text into vectors — this
   is the simple, batteries-included default, and there's no need to
   generate embeddings yourself or decide on a dimensionality up front;
   the official Qdrant MCP server's store tool creates the collection for
   you and sizes it to its own model automatically.

   **Prompt:**

   ```
   Create a Qdrant collection for the pilot batch and load the pilot
   documents and their metadata into it, letting Qdrant's own built-in
   embedder handle the text directly. Don't generate embeddings for this
   batch yourself.
   ```

4. **Verify the pilot landed correctly in Qdrant.** In Claude Code this
   is what the `pipeline-verify` skill is for; on other tools, ask
   directly for the same comparison — counts against the pilot batch,
   a field-by-field spot check on a handful of records, and a clear pass
   or fail with concrete numbers.

   **Prompt (Claude Code):**

   ```
   Run the pipeline-verify skill to check what actually landed in
   Qdrant, for the pilot batch, against the source files.
   ```

   **Prompt (Codex CLI / opencode):**

   ```
   Compare the number of pilot documents I loaded against the point
   count in Qdrant, spot-check a handful of records field by field
   against the original files, and report pass or fail with concrete
   numbers.
   ```

5. **Once the pilot checks out, scale up to the full 774 documents in
   Qdrant.**

   **Prompt:**

   ```
   Now that the pilot load checks out, scale up to the full 774
   documents in Qdrant.
   ```

6. **Run the comparison queries against Qdrant.** Ask your agent to run
   a semantic query — something like "customers frustrated about single
   sign-on breaking their login" — and present the top results with a
   short snippet and a similarity score. Then ask it to run a second,
   cross-genre query that deliberately spans support tickets and sales
   notes (the corpus reuses SSO-frustration vocabulary across both), and
   check whether Qdrant surfaces more than one document type, not just
   the most literal match. Hold onto both sets of results — you'll
   compare them against pgvector's in Part 2.

   **Prompt:**

   ```
   Run the query 'customers frustrated about single sign-on breaking
   their login' against Qdrant, and show me the top results with a
   short snippet and similarity score for each. Then run a second query
   that could reasonably match both a support ticket and a sales note,
   and check whether Qdrant surfaces more than one document type, not
   just the most literal match. Keep a note of both queries' exact
   wording and results — we'll run the same two against pgvector next.
   ```

### Part 2 — pgvector second

pgvector needs more from you: Postgres has no embedder of its own, so
every vector that goes in has to come from somewhere — Voyage AI, in this
lab. Build it, load it, and query it end to end the same way you just did
for Qdrant.

7. **Check the Supabase connection, and confirm your Voyage AI API key.**
   Ask your agent to run the `mcp-health-check` skill (or the manual
   equivalent) and confirm the Supabase MCP connection points at your
   Lab 1 project. Separately, confirm a Voyage AI API key is actually
   available to it — you'll need it for everything in this Part.

   **Prompt:**

   ```
   Run the mcp-health-check skill to confirm Supabase is connected and
   points at my Lab 1 project. Separately, confirm you have my Voyage AI
   API key available for this lab.
   ```

8. **Pick and record the embedding model.** Ask your agent to check
   Voyage's current documentation for the recommended general-purpose
   text embedding model, and to report back the exact model name and the
   vector dimensionality it produces. Write that number down — you'll
   need it to size the pgvector table correctly.

   **Prompt:**

   ```
   Check Voyage AI's current documentation for the recommended
   general-purpose text embedding model, and tell me the exact model
   name and the vector dimensionality it produces.
   ```

9. **Generate embeddings for the pilot batch.** Ask your agent to call
   the Voyage embeddings API directly on the pilot documents — not
   through any database's built-in embedder — batching many documents
   into each API call rather than sending one call per document. Ask it
   to confirm the number of documents embedded and the resulting vector
   dimensionality match what you expect.

   **Prompt:**

   ```
   Write and run a short script that calls the Voyage embeddings API
   directly on the pilot batch documents — not through any database's
   built-in embedder — batching many documents into each API call
   rather than one call per document. Confirm how many documents got
   embedded and that the vector dimensionality matches what we
   expect.
   ```

10. **Enable pgvector, create the table, and load the pilot batch.** Ask
    your agent to enable the pgvector extension on the Lab 1 Supabase
    project (if it isn't already), create a new table for these
    embeddings sized to the Voyage dimensionality from step 8, load the
    pilot vectors and metadata into it, and then add an appropriate
    index. Ask it to explain briefly why it picked the index type it
    did — and if it picks an index type that trains on existing data
    (IVFFlat), confirm it's building that index after the rows are
    loaded, not on an empty table.

    **Prompt:**

    ```
    Enable the pgvector extension on my Lab 1 Supabase project if it
    isn't already, create a new table for these embeddings sized to the
    same dimensionality, load the pilot vectors and metadata into it, and
    add an appropriate index. Tell me briefly why you picked that index
    type, and if it's one that trains on existing data, confirm you're
    building it after the rows are loaded, not on an empty table.
    ```

11. **Verify the pilot landed correctly in pgvector.** In Claude Code
    this is what the `pipeline-verify` skill is for; on other tools, ask
    directly for the same comparison — counts against the pilot batch, a
    field-by-field spot check on a handful of records, and a clear pass
    or fail with concrete numbers.

    **Prompt (Claude Code):**

    ```
    Run the pipeline-verify skill to check what actually landed in
    pgvector, for the pilot batch, against the source files.
    ```

    **Prompt (Codex CLI / opencode):**

    ```
    Compare the number of pilot documents I embedded against the row
    count in the pgvector table, spot-check a handful of records field
    by field against the original files, and report pass or fail with
    concrete numbers.
    ```

12. **Once the pilot checks out, scale up to the full 774 documents**,
    batching the Voyage calls again to stay within its rate limits, and
    load the resulting vectors and metadata into pgvector.

    **Prompt:**

    ```
    Now that the pilot load checks out, scale up to the full 774
    documents, batching the Voyage calls to stay within its rate limit,
    and load the resulting vectors and metadata into pgvector.
    ```

13. **Run the same comparison queries against pgvector.** Ask your agent
    to run the identical SSO-frustration query against pgvector, and
    present the top results with a short snippet and a similarity score.
    Then ask it to run the exact same cross-genre query you used against
    Qdrant in Part 1, and check whether pgvector surfaces more than one
    document type, not just the most literal match.

    **Prompt:**

    ```
    Run the query 'customers frustrated about single sign-on breaking
    their login' against pgvector, and show me the top results with a
    short snippet and similarity score for each. Then run the same
    second, cross-genre query you used against Qdrant in Part 1, and
    check whether pgvector surfaces more than one document type, not
    just the most literal match. Note both sets of results so we can
    compare them against what Qdrant returned.
    ```

### Part 3 — Compare and decide

14. **Ask your agent to summarize what it noticed** operating each
    backend during this lab — which one required more explicit decisions
    from you, which one hid more machinery, how latency and result
    quality compared. This is the raw material for the decision point
    below.

    **Prompt:**

    ```
    Summarize what you noticed operating each backend in this lab —
    which one needed more explicit decisions from me, which one hid
    more of the mechanism, and how latency and result quality
    compared.
    ```

### Optional bonus — true apples-to-apples

Because Qdrant embedded with its own model in Part 1, and pgvector used
Voyage in Part 2, the two backends answered the comparison queries in two
different embedding spaces. That's a fair comparison of the two systems
in general, but not a strict "same embeddings, different store" test. If
you want that stricter test, this step is for you — it's genuinely
optional, not a gate you need to pass before Qdrant can be considered
"done."

15. **(Optional) Redo the Qdrant load with the same Voyage vectors used
    in pgvector.** Ask your agent to write the Voyage vectors you already
    generated for the full 774 documents directly into Qdrant via its
    API, going around the store tool's own embedding step (it does this
    by calling Qdrant's API directly with the vector, rather than handing
    it raw text). Then re-run both comparison queries and note whether or
    how the results shift compared to Part 1. If you take this path, you
    must also retrieve using that same direct path for every query —
    never the Qdrant MCP's own query tool — or you're comparing two
    different embedding spaces all over again.

    **Prompt:**

    ```
    Redo the Qdrant load: write the Voyage vectors we already generated
    for the full 774 documents directly into Qdrant via its API, going
    around the store tool's own embedding step. Then re-run both
    comparison queries — using that same direct path for retrieval, not
    the Qdrant MCP's own query tool — and tell me whether or how the
    results shifted compared to Part 1.
    ```

## Explicit decision point

### Where should Larkspur's semantic search actually live?

Once both stores are loaded, verified, and queried, write down — in a
scratch note your agent keeps for you, the same way you did in Lab 2 —
one paragraph answering: for Larkspur's own support, sales, and email
corpus, would you recommend building semantic search on top of the
Postgres project you already operate (pgvector), or standing up Qdrant as
a dedicated store? Ground the answer in at least two concrete things you
personally saw in this lab, not database theory in the abstract — for
example, how much you had to configure by hand on each side, how the two
result sets actually differed on the same query, how filtering by
metadata (`customer_id`, priority, source type) felt in each, or how much
control you had over the index and the embedding itself. There is no
single correct answer here; the discipline is citing your own
observations, not defaulting to whichever backend felt more polished on
first contact.

## Verification

Under the default path in this lab (Parts 1 and 2, without the optional
bonus), expect Qdrant and pgvector to end up with different embedding
dimensions and different underlying models — that's the correct outcome
of Qdrant self-embedding and pgvector using Voyage, not a sign something
went wrong.

**In the Qdrant Cloud UI:**

- Open your cluster, go to Collections, and open the collection you
  created. Confirm the points count matches the number of documents you
  loaded (774, or your pilot batch's count if you haven't scaled up
  yet).
- Check the collection's configured vector size and distance metric.
  Under the default path (Part 1), Qdrant sized and created the
  collection automatically using its own built-in embedder's model and
  dimension — that's expected. Only if you completed the optional bonus
  step (writing Voyage vectors directly into Qdrant) should this instead
  match the Voyage dimensionality you recorded in Part 2, with cosine
  distance.
- Open a few individual points and confirm the payload holds the
  metadata you decided on (source type, source ID, `customer_id`, and so
  on) and reads back correctly for a document you recognize.

**In the Supabase UI:**

- Table Editor: confirm the new embeddings table exists with the
  expected row count, and Database → Extensions shows `pgvector` enabled.
- Open the SQL editor (or ask your agent to do it for you) and look at
  one row: confirm the vector column is not null and has the length you
  expect for the Voyage model you used.

**Via your agent:**

- Ask it to count points in Qdrant and rows in the pgvector table and
  compare both to the number of source files you loaded — the same
  comparison the `pipeline-verify` skill runs — and report a clear pass
  or fail with numbers, not "looks about right."
- Ask it to spot-check a handful of records by source ID against the
  original file, confirming the metadata matches (for example, a
  ticket's priority and status in the stored payload match what's
  actually written in the `.txt` file).
- Ask it to re-run the two semantic queries from the guided steps against
  both backends one more time, and present the results side by side,
  exactly as you'll want them for the decision-point writeup above.

## Known pitfalls

- **Voyage's rate limit, hit the hard way.** Without a payment method on
  file, Voyage's API throttles to a handful of requests per minute.
  Embedding 774 documents one call at a time at that rate can take hours.
  Always ask your agent to batch many documents into each API call, and
  pilot on a small subset before committing to the full corpus.
- **Vector-space mismatch between store and retrieve.** Under this lab's
  default path, Qdrant embeds with its own built-in model and pgvector
  uses Voyage — the two backends live in different embedding spaces, and
  that's expected, not a bug. It only becomes a real problem if you do
  the optional bonus step (writing Voyage vectors directly into Qdrant)
  and then mix retrieval paths: falling back to the Qdrant MCP's own
  query tool "just for a quick check" re-embeds your query text with its
  own built-in model and compares it against vectors from a different
  embedding space than the ones you wrote — results will look wrong, or
  the query will simply fail, for reasons that have nothing to do with
  semantic search itself. If you do the optional bonus, query directly,
  the same way, every time.
- **A read-only MCP connection quietly removes the write tools.** If
  your Qdrant or Supabase MCP connection from Lab 0 was configured
  read-only, the tools needed to create a collection or table, or write
  points or rows, may not be available at all, or will fail partway
  through. Confirm write access on both sides before starting.
- **Dimension or distance mismatch.** The pgvector column always needs
  the exact dimensionality your Voyage model actually produces. The
  Qdrant collection, under the default path in Part 1, sizes itself
  automatically to its own built-in model's dimension — that's the
  expected, correct outcome, not a bug, and it's normal for the two
  collections to end up with different dimensions and even different
  distance metrics as a result. The dimension only needs to match
  Voyage's if you complete the optional bonus step, where the Qdrant
  collection must be sized for the Voyage vectors you're writing into it
  directly. Whichever path you're on, keep the distance metric (cosine)
  consistent between whatever you're actually comparing, or the
  comparison isn't really comparing the same thing.
- **Mismatched chunking between the two backends.** If one backend ends
  up with whole files as chunks and the other with paragraph-split
  chunks, the two result sets no longer describe the same units of text.
  Keep chunking identical on both sides.
- **Cluster or project idle suspension.** Free-tier Qdrant Cloud clusters
  and Supabase projects can pause themselves after roughly a week of no
  activity. If you're repeating this lab at home some days after Lab 0,
  check both dashboards are awake before asking your agent to do
  anything.
- **Treating the pilot batch as the final answer.** The pilot exists to
  catch mistakes cheaply. Make sure your agent actually scales up to the
  full 774 documents — or explicitly tells you why it didn't — before you
  run the comparison queries; a comparison built on a partial, arbitrary
  slice of the corpus isn't a fair one.

## Multi-tool notes

### Using Codex CLI or opencode instead

Both Codex CLI and opencode now support the same open Agent Skills format
(SKILL.md files) Claude Code uses. opencode scans `.claude/skills/`
directly, so it already sees this repo's `mcp-health-check` and
`pipeline-verify` skills: invoke them by name exactly as in the steps
above. Codex CLI supports skills too, but scans `.agents/skills/` instead,
a path this repo doesn't use — so on Codex CLI specifically, every outcome
in this lab is reachable by asking directly:

- Instead of `mcp-health-check`, ask your agent to check its Qdrant
  connection in Part 1, and its Supabase connection plus your Voyage AI
  API key in Part 2, and report, in plain language, what it can
  currently see on each (tables or collections, row or point counts,
  read versus write access).
- Instead of `pipeline-verify`, ask, once per backend: "Compare the
  number of documents I loaded against the point count in Qdrant (or the
  row count in the pgvector table), spot-check a handful of records
  field by field against the original files, and report pass or fail
  with concrete numbers."
- Confirm both your Supabase and Qdrant MCP connections were configured
  for whichever tool you're using back in Lab 0 — Codex CLI and opencode
  each use their own configuration file format for MCP servers, and the
  connection step differs slightly between tools, but every step in this
  lab works identically once both connections are live.
- The optional bonus step (writing Voyage vectors directly into Qdrant,
  bypassing the store tool's own embedding step) is not a
  Claude-Code-specific technique — it's your agent calling Qdrant's API
  directly on your behalf, which works the same way regardless of which
  agent is driving.
