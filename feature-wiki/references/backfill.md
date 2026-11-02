# Backfilling a wiki on an existing project

The goal is an accurate wiki built from what the code does today. A smaller true wiki is worth more than a complete one with guesses in it.

## 1. Check the starting point

```bash
git status --short; git log --oneline | wc -l; git remote -v; ls wiki 2>/dev/null
```

- If `wiki/` is already a feature wiki, backfill only what is missing, and run `check` to find stale pages.
- If `wiki/` exists but is something else, stop and ask.
- If the user named a feature ("backfill auth"), skip to step 3 for that feature only.

## 2. Find the features

Work out what the product does, using the sources that fit the stack:

- `README`, existing docs, `CLAUDE.md`
- User-facing entry points: routes, controllers, screens, navigation graphs, CLI commands, Android activities and manifest entries
- Module and package layout, build modules, feature flags
- Commit history: `git log --format=%s | head -200`, and `git log --format= --name-only | sort | uniq -c | sort -rn | head -30` for the most-changed areas

Group these into capabilities someone would name. Three to fifteen features is typical. Fold helpers into the feature they serve; shared infrastructure only earns its own feature if it has real behavior worth explaining (for example networking or sync). Write the list down, each with its source globs.

If the list is long (over about 15 features) and the user didn't ask for everything, do the most-changed ones first and say which are left.

## 3. Write each feature

For each feature:

1. `$WIKI new <slug> --title "..." --status implemented --sources "..." --pages ...`
2. Read the code it covers, starting from the entry points. Follow the flow rather than reading every file.
3. Mine history for the why, which the code can't tell you:
   ```bash
   git log --format='%h %ad %s' --date=short -- <path> | head -40
   ```
   Open a commit (`git show <hash> --stat`) when its message suggests a real decision, bug or reversal.
4. Check whether the project's Claude sessions cover this feature with `$WIKI sessions`. If they do, follow `references/session-backfill.md`, because chat is the richest source of reasons.
5. Fill in pages following `references/page-format.md`:
   - Each real decision you found gets a design page: `$WIKI design <slug> <topic> --title "..." --status implemented --sources "..."`. A design replaced later is still worth a `superseded` page if the reason could stop someone repeating it.
   - Lasting traps from past bugs go in `$WIKI learn`, or in the troubleshooting section of `operations.md`.
6. Seed `changelog.md` with one entry dated today, "Wiki backfilled from code", plus any significant past changes you found, under their real dates.

Keep to what you verified in code or history. If a reason isn't recorded, write "Rationale not recorded". If you couldn't determine something, list it as an open question in `overview.md` rather than filling the gap with a guess. Don't create a design page for every choice visible in the code: without a known reason, it belongs on a reference page.

Large backfills may use subagents if the user asked for them, one per feature. Run `build` and `publish` once at the end from the main session, so indexes don't conflict.

## 4. Check and publish

```bash
$WIKI build && $WIKI check
```

Then test the result: pick two questions about features you documented and try answering them from the indexes alone, following the Retrieve steps in SKILL.md. If you can't reach the right section from the summaries, improve the summaries, keywords and questions.

Check `source '...' matches no files` warnings too, since they mean a glob is wrong.

Then publish. A backfill describes code that already exists, so it normally goes straight out. If the working tree has unrelated uncommitted edits to documented files, publish stops with `held-uncommitted-code`. A backfill is documenting committed code, so rerun with `--allow-dirty-sources` after checking that the pages describe the committed version and not those edits.

Report to the user: features documented, features left out and why, and any open questions.
