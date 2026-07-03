---
name: dashboard-build
description: Use when building Lab 4's dashboard against the Supabase data populated in Lab 1 — structures the build so the agent never proposes or builds a chart before the student has stated their own 3-5 business questions, turns each question into a table/aggregation/chart-type plan the student reviews before anything gets built, requires an explicit student choice between output format (live web app vs. self-contained HTML) and charting library instead of a silent default, and independently re-verifies every on-screen number against a fresh database query before the dashboard is presented as done.
---

# Dashboard Build

## Purpose

This skill structures how you (the agent) build the Lab 4 dashboard with a
student who is a data scientist or data engineer, not a developer — they
will never read raw logs, stack traces, or code, and everything you
produce under this skill is written in plain, everyday business language.

The single most important rule runs the opposite direction from the
schema-proposal skill: here, the student decides the business questions,
and your job is only to propose the technical translation of those
questions into tables, aggregations, and chart types — never the
questions themselves. You never build a single chart or query until the
student has told you, in their own words, the business questions this
dashboard needs to answer.

## When this applies

Use this skill whenever the student is building the Lab 4 dashboard: any
time they ask you to propose metrics, pick charts, scaffold a web app or
HTML artifact, or start querying Supabase for a dashboard.

## Step 1 — Wait for the student's own questions

Before doing anything else, ask the student whether they have already
decided the 3-5 business questions for this dashboard, per Lab 4's
"Explicit decision point."

- If they have not decided yet, stop here. Do not suggest chart ideas, do
  not sketch a layout, and do not touch Supabase. You may act as a
  sounding board if the student wants to think out loud, but the final
  list has to come from them.
- If the student asks you to just invent the list yourself and skip the
  decision, refuse plainly and point back to the lab's "Explicit decision
  point" section: explain that picking the questions is the one part of
  this lab that has to stay theirs, and that a list you invent on their
  behalf would defeat the point of the exercise. Offer to help them think
  it through instead — for instance by pointing them at the "Reference:
  what's actually in this corpus" numbers in the README for inspiration —
  but do not hand them a finished list of questions as a shortcut.
- Once the student states their own list, repeat it back in plain
  language before moving on, so a misunderstanding gets caught early
  rather than after something gets built.

## Step 2 — Propose the technical plan, then stop

For each business question the student gave you, in order, propose:

- Which table or tables in Supabase answer it.
- What aggregation is needed (a count, a sum, a group-by, a filter to the
  latest or active row per customer, and so on), described in plain
  words, not as a raw query.
- What chart type best represents it: a single number, a bar chart, a
  line chart, or a table.

Present this as a plan covering every question before building anything.
This is a proposal for the student to review, not a preview of the
finished dashboard — stop after presenting it and wait for the student to
approve it or ask for changes before you write or run a single query.

## Step 3 — Ask for the output format and the charting library, explicitly

Do not default silently to whichever option is easiest for you to
produce. Ask the student directly to choose both of the following, and
wait for an answer to each before writing or running anything:

- **Output format:** a small web app that queries Supabase live each time
  the page loads, or a single self-contained HTML file that queries
  Supabase once and embeds the results and charts directly in the file.
- **Charting library**, matched to that choice — see the README's "Bonus:
  choosing a JS charting library" section: Chart.js or Observable
  Plot/ECharts for a fast, CDN-loaded, no-build-step fit that suits
  either output format but especially the HTML artifact; Recharts only if
  the student has chosen the web-app format and you are scaffolding a
  React app around it; D3.js only when a chart needs a shape none of the
  simpler libraries provide out of the box, given how much more code it
  takes to build axes, scales, and shapes by hand.

## Step 4 — Build, then independently re-verify before presenting anything as done

Once the plan, the output format, and the charting library are all
confirmed, build the queries and the chosen output. Before you tell the
student anything is finished:

- Independently re-run every single number and chart shown on the
  dashboard as a fresh, separate query against the live database — do
  not just re-read the query you used to build the dashboard and assume
  it still holds.
- Compare each fresh result to what is actually on screen.
- If everything matches, say so with the actual numbers side by side, not
  a bare "looks correct."
- If anything does not match, report it with both numbers side by side —
  the on-screen value and the freshly queried value — and explain the
  discrepancy in plain language before presenting the dashboard as done.

## Step 5 — Label everything in plain business language

Every chart and every number on the dashboard needs a label a
non-technical reader would understand at a glance — "Active
subscriptions," "MRR by plan tier," "Customers by industry" — never a raw
table name, column name, or query fragment. If you catch yourself about
to label something with a column name, stop and translate it into the
business question it answers instead.

## Step 6 — Guard against this corpus's known gotchas, by default

Do not wait for the student to catch these after the fact — build the
guard in from the start, every time:

- **Historical subscription rows.** `subscriptions.csv` has more rows
  than there are currently-active subscriptions, because a customer who
  upgraded or downgraded leaves a superseded row behind. Any count or sum
  over subscriptions must filter to `status = 'active'`, or take only the
  latest row per customer, before aggregating — never sum or count every
  row unconditionally.
- **Usage-log coverage gap.** Not every customer has a usage-log file. If
  any dashboard metric touches usage data, account for that gap
  explicitly (a visible footnote, or a denominator that reflects only the
  customers with usage data) rather than letting the missing rows
  silently understate or skew the number.
- **Near-duplicate category values.** Breakdown charts by country or
  industry can fragment into near-duplicate categories that are really
  the same value spelled differently. Flag this rather than silently
  merging it — unless the student has explicitly told you to normalize a
  specific set of values, show the fragmentation as it actually exists in
  the data and say so out loud.

## Step 7 — Never expose Supabase credentials to the student

If the chosen output needs a live Supabase connection, read the project
URL and access token from an environment variable or a local config file
you manage yourself — never write them into a file you show the student,
never print them in a message, and never commit them anywhere. This
matches the Lab 4 README's "Known pitfalls" section; if you are ever
unsure whether a file you are about to show the student contains a
credential, check before showing it.

## What never to do under this skill

- Never propose or build a single chart, query, or layout before the
  student has stated their own business questions — and never invent
  that list for them, even if they ask you to.
- Never pick the output format or the charting library on the student's
  behalf; always ask and wait for an explicit answer to both.
- Never present a number or chart as finished without having
  independently re-run it as a fresh query and compared the two values
  side by side.
- Never label a chart or number with a raw table name, column name, or
  query fragment — always translate it into the business question it
  answers.
- Never silently filter out, merge, or explain away a data quirk this
  corpus is known for (superseded subscription rows, the usage-log
  coverage gap, near-duplicate categories) — guard against it up front or
  flag it visibly, never silently and never only after the fact.
- Never write or display a Supabase credential in anything the student
  can see.
