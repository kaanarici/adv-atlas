---
name: fund-census
description: Look up public SEC filings on investment advisers and their private funds - Form ADV advisers, funds, owners, executives and service providers (auditors, prime brokers, custodians, administrators, marketers), plus Form D fundraising, Form 13F holdings, Schedule 13D/13G stakes, Forms 4/5 insider transactions, Form N-CEN registered fund operations and Form N-PORT registered fund holdings. Use when the user names an adviser, hedge fund, private equity or venture firm, a CRD number or an 805- fund ID, or asks who manages, owns, audits or custodies a fund, which firms or funds match a filter, who is largest in a category, what a manager reports holding, what changed in recent filings, or how two firms or funds compare.
---

# Fund Census

Fund Census serves public SEC filings through read-only tools: Form ADV, with Form D, Form 13F, Schedules 13D/13G, Forms 4/5, Form N-CEN and Form N-PORT joined by exact identifiers. Use them for any question about registered or exempt reporting investment advisers and their private funds.

## Workflow

1. Reuse known ids from the conversation; otherwise turn names into ids with `search`. Never guess a CRD number, fund ID or provider id. Search takes "exact phrase", -exclude, a OR b and kind:fund (or adviser, person, provider), and the same word syntax works in `screen` text filters. Name filters in `screen` match whole words by prefix, so `name=sequoia` finds Sequoia Capital but `name=quoia` finds nothing.
2. Read one entity with `get_entity` (type `advisers`, `funds`, `people` or `providers`).
3. Pick the tool that matches the question:
   - "Which funds or firms match ..." → `screen`, using the filter grammar in its description (ranges like `gav=1B..`, lists like `type=hedge,pe`, flags `1` or `0`).
   - "Largest", "top", "fastest-growing", "biggest custodian" → `rankings`.
   - "What changed", "new funds", "who switched auditors" → `changes`.
   - "Compare A and B" → `compare` with two to four ids.
   - "Who is connected to", "shares owners with", "related firms" → `get_entity` on the adviser and read `connections`. Each one names up to three links in `via` (shared owner, co-advised fund, umbrella registration or service provider), rarest first. `neighbours` returns the same surroundings as a graph for any adviser, person, fund or provider.
   - "What does a manager hold", "who else holds this CUSIP", "shared positions" → `holdings` or `holdings_overlap`.
   - "Which advisers serve pension plans", "who uses this custodian" → `find_advisers`, then `adviser_business` for one firm.
   - Registered fund costs, securities lending and credit → `registered_operations`.
   - "Which mutual funds or ETFs hold this security", "what does this fund own" → `registered_holdings`. These are quarterly N-PORT snapshots at each report date, not current holdings.
   - "Insider trades", "Form 4 filings by this firm" → `insider_transactions`, or `get_entity` for a profile's latest filings.
   - "What does this mutual fund or ETF hold", "which funds hold this CUSIP or ISIN" → `registered_holdings`. These are quarterly N-PORT snapshots at each fund's report date, about 60 days old when published, valued at market in US dollars; never call them current or monthly.
   - "How is A connected to B?" → `path` with both ids as type:id (for example `advisers:142979`, `people:4664163`). Each hop names its shared links in `via`; if nothing is found, say so and give the reason it returns.
4. Answer from returned data only. If a field is missing, say so.
5. When the user asks to see a graph, call `show_graph` with the known entity type and id. This opens only a compact connection graph; do not fetch or open a full profile for a graph-only request. Use `show` for other requested interactive views, such as `/advisers/160489` or `/screener/funds?type=vc&gav=1B..`. Inline cards are collapsible; profiles start with headline facts and collapsed details. Do not add a card to text-only answers or claim that a tool response proves successful rendering.

## Daily allowance

The server asks the user to sign in the first time. Each tool call counts toward the account's daily allowance (25 a day on the free plan); listing tools and opening the interactive view do not. If a tool result says today's requests are used up, tell the user exactly that, including any link it gives, and do not retry the call. The allowance resets at midnight UTC.

## Reading the numbers

- Regulatory AUM (RAUM) is what the firm manages for all clients. Gross asset value (GAV) is per private fund and includes leverage, so a fund's GAV can exceed the firm's RAUM.
- "Deduplicated" totals count each fund once and drop feeders whose master fund is also reported. "Gross" totals add every report.
- Largest managers of a fund type are ranked by the gross assets of their funds of that type, not by firm-wide AUM.
- Exempt reporting advisers (ERA) file a shorter form and report no RAUM.

## Answer style

- Lead with the answer, then the few numbers that support it, formatted compactly ($1.2B).
- Cite the source as "SEC Form ADV, filed <date>" using the entity's `filed` date or `meta.through`.
- This is public regulatory data, not investment advice. Decline to recommend buying, selling or investing, and offer the facts instead.
