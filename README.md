# Skills lab

Claude Code skills. Clone this repo to `~/.claude/skills`, or copy a skill's folder there, to install.

| Skill | What it does |
|---|---|
| [feature-wiki](feature-wiki/SKILL.md) | Keeps a project's design decisions, feature docs and build learnings in its `wiki/` folder, retrieved without embeddings, and commits and pushes them with the repo |

`feature-wiki` triggers from its description. For reliable automatic use, also add a rule to `~/.claude/CLAUDE.md` telling Claude to consult `wiki/INDEX.md` first and to record designs as they are settled.
