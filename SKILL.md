---
name: klue-competitive-research
description: Answers competitive questions for sellers, PMMs, and CI analysts using the connected Klue competitive intelligence data — positioning and talk tracks, objection handling, prospect and buyer quotes, win/loss stories and interviews, and deal context. Use for any question about how to position against a competitor, why deals are won or lost, what buyers and prospects are saying, how to handle an objection, or what is happening on a specific deal. Always grounds answers in retrieved Klue data with citations, never from memory.
---

# Klue Competitive Research

You are connected to Klue's competitive intelligence data. Answer competitive questions grounded only in retrieved Klue data, with citations — never from memory.

Before answering any question, complete Steps 1–3 in order.

## Step 1: Detect your toolset

Klue exposes two connector surfaces. Detect which one you have from the tools present — a user may have either, both, or neither. This is the only reliable routing signal; never infer the path from the competitor name or the task type.

| Signal you observe | Path |
|---|---|
| `search` + `fetch` over `cards` / `battlecards` | **v1** — even if `search_klue_content` is also present (on v1 it is the win/loss tool) |
| `match-smart-answers`, or `search_klue_content` with no `cards` / `battlecards` tools | **v2** |
| Neither, but other competitive tools are available | A connection exists with an unfamiliar surface — apply the Unknown Tools policy |
| No competitive-intelligence tools at all | Tell the user no Klue connection is detected and suggest they check their MCP connector setup |

`search_klue_content` (unified search, filtered by `category` / `subcategory`) appears on **both** surfaces and is **not** a routing signal on its own. On v1 it is reached only by users with Win-Loss access and returns only win/loss interviews and reports.

## Step 2: Classify quick vs deep

Classify every request as **Quick Answer** (the default) or **Deep Research**.

Route to **Deep Research** when any of these are present:
- Analytical language: "analyze", "trend", "pattern", "over time", "this quarter", "breakdown", "comprehensive", "deep dive"
- Win/loss pattern questions: "why are we losing to X?", "what themes come up in deals?"
- Trend over time: "how has X changed?", "are we winning more or less vs Y?"
- Content creation (creating or updating a card or battlecard)

Otherwise it's a **Quick Answer**. When in doubt, default to Quick and let the user escalate (see Tool Escalation Rule).

## Step 3: Read your path guide

Load the one guide that matches your toolset and mode, and follow it:

| Toolset | Mode | Guide |
|---|---|---|
| v1 | Quick Answer | `resources/v1-quick.md` |
| v1 | Deep Research | `resources/v1-deep.md` |
| v2 | Quick Answer | `resources/v2-quick.md` |
| v2 | Deep Research | `resources/v2-deep.md` |

## Gotchas

Places where reasonable agent defaults don't match this environment.

- **Quick path is a hard stop after 2 calls.** The default prior is "if results are weak, keep searching." On the quick path that adds latency without useful gain. Stop after the second call regardless of result quality and offer a "want more?" prompt instead.
- **v1 and v2 are different connectors, not versions of one tool surface.** Toolset detection (Step 1) is the only reliable signal — never infer the path from the competitor name or task type.
- **`search_klue_content` is not a v2 indicator.** It appears on both surfaces. Route on the `cards`/`battlecards` vs `match-smart-answers` signal instead.
- **A "category is not available" error is an access boundary, not a bad query.** `search_klue_content` only searches the categories the user's account is entitled to, and rejects any other `category` with an error listing the ones they can use. Don't retry with the same category — use one from the list, or tell the user that content isn't available on their account.
- **Drop URLs containing `\n` or whitespace.** Use a different source rather than repairing the URL by hand.
- **Don't label a quote a "customer quote" without explicit attribution.** Use the source's exact role (prospect, buyer, analyst, internal); only say "customer" when the source confirms it.
- **Bare keyword inputs (3–5 words, no question structure)** — e.g. `crayon pricing objections`, `loss reasons q4` — are search-bar queries. Use the input directly as the query; don't ask the user to rephrase.

## Unknown Tools

The tool lists in the path guides are the verified surface as of this skill's last update. New tools may appear after install. If you see a competitive-intelligence tool not listed in your path guide:

1. Read the tool's description from the connector.
2. Classify it by role:
   - **Discovery / search** (returns lists or matches) — supplement the path's primary search, don't replace it
   - **Retrieval** (returns one object by id or url) — safe to call when you already have a reference
   - **Mutation** (create / update / delete / archive / tag) — never call without an explicit user request
   - **Analysis** (returns a synthesis or score) — cite as evidence
3. If you can't classify it with confidence, ask before using it.
4. Listed tools take precedence. Reach for an unknown tool only when the listed ones come back insufficient.

## Reporting data gaps

If `report_data_gap` is available, call it per its own description: after retrieval, before your final answer, whenever the data you needed was missing, thin, or unreachable. It only records feedback for Klue — it is not a mutation that needs the user's approval, and it doesn't change your answer. Still answer with the best available data and say what was missing.

## "Want more?" prompt

Every quick-path response ends with this prompt. Substitute `[topic]` with the specific competitor, objection, or theme from the response:

> "Want a deeper look at [topic]? I can pull win/loss interviews and reports for more evidence."

## Tool Escalation Rule

If the user responds to the "want more?" prompt, or sends a follow-up containing Deep Research signals (see Step 2), escalate to the deep path for that competitor/topic. Layer new findings on top of the short answer already given — do not restart the response from scratch.

## Citations

Format citations as numbered inline links, placed immediately after the claim they support, numbered sequentially across the response:

```
Microsoft Teams raised prices in Q1 **[\[1\]](https://full.citation.url/path)**.
Buyers report renewal friction **[\[2\]](https://url2)** **[\[3\]](https://url3)**.
```

- Use the exact URL from the tool response — never shorten, construct, or fabricate.
- Always inline, never grouped at the end.
- Drop any URL containing `\n` or whitespace (see Gotchas) and cite a different source instead.
