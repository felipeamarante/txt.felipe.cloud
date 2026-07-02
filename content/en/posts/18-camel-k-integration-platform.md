+++
title = "Dragging Legacy Integrations Into Kubernetes With Apache Camel K"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-04-07"
categories = [
    "AWS",
]
menu = "main"
+++

### "Why does one integration live in five different languages?"

Because nobody planned it. It just grew.

I work at an automotive software company, and one corner of our world connects car manufacturers (the OEMs) to dealer management systems. Every manufacturer has its own API patterns, its own auth scheme, its own idea of what a "vehicle order" looks like. Multiply that by decades of "just ship it" and you get integrations scattered across C#/.NET, Java, VB.NET, and yes, a couple of things still running on ASP Classic.

It works. That's the dangerous part. It works just well enough that nobody wants to touch it, and every new OEM partner adds another point-to-point thread to the spaghetti.

This is the unsexy stuff. It's also where a frightening amount of enterprise value is trapped.

### The shape of the mess

Picture it as three problems stacked on top of each other:

- **The edge is chaos.** Every partner hits a different endpoint, with a different contract and a different way of authenticating. There is no single front door.
- **The logic is buried.** The actual integration rules (transform this field, enrich with that lookup, retry on this error, fall back to that queue) live inside compiled code across five languages. You can't see it. You can't diff it. You can barely staff it.
- **The legacy backends are radioactive.** SQL stored procedures, DLL interfaces, things you poke at very carefully and never look directly at.

Any modernization that ignores even one of those layers fails. You can't just put a pretty gateway in front of a mess and call it modern.

### The platform: gateway out front, Camel in the middle, adapters quarantining the legacy

The target architecture splits the problem cleanly across those same three layers. We're not fully there yet (more on that in a moment), but this is the shape we're migrating to.

**The edge: an API gateway.** One standardized HTTPS entry point (we use Gravitee), fronting 130+ endpoints across dozens of domains, with multi-tenant auth. Partners stop caring where the backend lives. They get one front door, one auth model, one place to apply rate limits and policies. That alone removed a whole category of "which endpoint do I call again?" tickets.

**The middle: Apache Camel K.** This is the part I actually want to talk about. Camel K runs Apache Camel routes natively on Kubernetes, defined declaratively in YAML. All those Enterprise Integration Patterns you used to hand-roll (multicast, aggregation, content-based routing, error handling, fallbacks) become readable configuration instead of buried code. The integration logic stops being a mystery in a DLL and becomes a file in git.

**The bottom: a DMS adapter layer.** This is the quarantine. The adapters speak SQL and DLL to the ugly legacy backends on one side, and clean contracts to the cloud platform on the other. The radioactive stuff stays behind glass.

Gateway standardizes the edge. Camel standardizes the middle. Adapters isolate the legacy. Three layers, three jobs.

![Layered integration platform: OEM partners, API gateway, Camel K routes on Kubernetes, DMS adapter layer, legacy backends](/images/18-camel-k-arch.svg)

### What a Camel K route actually looks like

Here's a trimmed example: one endpoint that fans out to several backends in parallel and aggregates the responses. This is the EIP stuff (multicast plus aggregation) that used to be a few hundred lines of threading code.

```yaml
apiVersion: camel.apache.org/v1
kind: Integration
metadata:
  name: vehicle-lookup
spec:
  flows:
    - from:
        uri: "platform-http:/vehicle/{vin}"
        steps:
          - multicast:
              aggregationStrategy: "#mergeStrategy"
              parallelProcessing: true
              steps:
                - to: "direct:oem-inventory"
                - to: "direct:dealer-pricing"
                - to: "direct:warranty-status"
          - marshal:
              json: {}
          - setHeader:
              name: "Content-Type"
              constant: "application/json"
    - from:
        uri: "direct:oem-inventory"
        steps:
          - to: "https://backend.internal/inventory?bridgeEndpoint=true"
          - onException:
              handled: { constant: true }
              steps:
                - setBody:
                    constant: '{"inventory": null, "degraded": true}'
```

