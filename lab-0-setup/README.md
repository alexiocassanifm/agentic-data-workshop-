# Lab 0 - Setup: Accounts and MCP Configuration

This is the only lab of the day that happens mostly outside the agent.
You'll create four accounts, collect the credentials each one gives
you, and then connect your AI coding agent to three of them through
MCP (Model Context Protocol) servers, so it can reach each backend
directly instead of you copying data back and forth. From Lab 1
onward, you direct every backend in natural language and rarely open
a provider's web UI except to double-check what the agent did.

If MCP is new to you: it's a standard way for a coding agent to talk
directly to an external tool - a database, a vector store, an API -
the same way it already talks to files on your disk. Once a server is
"connected", you describe what you want in plain language ("list the
tables in my Supabase project") and the agent picks the right tool on
its own. You never call these tools yourself.

This lab has two parts. **Part 1** creates the four accounts you need
- pure browser work, identical no matter which agent you use. **Part
2** connects your agent to three of those accounts through MCP; pick
the track for the agent you're using (Claude Code, Codex CLI, or
OpenCode) and follow it start to finish. **Known pitfalls**, at the
very bottom, is worth skimming if something isn't working, but you
shouldn't need it to get through the lab.

## Objective

By the end of this lab you will have:

- four accounts created: Supabase, MongoDB Atlas, Qdrant Cloud, and
  Voyage AI;
- three of them - Supabase, MongoDB, and Qdrant - connected to your
  agent through MCP servers, so the agent can query and later populate
  them;
- a Voyage AI API key created and stored safely for Lab 3 (it has no
  MCP server of its own - more on that below);
- confirmation, both in each provider's own web UI and by asking your
  agent, that every backend is reachable and currently empty.

## Duration

75-90 minutes.

## Prerequisites

- One of the three supported coding agents: Claude Code, OpenAI Codex
  CLI, or OpenCode. If you don't have one installed yet, don't worry -
  Part 2 below links to the installer for each.
- Node.js 22.13 or later (needed to run the MongoDB MCP server) and
  `uv` / `uvx` (needed to run the Qdrant MCP server). Part 2 has your
  agent check this for you - you don't need to verify it yourself
  first.
- A web browser and an email address you're willing to use for four
  separate sign-ups.
- A password manager, or at minimum a private notes file kept outside
  this repository, to hold four sets of credentials.
- About 75-90 minutes of uninterrupted time.

This is the first lab of the day, so there is no previous lab to have
completed. If you want a preview of the corpus you'll be working with
from Lab 1 onward, skim `../corpus/README.md` - Lab 0 itself doesn't
touch it.

## Part 1 - Create your four accounts

Every step below is pure browser work, identical no matter which agent
you'll use in Part 2. Do them in any order; if four sign-ups back to
back feels like a lot in one sitting, this is a good place to take a
short break before moving into Part 2 (see Known pitfalls).

By the end of Part 1 you should have four values saved somewhere safe:
a Supabase project reference, a MongoDB Atlas connection string, a
Qdrant Cloud URL plus API key, and a Voyage AI API key.

### Supabase

