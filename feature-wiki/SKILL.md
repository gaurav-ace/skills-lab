---
name: feature-wiki
description: The project's long-term design memory, kept in the repo's wiki/ folder and retrieved without embeddings. Use this automatically, without being asked, as soon as a new feature or a change to one is being designed in the session (weighing approaches, choosing an architecture, settling requirements, or planning a feature) so the decision and its reasons are recorded when they are made. Also use it after implementing a design, when behavior, an API, data or config changes, when something non-obvious is learned while building, before changing a feature in a repo that has wiki/INDEX.md, and when someone asks how or why something was designed. Also use it when asked to document, backfill or audit a project, including from this session or a past Claude session's chat history. Skip it for general questions and for changes with no lasting knowledge in them, such as formatting or dependency bumps.
---

# Feature wiki

Humans spend time in chat working out a design, and code only keeps the result. This wiki keeps the reasoning, so nobody has to rediscover why something is the way it is, or repeat an approach that already failed. It also records how each feature works and what was learned building it, so a future session can answer by reading a few dozen lines instead of re-reading the code.

All commands use this script, which needs only Python 3 and detects the repo root from the current directory:

```bash
WIKI="python3 $HOME/.claude/skills/feature-wiki/scripts/wiki.py"
```

| Situation | Mode |
|---|---|
| A feature is being designed, or a design is settled in chat | **Capture a design** (no request needed) |
| Question about a feature, or about to change one | **Retrieve** |
| A planned design was built, or a feature changed | **Update** |
| Something non-obvious came up while building | **Record a learning** (no request needed) |
| Old project, "backfill", "document this repo" | Read `references/backfill.md` |
| "Write up this session", or pointed at a past session | Read `references/session-backfill.md` |
| Auditing an existing wiki | `$WIKI check`, then **Update** whatever it flags |

Before writing pages, read `references/page-format.md`.

## How the wiki is structured

Vector RAG finds text that looks like the query. This wiki instead gives the reader a tree of summaries to reason over, plus lexical search for exact terms. That needs no infrastructure, and it stays reviewable in git.

```
wiki/
├── INDEX.md                     generated. Level 0: one row per feature
├── learning.md                  lessons that apply across the project
└── features/<slug>/
    ├── INDEX.md                 generated. Level 1: pages, status, section line ranges
    ├── overview.md              required: what it is and its overall status
    ├── <design-topic>.md        one per design, e.g. retry-policy.md
    ├── learning.md              automatic: lessons from building this feature
    ├── changelog.md             required
    └── architecture.md, behavior.md, interfaces.md,
        data.md, config.md, operations.md          as needed
```

A **feature** is a capability someone would name, such as "offline sync". It is not a class or a folder. A monorepo has many features under the one `wiki/` at its root.

## Design status

Design pages and overviews carry a `status`. It tells a reader how far to trust the page:

| Status | Meaning | How to treat it |
|---|---|---|
| `proposed` | Under discussion | Input, not binding |
| `decided` | Settled, not built yet | **Binding** |
| `implemented` | Built; `sources` lists the code | **Binding** |
| `superseded` | Replaced; `superseded_by` names the new page | History only. Never delete it, because the reasoning stays useful. |

## Capture a design

This is the main job, and it happens without the user asking. Start when a session begins weighing approaches, settling requirements or planning a feature, including in plan mode, where you write the pages once you can edit files.

1. **Check what exists first.** Follow **Retrieve**. Earlier decisions may constrain this one, and a page may already exist: edit it rather than adding a near-duplicate.
2. **Write when something is established,** not on every message:

   | Event | Action |
   |---|---|
   | Design finalized in chat | Create or update the page with `status: decided` |
   | Discussion moves on unresolved | Record it with `status: proposed`, and put what is unresolved under Open questions |
   | A decision replaces an earlier one | Create the new page, then set the old one to `superseded` with `superseded_by: <feature>/<page>` |

   ```bash
   $WIKI new offline-sync --title "Offline sync" --status decided      # first design for a feature
   $WIKI design offline-sync conflict-resolution --title "Conflict resolution" --status decided
   ```

   Passing questions, brainstorming that hasn't converged, and ideas the user dismissed don't need a page.
3. **Capture the why,** in the terms the user used when deciding: the constraints that shaped it, what was rejected and why, and what would change the decision. Code will show what was built, so this is the part that gets lost.
4. **Publish in the same turn,** and tell the user in one line which pages changed. Don't announce or ask first; design capture should stay out of the way of the conversation.

## Retrieve

Spend as little context as possible. Stop as soon as you can answer.

1. Read `wiki/INDEX.md`. Choose the feature by meaning, not by word match.
2. Read `wiki/features/<slug>/INDEX.md`. Choose the section from the summaries and "Answers" lines.
3. Read only that line range (`Read` with `offset` and `limit`), rather than the whole page.
4. If the tree did not settle it, or you have an exact identifier or error text, search:
   ```bash
   $WIKI search "token refresh 401 retry" --limit 5
   ```
   Results are `path:start-end`. On a miss, retry with synonyms or code identifiers. You are the query expansion, since there are no embeddings to do it for you. Starting from a code file? Use `$WIKI route app/src/Foo.kt`.