You can read that. A new engineer can read that. You can review it in a pull request, you can grep it, you can see exactly what happens when `oem-inventory` falls over (it degrades gracefully instead of taking the whole response down). Compare that to finding the same logic spread across a C# service and a VB.NET helper.

### Strangler fig, not big bang

Here's the rule I will die on: no big-bang cutover. Ever.

Old and new integrations coexist on the same platform. Today most integrations still run on the old runtime (a mix of Azure Functions and logic embedded in the DMS itself); Camel K is where new and freshly-migrated routes land. The gateway routes some traffic to a shiny Camel K route and the rest to the legacy path, and we move endpoints over one at a time. This is the strangler fig pattern, and it's the only sane way to modernize something that's actively making money. If route 37 misbehaves, you flip that one endpoint back. Nobody gets a 3am call because "the migration" failed as a single event. There is no single event.

It's slower. It's also the difference between a modernization that ships and one that becomes a cautionary slide deck.

### The AI bits (kept in proportion)

A few touches that earned their place. I'll keep it short, because the integration architecture is the story, not the AI.

- **An AI migration pipeline.** This is the one that actually pulls weight. A chain of agents scans a legacy integration, triages and assesses what it really does, drafts the equivalent Camel K route, and runs a conformity check against our patterns, with dashboards tracking how far the migration has gotten across the estate. It doesn't migrate anything unattended. It turns "reverse-engineer this undocumented VB.NET integration by hand" into "review what the agent drafted," which is a very different Tuesday.
- **A wiki that stays current.** An LLM keeps our integration docs in sync as routes evolve. The classic failure of integration platforms is that the docs rot the moment the first route changes. Tying doc generation to the route definitions means the wiki actually reflects reality.
- **An MCP server over the DMS endpoints.** We expose the DMS operations as Model Context Protocol tools, so agents can help author and operate integrations against real (sandboxed) endpoints. Combined with Claude Code agents that scaffold Camel routes, generate tests, and write the first draft of the docs, the boring 70% of a new integration gets a lot less boring.

That's it. No "AI transforms everything." It just removes toil.

### How I'd actually start (steal this)

You don't need a cluster or a budget to learn this. I prototyped the whole shape locally first.

```bash
# Local Kubernetes in Docker
kind create cluster --name camel-poc

# Install the Camel K operator via Helm
helm repo add camel-k https://apache.github.io/camel-k/charts/
helm install camel-k camel-k/camel-k --namespace camel-k --create-namespace

# Deploy a route
kamel run vehicle-lookup.yaml
```

Build that fan-out-and-aggregate sample against a few public APIs. It'll deploy. It'll feel great.

Then do the part everyone skips: **day-2 ops.** Because "it deployed" is where the real work starts, not where it ends.

- **Health probes.** Wire up readiness and liveness so Kubernetes actually knows when a route is wedged.
- **Scaling.** Watch how routes behave under load. Decide what scales horizontally and what can't.
- **Metrics.** Get Prometheus scraping the route metrics. You want to see throughput and error rates per integration, not guess.
- **Patching and upgrades.** Practice upgrading the operator. Practice rolling a route with zero dropped messages.
- **Troubleshooting.** Break something on purpose. Read the logs. Learn where Camel hides its errors before production teaches you.

### The honest part

Camel K is not free lunch. There's a real learning curve: the Camel DSL, the EIP vocabulary, the operator's quirks. The first few routes feel slower than just writing the code you already know how to write.

And day-2 ops is the actual job. Anyone can make a route deploy. Running 100+ of them reliably, with observability and graceful degradation and a patching story, is the thing you're really signing up for.

But the payoff is real: integration logic becomes declarative, version-controlled, and observable YAML instead of tribal knowledge buried across five languages. The edge is standardized. The legacy is quarantined. And you migrate one endpoint at a time, so you never bet the business on a single scary night.

Unsexy? Completely. Worth it? Also completely.