1. Go to supabase.com, sign up, and create a new project: pick an
   organization, a project name, a database password (store it safely
   even though the MCP setup below won't need it directly), and a
   region. Wait for provisioning to finish, a couple of minutes.
2. Once the project is Active, find its reference ID under Project
   Settings > General > Reference ID - you'll need it in Part 2. It's
   also worth locating Project Settings > Database > Connection string
   now, even though this MCP server won't need it: Lab 3's pgvector
   work talks to Postgres directly and will want it then.

### MongoDB Atlas

3. Go to mongodb.com/cloud/atlas, sign up, and create a free (M0)
   cluster.
4. Under Database Access, create a database user with a username and
   password, and give it "Read and write to any database" access - Lab
   2 needs to create collections and insert documents with it. Store
   the password safely - you'll need to paste it into the connection
   string. If it contains characters like `@`, `:`, `/`, or `%`,
   percent-encode them when you paste the password into the connection
   string below, or just avoid those characters when you set it.
5. Under Network Access, add an entry for your current IP address. For
   a workshop laptop that might move between networks during the day,
   allowing access from anywhere (0.0.0.0/0) is the pragmatic choice -
   see Known pitfalls for what that trade-off costs you.
6. Click Connect on your cluster, choose the driver connection string
   option, and copy the `mongodb+srv://...` string. Replace the
   password placeholder in it with your database user's actual
   password.

### Qdrant Cloud

7. Go to cloud.qdrant.io, sign up, and create a free cluster.
8. From the cluster's Data Access / API Keys area, generate an API key
   and copy it immediately - it's shown once - along with the
   cluster's URL.

### Voyage AI

9. Go to voyageai.com, sign up, and from the dashboard generate a new
   API key. Copy it immediately. There's no MCP server for Voyage AI:
   Lab 3 is where this key gets used, through a small script your
   agent writes at that point. For now, just keep it alongside your
   other three secrets.

## Part 2 - Connect your agent

Account creation is done. Pick the agent you're using and follow that
track start to finish - nothing below needs a browser except the
one-time login your agent opens for Supabase.

### A note on where these live

Each server you add can be stored privately on your machine, or
checked into this repository so the setup travels with it. The tracks
below default to the private option, which is the lower-friction
choice for a one-day workshop. One rule holds regardless of which you
pick: MongoDB's connection string and Qdrant's API key must never
appear as a literal value in a file that's part of this repository -
only as a reference to an environment variable set outside it.

### Check your machine is ready

Whichever agent you install below, once it's running, ask it this
before connecting anything:

**Prompt:**

```
Check whether Node.js 22.13 or later, and uv/uvx, are installed on
this machine. If either is missing, walk me through installing it
for my operating system, then confirm both are ready before we
continue.
```

One rule applies to every "Connect" step in every track below: reload
or restart your agent after it adds a server, before you ask it to
verify that server - a newly added server doesn't show up until you
do.

### A note on skills

Besides the three MCP servers you're about to connect, this repo also
ships five small "skills" under [`.claude/skills/`](../.claude/skills) -
`mcp-health-check`, `schema-proposal`, `pipeline-verify`,
`backend-compare`, and `dashboard-build` - that later labs ask you to
invoke by name (for example, "Run the mcp-health-check skill"). Unlike
the MCP servers above, skills need no setup from you: they're just files
already checked into this repo, and your agent discovers them
automatically the moment it starts inside this folder - there's nothing
to add, connect, or authenticate.

Discovery does differ slightly by tool, which matters for how later
labs' prompts are worded:

- **Claude Code** and **opencode** both read `.claude/skills/` directly,
  so every skill just works once you start either tool inside this
  repository. The one thing to do on Claude Code specifically: the first
  time it opens this folder, accept its one-time workspace trust prompt
  - skills, like any other project file, only activate once you do.
- **Codex CLI** supports the same open skill format, but looks for
  skills under `.agents/skills/` instead, a folder this repo doesn't use
  - so on Codex CLI, whenever a later lab tells you to invoke a skill by
  name, ask your agent directly for the same outcome instead, exactly as
  each lab's "Using Codex CLI or opencode instead" section describes.

Now jump to your track: [Claude Code](#track-claude-code) ·
[Codex CLI](#track-codex-cli) · [OpenCode](#track-opencode)

## Track: Claude Code

**Install**, if you don't already have it:

- macOS / Linux / WSL: `curl -fsSL https://claude.ai/install.sh | bash`
- Windows (PowerShell): `irm https://claude.ai/install.ps1 | iex`
- Homebrew: `brew install --cask claude-code`
- WinGet: `winget install Anthropic.ClaudeCode`
- Full instructions and troubleshooting: code.claude.com/docs/en/overview

Start it with `claude` inside this repository and log in when
prompted.

1. **Connect Supabase.**

   **Prompt:**

   ```
   Add an MCP server named `supabase`, in local scope, using the
   hosted endpoint
   `https://mcp.supabase.com/mcp?project_ref=<YOUR PROJECT REFERENCE>`.
   Don't add any API key or access token - this server logs in
   through a browser.
   ```

2. **Authenticate Supabase.** Reload or restart Claude Code, then
   inside the session type `/mcp`, select `supabase`, and choose
   Authenticate. A browser window opens - log in to Supabase there and
   approve access to the organization that owns your project. No
   personal access token is needed for this flow.

3. **Verify Supabase.**

   **Prompt:**

   ```
   List every table in my Supabase project through the MCP
   connection, and tell me whether the project is empty.
   ```

4. **Connect MongoDB.**

   **Prompt:**

   ```
   Add an MCP server named `MongoDB`, in local scope, running the
   command `npx` with arguments `-y mongodb-mcp-server@latest`. Set
   an environment variable called `MDB_MCP_CONNECTION_STRING` to
   this value: `<YOUR MONGODB ATLAS CONNECTION STRING>`. Make sure
   that value is only ever stored as this environment variable -
   never pass it as a command-line argument, and never write it
   into any file that's part of this repository.
   ```

5. **Verify MongoDB.**

   **Prompt:**

   ```
   List the databases and collections you can currently see in
   MongoDB.
   ```

6. **Connect Qdrant.**

   **Prompt:**

   ```
   Add an MCP server named `qdrant`, in local scope, running the
   command `uvx` with the argument `mcp-server-qdrant`. Set two
   environment variables: `QDRANT_URL` to `<YOUR QDRANT CLOUD
   CLUSTER URL>`, and `QDRANT_API_KEY` to `<YOUR QDRANT CLOUD API
   KEY>`. Don't set a collection name - we'll create the real one in
   Lab 3.
   ```

7. **Verify Qdrant.** This server only exposes a store tool and a find
   tool - there's no "list collections" tool - so the strongest
   confirmation you can get here is that the connection is alive; the
   Qdrant Cloud dashboard is the source of truth for what it contains.

   **Prompt:**

   ```
   Confirm the qdrant MCP server is connected and ready to use.
   ```

8. **Run the full health check.**

   **Prompt:**

   ```
   Run the mcp-health-check skill.
   ```

Once configured, `~/.claude.json` holds a local-scoped entry nested
under your project's own path, shaped like this:

```json
{
  "projects": {
    "/path/to/this/repo": {
      "mcpServers": {
        "supabase": {
          "type": "http",
          "url": "https://mcp.supabase.com/mcp?project_ref=your-project-reference"
        },
        "MongoDB": {
          "command": "npx",
          "args": ["-y", "mongodb-mcp-server@latest"],
          "env": {
            "MDB_MCP_CONNECTION_STRING": "<your MongoDB Atlas connection string>"
          }
        },
        "qdrant": {
          "command": "uvx",
          "args": ["mcp-server-qdrant"],
          "env": {
            "QDRANT_URL": "<your Qdrant Cloud cluster URL>",
            "QDRANT_API_KEY": "<your Qdrant Cloud API key>"
          }
        }
      }
    }
  }
}
```

**Project scope instead?** Ask your agent to add each server with
`project` scope rather than `local`. The three entries then live in a
`.mcp.json` file at the repo root, which is safe to commit - but
MongoDB's connection string and Qdrant's API key must be referenced as
`${MDB_MCP_CONNECTION_STRING}` / `${QDRANT_API_KEY}` rather than typed
in literally, with the real values set as environment variables
outside the repo. Ask your agent to show you what it wrote once it's
done, so you can confirm no literal secret ended up in the file.

## Track: Codex CLI

**Install**, if you don't already have it:

- macOS / Linux: `curl -fsSL https://chatgpt.com/codex/install.sh | sh`
- Windows (PowerShell):
  `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`
- npm: `npm install -g @openai/codex`
- Homebrew: `brew install --cask codex`
- Full instructions: developers.openai.com/codex/quickstart

Start it with `codex` inside this repository and sign in when
prompted.

A Codex-specific note before you start: adding a server through Codex
always writes into your personal `~/.codex/config.toml`, not into this
repository - there's no scope choice to make. If you want a
project-shared setup instead, ask your agent to hand-create or edit a
`.codex/config.toml` file at the repo root, using the same
`[mcp_servers.<name>]` shape shown below; Codex only loads a
project-level config once you've marked this folder as trusted.

1. **Connect Supabase.**

   **Prompt:**

   ```
   Add an MCP server named `supabase` for the hosted endpoint
   `https://mcp.supabase.com/mcp?project_ref=<YOUR PROJECT REFERENCE>`.
   Don't add any API key or bearer token - this server logs in
   through a browser.
   ```

2. **Authenticate Supabase.** Codex detects that this server supports
   OAuth and normally starts the login flow automatically as part of
   adding it, opening a browser window - log in there and approve
   access. If it doesn't prompt automatically, ask your agent to
   authenticate the `supabase` server explicitly.

3. **Verify Supabase.**

   **Prompt:**

   ```
   List every table in my Supabase project through the MCP
   connection, and tell me whether the project is empty.
   ```

4. **Connect MongoDB.**

   **Prompt:**

   ```
   Add an MCP server named `MongoDB`, running the command `npx`
   with arguments `-y mongodb-mcp-server@latest`. Set an environment
   variable called `MDB_MCP_CONNECTION_STRING` to this value: `<YOUR
   MONGODB ATLAS CONNECTION STRING>`. Make sure that value is only
   ever stored as this environment variable - never pass it as a
   command-line argument, and never write it into any file that's
   part of this repository.
   ```

5. **Verify MongoDB.**

   **Prompt:**

   ```
   List the databases and collections you can currently see in
   MongoDB.
   ```

6. **Connect Qdrant.**

   **Prompt:**

   ```
   Add an MCP server named `qdrant`, running the command `uvx` with
   the argument `mcp-server-qdrant`. Set two environment variables:
   `QDRANT_URL` to `<YOUR QDRANT CLOUD CLUSTER URL>`, and
   `QDRANT_API_KEY` to `<YOUR QDRANT CLOUD API KEY>`. Don't set a
   collection name - we'll create the real one in Lab 3.
   ```

7. **Verify Qdrant.**

   **Prompt:**

   ```
   Confirm the qdrant MCP server is connected and ready to use.
   ```

8. **Run the health check.** Codex CLI supports skills, but it looks for
   them under `.agents/skills/`, a path this repo doesn't use — so it
   won't discover the `mcp-health-check` skill here. Ask directly instead.

   **Prompt:**

   ```
   Check my Supabase, MongoDB, and Qdrant connections and tell me,
   for each one, whether it's connected and what it currently
   sees.
   ```

Once configured, `~/.codex/config.toml` holds entries shaped like
this:

```toml
[mcp_servers.supabase]
url = "https://mcp.supabase.com/mcp?project_ref=your-project-reference"

[mcp_servers.MongoDB]
command = "npx"
args = ["-y", "mongodb-mcp-server@latest"]
[mcp_servers.MongoDB.env]
MDB_MCP_CONNECTION_STRING = "<your MongoDB Atlas connection string>"

[mcp_servers.qdrant]
command = "uvx"
args = ["mcp-server-qdrant"]
[mcp_servers.qdrant.env]
QDRANT_URL = "<your Qdrant Cloud cluster URL>"
QDRANT_API_KEY = "<your Qdrant Cloud API key>"
```

Verify any time with `codex mcp list` from the shell, or `/mcp` inside
a session.

## Track: OpenCode

**Install**, if you don't already have it:

- Install script: `curl -fsSL https://opencode.ai/install | bash`
- npm: `npm install -g opencode-ai`
- Homebrew: `brew install anomalyco/tap/opencode`
- Windows: `scoop install opencode` or `choco install opencode`
- Download page and full instructions: opencode.ai/download,
  opencode.ai/docs

Start it with `opencode` inside this repository and sign in when
prompted.

1. **Connect Supabase.**

   **Prompt:**

   ```
   Add an MCP server named `supabase` to my global OpenCode config,
   as a remote server for the hosted endpoint
   `https://mcp.supabase.com/mcp?project_ref=<YOUR PROJECT REFERENCE>`.
   Don't set any headers or API key - this server logs in through a
   browser.
   ```

2. **Authenticate Supabase.** OpenCode detects that this server needs
   authentication and normally offers to start the OAuth flow the
   first time you use it. If it doesn't prompt automatically, ask your
   agent to authenticate the `supabase` server explicitly - a browser
   window opens; log in there and approve access.

3. **Verify Supabase.**

   **Prompt:**

   ```
   List every table in my Supabase project through the MCP
   connection, and tell me whether the project is empty.
   ```

4. **Connect MongoDB.**

   **Prompt:**

   ```
   Add an MCP server named `MongoDB` to my global OpenCode config,
   as a local server running the command `npx` with arguments `-y
   mongodb-mcp-server@latest`. Set an environment variable called
   `MDB_MCP_CONNECTION_STRING` to this value: `<YOUR MONGODB ATLAS
   CONNECTION STRING>`. Make sure that value is only ever stored as
   this environment variable - never pass it as a command-line
   argument, and never write it into any file that's part of this
   repository.
   ```

5. **Verify MongoDB.**

   **Prompt:**

   ```
   List the databases and collections you can currently see in
   MongoDB.
   ```

6. **Connect Qdrant.**

   **Prompt:**

   ```
   Add an MCP server named `qdrant` to my global OpenCode config, as
   a local server running the command `uvx` with the argument
   `mcp-server-qdrant`. Set two environment variables: `QDRANT_URL`
   to `<YOUR QDRANT CLOUD CLUSTER URL>`, and `QDRANT_API_KEY` to
   `<YOUR QDRANT CLOUD API KEY>`. Don't set a collection name - we'll
   create the real one in Lab 3.
   ```

7. **Verify Qdrant.**

   **Prompt:**

   ```
   Confirm the qdrant MCP server is connected and ready to use.
   ```

8. **Run the health check.** OpenCode reads skills straight from
   `.claude/skills/`, so it already sees this repo's `mcp-health-check`
   skill — ask for it by name, the same way you would in Claude Code.

   **Prompt:**

   ```
   Run the mcp-health-check skill.
   ```

Once configured, your global config at
`~/.config/opencode/opencode.json` holds entries shaped like this:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "supabase": {
      "type": "remote",
      "url": "https://mcp.supabase.com/mcp?project_ref=your-project-reference"
    },
    "MongoDB": {
      "type": "local",
      "command": ["npx", "-y", "mongodb-mcp-server@latest"],
      "environment": {
        "MDB_MCP_CONNECTION_STRING": "<your MongoDB Atlas connection string>"
      }
    },
    "qdrant": {
      "type": "local",
      "command": ["uvx", "mcp-server-qdrant"],
      "environment": {
        "QDRANT_URL": "<your Qdrant Cloud cluster URL>",
        "QDRANT_API_KEY": "<your Qdrant Cloud API key>"
      }
    }
  }
}
```

**Project scope instead?** Ask your agent to add the same entries to
an `opencode.json` file at the repo root instead of the global config
- it takes precedence over the global file and is safe to commit,
under the same rule as above: no literal MongoDB connection string or
Qdrant API key in that file, only environment-variable references.

Verify any time with `opencode mcp list`. If Supabase specifically
looks connected but authentication isn't behaving as expected,
`opencode mcp debug supabase` diagnoses OAuth connections in
particular - it isn't a general MongoDB/Qdrant health check.

## Final verification

Regardless of which track you followed, confirm each backend in its
own web UI too:

- Supabase dashboard: project status is "Active"; Table Editor shows
  no tables.
- MongoDB Atlas: cluster status is healthy; Database > Collections
  shows no collections of your own (only the system databases MongoDB
  creates automatically).
- Qdrant Cloud: cluster dashboard shows zero collections and a healthy
  status.
- Voyage AI dashboard: your new API key is listed, with no usage yet.

None of the three should show any data yet - if one does, you're
either looking at the wrong project/cluster, or something from a
previous attempt at this lab is still sitting there.

## Known pitfalls

- **MongoDB Atlas network access is the single most common stall.**
  Without your current IP (or 0.0.0.0/0) added under Network Access,
  the connection simply hangs or times out, with no clear error at the
  MCP level. The trade-off with 0.0.0.0/0: it opens the cluster to
  connections from any address, so the username and password in your
  connection string become the only thing standing between your data
  and the internet - fine for a throwaway workshop cluster you'll tear
  down afterward, not something to carry over into a real project.
- **Four different secrets, four different handling rules.** Supabase's
  login is OAuth, so there's no key to store for it at all. MongoDB's
  connection string, Qdrant's API key, and Voyage's API key are all
  plain secrets you copy once and must store safely. Never let any of
  them end up in a file that's part of this repository, and never type
  one as a bare command-line argument - it can end up saved in your
  shell history.
- **Four sign-ups back to back is genuinely tiring.** If you don't get
  through Part 1 in one sitting, that's fine - stop after any account
  and pick back up later; Part 2 doesn't depend on doing Part 1 in a
  single stretch.
- **Config changes don't take effect until you reload or restart your
  agent.** If a newly added server doesn't show up, that's usually
  why.
- **Codex's server-add step always targets your personal config, never
  this repository.** There's no scope choice - if you want the setup
  checked in, you (or your agent) have to hand-edit a project
  `.codex/config.toml` instead.
- **Supabase's recommended setup is OAuth-only - no connection string,
  no personal access token.** Older guides describing a
  personal-access-token flow are describing a legacy method you don't
  need here. If what you see doesn't match this README, check the
  current docs at supabase.com/docs/guides/ai-tools/mcp.
- **The Qdrant MCP server can't list or count anything.** It only
  exposes a store tool and a find tool, so don't expect your agent to
  "list collections" the way it can for Supabase and MongoDB. Treat
  the Qdrant Cloud dashboard as the source of truth for "is it empty
  and ready."
- **Free tiers idle out.** An inactive Supabase project can pause
  after about a week, and an idle Qdrant free cluster can suspend on a
  similar timescale. If you come back to redo this workshop at home
  later and a connection that used to work suddenly doesn't, check
  whether the project or cluster needs to be resumed in its web UI
  before you start troubleshooting the MCP configuration itself.
- **Keep manual tool-call approval switched on** in your agent for the
  rest of the workshop. Supabase's own documentation notes that data
  returned from a query can contain text designed to redirect the
  agent - approval prompts are your safety net. None of these backends
  should ever be pointed at real production data.
- **These MCP servers are young and still changing.** They're updated
  frequently even past their first stable release; if a tool name,
  flag, or exact config field doesn't match what's described
  here, check the current docs linked above - the overall workflow
  will still hold even if a detail has moved.