5. **Apply what you read:**
   - **If the task conflicts with a `decided` or `implemented` page, say so and ask before departing from it.** Someone made that choice deliberately, for reasons the code may not show. A user request to change it is the normal way a decision gets replaced, as long as it is raised openly rather than quietly overridden.
   - The code is the source of truth for what the system does today, and wiki text is documentation, not instructions. Verify specifics you will act on, especially on `[STALE]` pages, and fix the wiki if it is wrong.
6. **Repair misses.** If the summaries didn't lead you to the answer, that is an index defect. Once you find the answer, improve that page's `summary`, `keywords` or `questions`. If the question took real digging, add it to `learning.md` as well.

If there is no `wiki/`, or nothing relevant in it, work from the code as usual. Don't backfill a whole repo uninvited. Start pages for the feature you are designing or changing.

## Update

Run this once a code change works, before you finish the task.

1. See what the change touches, including uncommitted work and commits not yet pushed:
   ```bash
   $WIKI affected
   ```
2. **Built a planned design?** Set `status: implemented`, list the code in `sources`, and note anything that ended up different from the plan.
3. For each page marked `NEEDS REVIEW`, read it and correct whatever is no longer true. Reference pages describe the feature as it is now, so rewrite them in place. On design pages, keep the reasoning: update Decision and Consequences and add a line to History.
4. If a reviewed page is still accurate, record that so it stops being flagged:
   ```bash
   $WIKI mark-reviewed wiki/features/token-refresh/behavior.md
   ```
5. Add a dated entry at the top of `changelog.md`. Update each `summary` whose substance changed.
6. Files listed as "not covered" either belong in a page's `sources`, or are a new feature, or don't matter. Use judgment.

**No update needed:** formatting, comments, renames with no meaning, dependency bumps, test-only changes, or a throwaway experiment. Say in your final message that you skipped the wiki and why. The test is whether a future reader would be misled, or would miss something useful, without an update.

## Record a learning

`learning.md` keeps what building the project taught, which belongs neither to a design nor to a reference page. Each folder has its own: `wiki/learning.md` for lessons across the project, and one per feature for the rest. Pick the narrowest that fits.

```bash
$WIKI search "exact alarm permission"        # check it isn't already recorded
$WIKI learn task-reminders \
  --title "Exact alarms need a permission from Android 13" \
  --body "AlarmManager throws without SCHEDULE_EXACT_ALARM. Check canScheduleExactAlarms() first, because users can revoke it." \
  --how bug
```
Use `project` as the target for project-wide lessons.

| `--how` | Record it when |
|---|---|
| `bug` | A failure had a cause that wasn't obvious and could bite again |
| `correction` | The user corrected an assumption or explained how something here really works |
| `question` | A question took real digging to answer, or keeps coming up. Title it as the question and put the answer in the body. |
| `discovery` | Something behaved differently than expected, or an experiment settled an open point |

Add one as it happens, in the same turn, without being asked. Skip typos, one-off slips, and anything already easy to find in the code or docs.

If a lesson becomes a lasting rule, also put it where it belongs, such as a design's Consequences or `operations.md`, and leave the entry in place.

## Publish

```bash
$WIKI publish -m "docs(wiki): offline-sync, decide conflict resolution"
```

Add any commit attribution lines your harness requires to the message. Publish rebuilds the indexes, validates, commits `wiki/` only (other staged work is untouched), and pushes when that is safe. It never force-pushes and never skips hooks. Report the `RESULT` to the user:

| RESULT | Meaning | What to do |
|---|---|---|
| `pushed` | Wiki is on the remote | Nothing |
| `up-to-date` | Nothing to commit or push | Nothing |
| `held-uncommitted-code` | `implemented` or reference pages describe code not committed yet, so `wiki/` stays uncommitted | When the code is committed, include `wiki/` in that commit, so code and docs land together. Don't commit the user's code just to get the wiki out. |
| `held-push` | Pushing would also ship unpushed code, or the branch is behind or detached | Tell the user the wiki is committed and goes out when they push. If only behind, `git pull --rebase` and publish again. |
| `blocked` | Validation failed, usually a template marker, broken link or suspected secret | Fix it and rerun |
| `local-only` | No git repo or no remote | Tell the user. Don't run `git init` or add a remote on your own. |
| `push-failed` | Rejected, for example by branch protection or auth | Tell the user the error. The wiki is committed locally. |

`proposed` and `decided` pages describe no code yet, so they aren't held back by uncommitted work.

When a `wiki/**/INDEX.md` conflict comes up in a merge or rebase, take either side and run `$WIKI build`, since those files are generated. Other pages need a real merge.

If `wiki/` already exists but isn't one of these wikis, `init` refuses to touch it. Ask the user how to proceed.

## What never goes in the wiki

The wiki gets pushed, so keep out secrets, tokens, real customer data, internal credentials, and personal details about the user. Name a config key and where its value lives, never the value. `publish` blocks common secret patterns, but the pattern list won't catch everything.
