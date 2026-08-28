# Don't task an AI to collect threat intel until you can say what is being threatened

**Status:** draft for cyaichi.com
**Date:** 2026-08-28
**Related:** Cyaichi threat-intelligence methods (direction, then span-grounded extract)

---

Intelligence is information in the service of a decision about something you actually defend. That was true when the collector was a junior analyst. It is still true when the collector is an agent.

If you skip that sentence, you get a fluent content farm: summaries of every ransomware blog this week, none of which change a detection, a hunt, or a patch.

## The order

```
what we defend, what we already cover
        →  PIRs (what we need to know, by when, for whom)
        →  a harness (context pack, allowlist, schema, envelope)
        →  then collect and extract
        →  hunts / detections / an updated picture of what we defend
```

Collection is not step one. Direction is.

## What you need before collection

1. **A decision owner.** Someone who will act. If no one will, stop.
2. **The defended object.** One paragraph: if this fails, we cannot X.
3. **A slice of the environment, not the CMDB.** Tech *classes* (IdP, OS mix, public API, mail path), not hostnames and account lists.
4. **Coverage.** Which techniques you already detect. Gaps are where PIRs come from.
5. **PIRs.** Specific questions with a due date and a test for “done.”
6. **An envelope.** What the agent may fetch and what it must never do. Default: read-only.
7. **A data rule.** What may enter a model. Default: public sources. Your inventory and telemetry stay out until you have a reason and a gate.

A PIR is not “the ransomware landscape.” It looks like: *If we knew which current techniques against our class of IdP are missing from production detections, we would write or skip those detections this week.*

If you cannot finish “If we knew ____, we would ____,” you are not ready to collect.

## The harness

A system prompt is not a harness. A harness is the context and the constraints in a form the agent cannot talk its way around:

- a **context pack**: sanitized slice + the PIR list + source allowlist + output schema
- **typed tools** that query coverage and PIRs; getters, not dumpers
- **gates in application code**: structured output, catalog checks, no auto-accept
- **budgets**: tokens, time, tool calls
- **pins**: model and ATT&CK versions, not `latest`
- an **audit line**: which pack tasked which run

The agent scores allowlisted sources against PIRs, extracts into objects a human can check, and stops. It does not browse the open web as the source of record. It does not push a block list.

Two design rules from the last two years of LLM application failures:

- The model does not distinguish instructions from data. Treat every report as hostile. Bound what the model can *do*, not just what you *told* it.
- Do not give one agent untrusted input, private environment detail, and a tool that sends or changes state. Drop a leg, or require a human on every such action.

That is why the context pack is a *slice*. “We run an enterprise IdP, Windows endpoints, M365 mail, one public API” is enough to aim collection. The real CMDB stays in your tools. The agent may query classes and coverage. It does not get the building.

## Extraction comes after

Once PIRs and a harness exist, the language work is: pull indicators with parsers, pull TTPs and relationships with a model that must quote the source, store objects with provenance, and draft hunts a human runs. That is a separate method (span-grounded extract into a graph). It is wasted motion without direction.

Prefer hunts and detections that still work when every IP and hash in the report is rotated. Indicators expire. Behaviors last longer. Direction is how you decide *which* behaviors were worth the work.

## “Train it on our data”

Your data is two different things. Mixing them is how you leak the environment and teach the model yesterday’s stack.

**What you run and cover** should not go into weights. It changes. It is sensitive. Weights are hard to revoke and easy to extract. Put it in the harness: the context pack, the tools, short-lived retrieval.

**How your team accepts work** can be used to adapt the model: accepted vs rejected extracts, hunts you actually ran, PIR wording that survived review. Strip hostnames and secrets. Prefer few-shot examples in the harness before any fine-tune. If you later train, train *format and judgment*, not the asset list. Evaluate on whether a PIR got a grounded answer, not on a public IDS leaderboard.

The thing that should learn over time is the world model, the PIR list, and the intel graph — not a silent update to a 7B file you cannot inspect.

## A first pass that fits on one page

Pick one defended object. Write three PIRs with owners and due dates. List the tech classes and the ATT&CK IDs you already cover. Allowlist ten public reports. Let an agent score those reports against the PIRs and extract candidates. You accept or drop them. No fine-tune. No auto-hunt. Log the pack.

If the agent still wanders, the PIRs are slogans or the slice is empty. Fix that before you buy another model.

---

Cyaichi is independent research on AI in threat intelligence, modeling, detection, and response. This post is direction: what must exist before collection. The extract-and-graph method is the next layer.
