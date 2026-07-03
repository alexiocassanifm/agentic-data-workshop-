# Lab 4 — Dashboard

## Objective
Turn the data now sitting in your databases into a small set of business-relevant
charts, and confirm that every number on screen matches what is actually stored.
The point of this lab is not "make a pretty chart" — it is picking, before any
chart exists, which business questions matter enough to answer, and then
directing the agent to build exactly those and nothing else. By the end you
will have either a small running web dashboard or a single self-contained
HTML file, plus direct proof, via a database query, that its numbers are
correct.

## Duration
About 60 minutes: roughly 10 minutes deciding what to measure, 35–40 minutes
building and iterating with the agent, and 10–15 minutes verifying the result
against the database.

## Prerequisites
- The shared corpus at [`../corpus/`](../corpus/) — for reference only, you
  will not touch it directly in this lab.
- A populated Supabase project from [Lab 1](../lab-1-relational-supabase/README.md).
  This lab reads from the relational store you built there: customers,
  contacts, subscriptions, and whatever else your Lab 1 schema captured.
  Business metrics like MRR, churn, and counts by segment are naturally SQL
  aggregations, which is why this dashboard is scoped to the Supabase data
  rather than to MongoDB or the vector stores.
- By this point in the day you will also have been through Lab 2 (MongoDB)
  and Lab 3 ([semantic/vector search](../lab-3-vector-search/README.md)) on
  the same corpus. You do not need their output for this lab, but it is the
  same customers and subscriptions you have been comparing across backends
  all day.
- Confirm your agent still has a live MCP connection to that Supabase
  project. Ask it to check — the mcp-health-check skill does this, or you
  can just ask directly — it should be able to list your tables and report
  non-zero row counts.

## Guided steps
1. Do not ask for any chart yet. First decide which business questions this
   dashboard needs to answer — see "Explicit decision point" below — and
   write your final list down before moving on. If you are using Claude
   Code, invoking the `dashboard-build` skill at this point will structure
   the rest of this lab for you — it enforces the same order these guided
   steps walk through by hand, including refusing to build anything before
   you have written your questions down. On other tools, work through the
   numbered steps below directly (see "Using Codex CLI or opencode instead"
   under Multi-tool notes).
2. Tell your agent the list of questions/metrics you settled on. For each
   one, ask it to propose which table(s) and aggregation answer it, and what
   chart type (single number, bar, line, table) best represents it. Review
   this proposal before anything gets built; it is a plan, not a preview.

   **Prompt:**

   ```
   Here are the business questions I want this dashboard to answer:
   <YOUR LIST OF 3-5 QUESTIONS>. For each one, propose which table(s)
   and aggregation would answer it, and what chart type — single
   number, bar, line, or table — best represents it. Don't build
   anything yet; just show me the plan.
   ```
3. Choose which output you are building, based on the time you have left:
   - A small web app that queries Supabase live, each time the page loads.
   - A single self-contained HTML file that the agent queries Supabase to
     build once, with the results (and a small charting library) embedded
     directly in the file, so no live connection is needed to view it
     afterward.
   Tell your agent which one you want; do not let it default to whichever
   is easiest for it to produce.

   **Prompt (pick one):**

   ```
   I want a small web app that queries Supabase live each time the
   page loads. Let's build that.
   ```

   — or —

   ```
   I want a single self-contained HTML file: run the queries once and
   embed the results and a small charting library directly in the
   file. Let's build that.
   ```
4. For the web app variant: ask your agent to scaffold a minimal app, wire
   it to Supabase using your project's connection details — your Supabase
   project URL and access token — describing these to your agent by name
   rather than pasting them into a file it will show you, and run the app
   locally so you can view it in a browser.

   **Prompt (web app variant):**

   ```
   Scaffold a minimal web app and wire it to my Supabase project using
   my project URL and access token — read those from an environment
   variable or local config you set up, don't put them in a file you
   show me. Then run it locally so I can view it in a browser.
   ```

   For the HTML artifact variant: ask your agent to run the queries for each
   metric now and generate one HTML file containing the results and the
   charts, then open that file in a browser.

   **Prompt (HTML artifact variant):**

   ```
   Run the queries for each metric now, and generate one
   self-contained HTML file with the results and the charts embedded
   directly in it. Then open that file so I can view it.
   ```
