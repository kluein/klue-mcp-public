---
name: klue-mcp
description: Queries the connected competitive intelligence data source (cards, battlecards, smart answers, agent insights, win/loss interviews and reports) and returns cited answers about competitors, positioning, objections, win/loss patterns, buyer feedback, and battlecard content. Use for any seller, PMM, or CI analyst question that should be grounded in this data rather than memory.
---

# Competitive Research

## Overview

You are connected to a curated competitive intelligence and enablement data source: positioning, objection handling, win/loss interviews, buyer intelligence, and market signals sourced from internal teams and external feeds.

Before answering any research question, complete Steps 1–3 in order.

## Step 1: Detect Your Toolset

Toolset detection is by signal presence. A user may have either toolset, both, or neither. Win/loss tools (`search_win_loss_transcripts`, `list_win_loss_transcripts`, `get_win_loss_transcript`, `search_win_loss_reports`, `list_win_loss_reports`, `get_win_loss_report`) can appear alongside either toolset and are not a routing signal on their own.

| Signal you observe | Path |
|---|---|
| `match-smart-answers` or `search_agent_insights` available | v2 |
| `search` + `fetch` with `cards`/`battlecards` available, no v2 signals | v1 |
| Both v1 and v2 signals present | v2 (newer surface) — apply Unknown Tools policy to anything v1-only |
| Neither, but other competitive / win-loss tools available | A connection exists with an unfamiliar surface. Apply Unknown Tools policy. |
| No competitive-intelligence tools available | Tell the user no connection is detected and suggest they verify their MCP connector setup. |

## Step 2: Classify the Request

Classify every request as **Quick Answer** or **Deep Research**.

**Deep Research** triggers when *any* of the following are present:

Analytical language:
- "analyze", "trend", "deep dive", "comprehensive", "pattern", "over time", "quarter", "breakdown"

Question type:
- Win/loss pattern analysis ("why are we losing to X?", "what themes come up in deals?")
- Trend over time ("how has X changed this year?", "are we winning more or losing more to X?")
- Content creation (creating or updating a card or battlecard)

**Quick Answer** is the default when none of the above signals are present. When in doubt, default to Quick Answer and let the user escalate.

## Step 3: Read Your Path Guide

| Toolset | Mode | Guide |
|---|---|---|
| v1 | Quick Answer | `resources/v1-quick.md` |
| v1 | Deep Research | `resources/v1-deep.md` |
| v2 | Quick Answer | `resources/v2-quick.md` |
| v2 | Deep Research | `resources/v2-deep.md` |

## Gotchas

Places where reasonable agent defaults don't match this environment.

- **Quick path is a hard stop after 2 calls.** The default prior is "if results are weak, keep searching." On the quick path that produces latency without useful gain. Stop after the second call regardless of result quality and surface a "want more?" prompt instead.
- **v1 and v2 are different connectors, not different versions of the same tool surface.** Toolset detection (Step 1) is the only reliable signal — never infer which path to use from the competitor name or task type.
- **Win/loss tools are not a v2 indicator.** They can appear in either v1 or v2 sessions. Use the `cards`/`battlecards` vs `match-smart-answers`/`search_agent_insights` signal for routing instead.
- **Drop URLs containing `\n` or whitespace.** Use a different source rather than fixing the URL by hand.
- **Don't label a quote as a "customer quote" without explicit attribution.** The natural prior is to round "buyer" up to "customer." Use the source's exact role designation (prospect, buyer, analyst, internal team) and only say "customer" when the source confirms it.
- **Bare keyword inputs (3–5 words, no question structure)** — e.g. `enterprise pricing`, `loss reasons q4`, `video integration` — are search-bar style queries. Treat the input directly as the search query. Do not ask the user to rephrase as a full question; they are using chat as a search bar and expect search-like responses.

## Unknown Tools

The tool lists in path guides are the verified surface as of this skill's last update. New tools may appear after install. If you see a competitive-intelligence tool not listed in your path guide:

1. Read the tool's description from the connector.
2. Classify by role:
   - **Discovery / search** (returns lists or matches) — supplement the path's primary search, don't replace it
   - **Retrieval** (returns one object by id or url) — safe to call when you already have a reference
   - **Mutation** (create / update / delete / archive / tag) — never call without an explicit user request
   - **Analysis** (returns a synthesis or score) — cite as evidence, treat like `search_agent_insights`
3. If you can't classify with confidence, ask before using: *"I see a `[tool name]` tool I'm not familiar with — should I try it for `[purpose]`?"*
4. Listed tools take precedence. Reach for an unknown tool only when the listed ones come back insufficient.

## "Want More?" Prompt

Every quick-path response ends with this prompt. Substitute `[topic]` with the specific competitor, objection, or theme from the response:

> "Want a deeper look at [topic]? I can pull win/loss interviews and reports for more evidence."

## Tool Escalation Rule

If the user responds to the "want more?" prompt, or sends a follow-up that contains Deep Research signals (see Step 2), escalate to the deep path for that competitor/topic. Layer new findings on top of the short answer already given — do not restart the response from scratch.

## Citations

Format citations as numbered inline links — keeps responses readable while giving users a direct path to the source. Number sequentially across the full response and place each citation immediately after the relevant statement or quote:

```
Microsoft Teams raised prices in Q1 **[\[1\]](https://full.citation.url/path)**.
Buyers report renewal friction **[\[2\]](https://url2)** **[\[3\]](https://url3)**.
```

Rules:
- Use the exact URL from the tool response — never shorten, construct, or fabricate
- Never group citations at the end of the response — always inline, immediately after the claim
- See Gotchas for handling URLs that contain `\n` or whitespace
