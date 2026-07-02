+++
title = "Building an AI-Tooling Registry for a Whole Org"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-04-21"
categories = [
    "AI",
]
menu = "main"
+++

### "Wait, we have an MCP server for that?"

MCP (Model Context Protocol) gets sold as the "USB-C for AI tools". One standard plug, every model, every tool. And that part is genuinely true. The hard part shows up later.

Last quarter I went looking for how many MCP servers existed across our engineering org. The honest answer was: nobody knew. Some teams had first-party servers. Some had wrapped third-party ones. A few lived in private repos that two people had bookmarked. No source of truth for what existed, what it did, what credentials it needed, or whether it had ever passed a security review.

So our agents did what agents do when they have no map. They hallucinated tools, or somebody hardcoded a half-correct README into a prompt and called it a day.

### The real problem isn't writing a server

Writing one MCP server is a weekend. There are templates, the spec is small, the SDKs are decent.

At org scale the problem is different. It's governance and discovery:

- Where is the server?
- Is it safe to use? Who decided that?
- What version am I plugging into?
- What credentials does it need, and how do I configure my client?

None of that is solved by writing more servers. It's solved by treating MCP as a platform, not a pile of repos. I built three pieces.
### Piece 1: a metadata-only catalog

The instinct here is to build a big registry service with a database. Resist it. You don't need runtime infra to babysit. You need a source of truth that's boring and auditable.

Each MCP server is just a folder with a YAML manifest and a README. The actual source code stays in its own repo. The manifest declares the essentials:

```yaml
name: wiki
description: Read and search the company wiki as MCP tools
version: 1.2.0
remotes:
  - https://example.internal/mcp/wiki
auth:
  required_headers:
    - X-Wiki-Token
```

A static site (Next.js static export) walks every manifest at build time and renders a private catalog page. No server, no DB, no on-call.

Adding a new MCP server is the part people actually feel:

1. Copy the `example` folder.
2. Fill in the manifest and README.
3. Open a PR.

About 90 seconds after merge, it's live in the catalog. Versioning is per-server via git tags like `wiki/v1.2.0`, so one team shipping a new version never blocks another.

Bonus that paid for itself fast: because the manifest declares the required credentials, the catalog can auto-generate the client config and the CLI header flags for you. Copy, paste, connected. No more guessing which header the server wanted.

### The catalog isn't just for MCP servers

Once the pattern worked for MCP servers, I pointed it at everything else that has the same "where is it, and is it safe?" problem. The catalog ended up holding three kinds of artifact, all pushed through the same PR-and-badges flow.

![One registry catalogs three artifact types: MCP servers, plugins, and product context packs, all through one PR-and-badges pipeline into a private catalog](/images/19-registry-types.svg)

- **MCP servers.** Metadata only. The manifest points at a server whose code lives in its own repo.
- **Plugins.** Claude Code plugins that bundle skills, commands, and agents. These are the odd one out: the plugin *is* the artifact, so its content lives in the registry itself and installs straight from the marketplace with a single command. It's how we share reusable skills across the org instead of everyone re-inventing the same `CLAUDE.md` tricks in private.
- **Product context packs.** A small manifest maps a product to its repos, and a nightly job runs repomix to bundle that code into one file. So when someone writes a PRD, or points an agent at "the billing product," they can ground it in the actual current code instead of a stale wiki page.

Same pipeline, same governance, three different payloads. The registry stopped being an MCP list and turned into the org's map of what AI can actually plug into.

### The governance bit: security lives in a separate file

This is the part I'd fight for if someone tried to cut it.

The "is this allowed?" decision must not belong to whoever wrote the manifest. Otherwise every author silently approves their own server, and your catalog becomes a list of things that look official but were never reviewed.

So the badges live in a separate file: `security_reviewed`, `recommended`, `deprecated`. That file is owned by a CODEOWNERS rule, so only the security folks can change it. The author opens the PR for their server. A different, required reviewer flips the badge.

```
# CODEOWNERS
/catalog/badges.yaml   @security-review
```

No silent approvals. The audit trail is just git history. If someone asks "who said this server was safe and when," the answer is a commit, not a Slack thread that scrolled away six months ago.

### Piece 2: a template so nobody starts from zero

Discovery is half the battle. The other half is making the right thing the easy thing.

We shipped a reusable TypeScript template. Run it and you get a new MCP server in minutes: consistent project structure, CI wired up, tests already passing. Same shape every time, which means anyone can read anyone else's server without re-learning a new layout.

This matters more than it sounds. Half the sprawl wasn't malice, it was friction. People built one-off servers because bootstrapping a "proper" one felt like a project. Remove that friction and the weird snowflakes mostly stop appearing.

### Piece 3: a real server, not a toy

A platform with no real consumers is a slide deck. So the first server we put through the whole pipeline was the one I wanted most: the company wiki, exposed to agents as MCP tools.

The tools were the obvious ones:

- `read_wiki_page`
- `search_wiki` (full-text)
- `list_wiki_index`
- `get_page_relationships`

A few decisions that mattered:

- **Auth**: token-based, hashed with SHA256 server-side, one token per client so a single consumer can be revoked without touching the rest. Each token carries an expiry date, though enforcing it on every call was still on my list, if I'm honest.
- **Hosting**: serverless on AWS Lambda, behind a Lambda function URL, deployed with SAM. Nothing to patch between requests.
- **Stateless**: every call is a live query. The answer is always fresh, because there's no cache pretending the wiki hasn't changed.

It got consumed by Claude Code, by a Copilot Studio setup, and by CI agents. And here's the payoff: it killed a huge amount of hallucination. Once an agent could consult the org's actual memory instead of inventing it, "I think the doc says..." turned into "the doc says, and here's the page."

### What I'd tell you if you're about to do this

A few opinions, earned the annoying way:

1. **Metadata first.** Don't force every server into a monorepo or a database. A folder of manifests is searchable, diffable, and free.
2. **Static builds.** No runtime registry to keep alive. The catalog is just HTML that rebuilds on merge.
3. **Separate the security decision from the authorship.** CODEOWNERS on a badges file is the cheapest real governance you'll ever ship.
4. **Ship one server that people actually want.** The platform earns trust through a consumer, not a README.

MCP being the USB-C moment is real. But a USB-C port nobody can find, with no label saying whether it's safe to plug into, is just a hole in the wall. The plumbing is the easy part. The map is the work.

Worth doing properly.