5. Ask your agent to label every chart and number with the business
   question it answers, in plain language (for example "Active
   subscriptions" or "MRR by plan tier"), not with raw column or table
   names.

   **Prompt:**

   ```
   Label every chart and number with the business question it
   answers, in plain language — for example 'Active subscriptions' or
   'MRR by plan tier' — not with raw column or table names.
   ```
6. Review the first version together. Pick one or two concrete things to
   change — a grouping, a sort order, a missing filter, a second axis — and
   ask the agent to make that specific change. Repeat once or twice; resist
   the urge to keep adding new metrics at this stage.

   **Prompt:**

   ```
   Make this one specific change: <THE ONE CHANGE YOU AND YOUR AGENT
   JUST AGREED ON — e.g. a grouping, sort order, missing filter, or
   second axis>. Don't add any other metrics yet.
   ```
7. Once you are satisfied with the layout, move to Verification below
   before calling the lab done.

## Explicit decision point
Before step 2 above, decide on 3 to 5 business questions this dashboard will
answer. Do this yourself, on paper or out loud with your agent as a
sounding board — do not ask the agent to propose the list and then simply
approve it. For each question, be able to say what decision it would inform
and roughly which data it depends on.

A menu to choose from — adapt it to whatever actually ended up in your
Supabase tables, and mix types freely:
- **A single headline number**: total current MRR; count of active
  customers; churn rate (churned customers versus active customers).
- **A breakdown by category**: MRR or customer count by plan tier;
  customers by industry; customers by country; customers by company size
  band.
- **A trend over time**: signups by month or quarter; MRR trajectory, if
  your schema tracks subscription history.
- **A risk or health signal**: proportion of customers on trial versus
  paying; count of contracts approaching renewal, if your Lab 1 schema
  modeled contract terms.

Deliberately pick a mix — at least one single number, one category
breakdown, and one trend — so the dashboard exercises more than one chart
type. Write your final 3–5 down before telling the agent to build anything.

## Reference: what's actually in this corpus

The menu above is deliberately abstract so that picking the final list stays
your decision, not the agent's. To ground your thinking, here is what is
actually in this repo's `corpus/customers-export/customers.csv` and
`subscriptions.csv` right now, computed directly from those two files —
not invented placeholders, so you can treat these as ground truth. Use
them for inspiration while you pick your 3-5 questions, and again later to
sanity-check whatever numbers your dashboard ends up showing. This is not
a checklist to copy-paste as your final answer — you still have to decide
that list yourself, per "Explicit decision point" above.

**Single headline numbers**
- Total current MRR across active subscriptions: **$603,547** (156 active
  subscription rows out of 200 total).
- Total customers: **180**.
- Churn rate: **about 11.4%** (20 distinct customers with a churned
  subscription row versus 156 with an active one).

**A breakdown by category**
- MRR and customer count by plan tier, active subscriptions only:
  Enterprise **$394,327** across 23 subscriptions; Professional
  **$160,389** across 44; Growth **$45,066** across 61; Starter **$3,765**
  across 28.
- Customers by industry: 15 distinct industries populated (4 customers
  have a blank industry value). The three largest are Nonprofit & Public
  Sector (19), Energy & Utilities (17), and Media & Entertainment (16).
- Customers by country: 29 distinct raw string values — but several of
  them are the same country spelled differently. "United States" (54),
  "united states" (3), and "US" (1) are all one country (58 combined);
  "Denmark" (1) and "DENMARK" (1) are another. This is a live instance of
  the country-fragmentation pitfall described later in Known pitfalls —
  notice it and decide what to do about it yourself; it is left unmerged
  here on purpose.
- Customers by company size band: 51-200 employees is the largest band
  (59 customers), then 11-50 (43), 201-500 (38), 1-10 (14), 1,000+ (13),
  501-1,000 (10); 3 customers have a blank size band.

**A trend over time**
- Signups by quarter run from 2022 Q1 through 2025 Q4 — 16 quarters —
  ranging from a low of 5 signups (2025 Q4) to a high of 18 (2023 Q1).
  Real variation, not a flat line.

**A risk or health signal**
- 4 subscription rows are currently `trial` status, out of 200 total.
  Note that `trial` only ever appears in `subscriptions.status` — it
  never appears in `customers.plan_tier` — a distinction worth surfacing
  to your agent if one of your questions touches trial customers.
- Separately, 19 customers have more than one subscription row — an
  upgrade or downgrade history — a candidate for a "customers with a plan
  change" signal.

Again: this box is for inspiration and later sanity-checking, not a
shortcut around picking your own 3-5 questions.

## Bonus: choosing a JS charting library

If any of your chart types is a bar, line, or table (a single-number tile
needs no library at all), the output format you picked in step 3 also
implies a charting-library decision. Don't let the agent default silently
to whichever library it reaches for first — a few real options, without
one obviously "correct" answer:

- **Chart.js** — a handful of declarative chart types (bar, line, pie),
  loaded from a single CDN script tag, no build step. A good fit for a
  fast self-contained HTML artifact.
- **Observable Plot or ECharts** — similarly CDN-friendly and
  declarative, with broader chart-type coverage than Chart.js, still no
  build step.
- **Recharts** — a React component library. It only fits the live
  web-app variant, and only if your agent scaffolds a React app; it does
  not fit the plain self-contained HTML artifact.
- **D3.js** — the most control, with no built-in chart types at all: axes,
  scales, and shapes are composed by hand through the agent. Worth it when
  a chart needs a shape none of the above provide out of the box; more
  code than a standard bar/line/single-number dashboard needs.

Ask your agent to name the library it wants to use and justify the choice
against the output format and chart types you already decided on, before
it builds anything.

**Prompt:**

```
Before building anything, tell me which JS charting library you plan
to use for this dashboard and why, given the output format and chart
types we already decided on.
```

## Verification
In the provider's UI: open your Supabase project's Table Editor, apply the
same filter as one of your dashboard metrics (for example, filter the
`subscriptions` table to `status = active` and read the row count off the
UI), for at least two of your metrics. No query typing required. If a
metric needs an aggregation the Table Editor cannot show directly (a sum
or a group-by), ask your agent for the exact query behind that number and
paste it into the SQL editor instead. Compare the number you get to the
number on the dashboard.

Via the agent: ask it to run a direct check comparing every dashboard
number against a fresh query against Supabase, and to report any mismatch
with both numbers side by side, not just "looks correct." This is the same
comparison the pipeline-verify skill performs after a population pipeline —
here you are pointing it at the dashboard instead of at the database alone.

Two numbers worth checking specifically, because they are easy to get
wrong:
- **Active subscriptions**: a direct `status = 'active'` count should
  return 156. If a chart or KPI reads a different number, it is almost
  certainly counting all 200 subscription rows rather than just the
  current ones.
- **Total customers**: 180. If a customer-count tile reads higher than
  that, it is counting subscription rows or contacts instead of distinct
  customers.

If any number does not match, ask the agent which query produced the
on-screen number and which query produced the verification number, then
have it explain the discrepancy before you accept the dashboard as correct.

## Known pitfalls
- **Double-counting historical subscription rows.** `subscriptions.csv` has
  200 rows but only 156 are currently active; the other 44 are churned,
  trial, or `upgraded` — a superseded historical row for a customer who has
  since changed plans (about 19 customers have more than one row). Any
  `SUM(mrr_usd)` or `COUNT(*)` grouped by plan that does not filter to
  `status = 'active'`, or that does not take only the latest row per
  customer, overstates revenue and customer counts.
- **Usage-log coverage gap.** Only 177 of the 180 customers have a
  usage-log file — 3 very recently signed customers have no usage history
  yet. Any usage-based metric, if your schema pulled usage data into
  Supabase, will be missing those 3 rows. That is a real, expected join
  gap, not a bug, but it is worth a one-line footnote on the dashboard
  rather than a silent omission.
- **Inherited messiness from customers.csv.** If `country` or `industry`
  values were not normalized during Lab 1, or were normalized in a way you
  do not remember, a "customers by country" chart can fragment into
  near-duplicate categories, such as `"US"` and `"United States"` as
  separate bars. Do not let the agent silently merge categories it invents
  on the spot while building the chart — either fix it at the source in
  Lab 1's tables, or flag the fragmentation visibly.
- **Charting before deciding.** If you let the agent propose metrics and
  charts before you have picked your questions, review time gets spent
  tuning colors and layout instead of checking whether the dashboard
  actually answers something. The order in the guided steps above is
  deliberate.
- **Credentials in the web app variant.** If you choose the live web app,
  make sure your agent reads your Supabase project URL and access token
  from an environment variable or local config it does not show you or
  commit anywhere, rather than writing them directly into a file. Ask it to
  confirm how it is storing them before you consider the app done.
- **The HTML artifact is a snapshot.** It is correct at the moment it is
  generated but will not reflect later changes to the database. If you
  update data in Supabase afterward, you have to ask the agent to
  regenerate the file; it will not refresh itself.

## Multi-tool notes
### Using Codex CLI or opencode instead
Both Codex CLI and opencode now support the same open Agent Skills format
(SKILL.md files) Claude Code uses — this is no longer Claude-Code-specific.
opencode scans `.claude/skills/` directly, so it already sees this repo's
`mcp-health-check`, `dashboard-build`, and `pipeline-verify` skills: invoke
them by name exactly as you would in Claude Code (for example, "Run the
dashboard-build skill"). Codex CLI supports skills too, but scans
`.agents/skills/` instead, a path this repo doesn't use — so on Codex CLI
specifically, ask your agent directly to perform the same steps: "list the
tables in my Supabase project and their row counts" at the start, and
"compare every number in the dashboard against a fresh query against the
database and show me both numbers" at the end.

On Codex CLI, instead of the `dashboard-build` skill specifically, ask your
agent directly for the same discipline it would otherwise enforce: to hold
off proposing or building any chart until you have stated your business
questions, and to refuse if you ask it to invent that list for you; to
propose, for each question, which table(s), aggregation, and chart type
would answer it, and stop for your review before building anything; to
ask you explicitly which output format and which charting library you
want, rather than picking either on its own; and, once something is
built, to independently re-run every on-screen number as a fresh query
against the database and report any mismatch with both numbers side by
side, rather than asserting it looks correct.

For the live web app variant, Claude Code can often run and preview a
local server for you inline. With Codex CLI or opencode you may need to
ask your agent for the local address it started the server on and open
that address yourself in a browser, then report back what you see so the
agent can keep iterating with you.

Credential handling works the same way in principle across tools: describe
your Supabase project URL and access token to your agent by name and let
it store them in whatever environment-variable or local-config mechanism
that tool uses, rather than pasting them into a source file.
