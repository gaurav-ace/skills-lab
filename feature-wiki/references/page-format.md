# Writing wiki pages

Without embeddings, a page is only found if its words and structure lead there. The reader, a future AI session, sees the summaries first and decides from those whether to read further. Write for that moment.

## Frontmatter

```yaml
---
title: Token refresh retry policy
feature: token-refresh
type: design
status: implemented
summary: One retry then forced logout, with refresh serialized behind a mutex because the server revokes tokens on first use.
keywords: [jwt, refresh token, 401, TokenAuthenticator, invalid_grant, Mutex]
aliases: [silent login, session renewal, re-auth]
questions:
  - Why does the user get logged out after a 401?
  - How many times is a refresh retried?
  - What happens when two requests refresh at the same time?
sources:
  - app/src/main/java/com/acme/auth/**
related: [networking/overview]
---
```

| Field | Purpose | How to write it well |
|---|---|---|
| `summary` | Shown in the index on every lookup, and decides whether the page gets opened | A retrieval key, not a teaser. **Name the concrete choices**: "Redis cache with 5-min TTL, cursor pagination", not "How reads and caching work". Under 200 characters. |
| `status` | How far to trust the page | Required on `overview` and `design` pages: `proposed`, `decided`, `implemented` or `superseded`. |
| `superseded_by` | Points to the replacement | Required when superseded. A `feature/page` id. |
| `keywords` | Matched by lexical search | Exact identifiers, error text, class and endpoint names: words someone would grep for. |
| `aliases` | Bridges vocabulary gaps | Other names for this: what a user, PM or support ticket would call it. This replaces what embeddings would otherwise do for synonyms. |
| `questions` | Shown in the index and matched by search | Real questions, phrased as someone would ask them. Use block-list form, since questions contain commas. |
| `sources` | Powers staleness detection and `route` | Repo-relative globs for the code this page describes. Required once `implemented`. Keep them narrow: a broad glob flags the page on every unrelated change. On a `decided` page, list planned paths if you know them. |
| `related` | Lets the reader follow connections | `feature/page` ids. Validated, so they can't silently rot. |

`title`, `feature`, `type` and `summary` are required. `reviewed` is written by `mark-reviewed`, so leave it alone. Inline lists split on commas, so any value containing a comma goes in a block list.

## Page types

| Type | Answers | Include |
|---|---|---|
| `overview` (required) | What is this, and where do I start? | Purpose, capabilities, entry points in code, current limits, overall status |
| `design` | What did we decide, and why? | One page per design or separable decision. See below. |
| `architecture` | How is it built? | Components, data flow, dependencies, invariants that must hold |
| `behavior` | What exactly does it do? | Rules, flows, edge cases, failure handling |
| `interfaces` | How do other things use it? | APIs, events, contracts, known consumers |
| `data` | What does it store? | Models, storage, migrations, retention |
| `config` | What can be changed? | Flags and settings, defaults, environments. Key names only, never values. |
| `operations` | How do I run or fix it? | Test commands, and troubleshooting as symptom, cause, fix |
| `learning` (automatic) | What did building this teach us? | Added with `wiki.py learn`, newest first |
| `changelog` (required) | What changed recently, and why? | Newest first, `## YYYY-MM-DD: Short title` |

## Design pages

Name the file after the topic (`retry-policy.md`) and make one per separable decision. These sections are the template, in this order:

```markdown
## Summary
Two or three sentences: what was decided, in plain terms.

## Context
The problem and the constraints that shaped it: volumes, latency, deadlines, team, platform limits.

## Decision
What we do, precisely enough to re-implement it.

## Alternatives considered
- **Option:** why it was rejected.

## Consequences
Trade-offs accepted, known limits, and what would change this decision.

## Open questions
What is still unresolved. "None." is fine.

## History
- 2026-11-02: Chose a mutex over request queueing, which deadlocked on Android 9.
```

The why matters most: code shows what was built, and only this page keeps the reasons and the rejected options. If nothing else was considered, say "None considered" rather than inventing alternatives. Keep the user's own terminology and reasoning when you record a decision, but state it in prose instead of pasting chat text.

Split a design page once it covers two separable decisions.

## Writing sections

The section is what gets retrieved, so each must stand alone.

- **Make headings specific and searchable** on reference pages. `## Refresh retry limit and backoff` gets found; `## Details` doesn't. Heading words carry extra search weight. Design pages keep their template headings, so their summary and keywords must do that work.
- **Keep sections self-contained.** Name the subject in every section. A reader who lands at line 80 never saw line 20, so "it retries three times" has no meaning there.
- **One topic per `##` section, ideally under 60 lines.** A page over 250 lines should be split.
- **Lead with the answer**, then the reasoning.
- **State what the code can't tell you.** Don't paraphrase code line by line, because that goes stale and saves nobody any time.
- **Refer to code by path and symbol, not line number:** `auth/TokenAuthenticator.kt`, `authenticate()`. Line numbers rot quickly.
- **Keep to facts you verified.** If a rationale isn't recorded anywhere, write "Rationale not recorded" rather than inferring a plausible one. A confident wrong answer in a knowledge base costs more than a gap.

## Learning entries

Written by `wiki.py learn`, newest first. Title each entry with the lesson itself, or with the question if it answers one, so the index alone is useful:

```markdown
## 2026-11-02: Why do daily reminders drift by an hour after a clock change?

Daily alarms are scheduled with a fixed 24-hour interval, so they keep the old UTC offset. Reschedule from the local time after each firing.

**How found:** question
```

Record what's true of this project, in terms useful to someone working on it later. A concept is only worth an entry if the answer depends on how this project uses it.
