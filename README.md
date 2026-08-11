# Bamboo skills — external preview

Three skills from Bamboo's marketing-agent improvement stack, shared for hands-on testing and method review. Together they cover one arc: **reflect on a working session → capture what's durable in the right place → surface what should improve → turn that into a ranked roadmap.**

Laid out as Claude Code **project skills** (`.claude/skills/<name>/SKILL.md`) — open this repo as a Claude Code project (web or local) and they're auto-discovered, no copying required.

| Skill | What it does | Runnable where |
|---|---|---|
| `reflect-session` | End-of-session retro: recaps what happened on its own, checks what memory already holds, asks only to confirm genuine gaps, saves user-confirmed entries to the right place, and names improvement opportunities. | **Plain Claude Code, today** — see its `## Host bindings` section. (Inside Bamboo it ships as the `brain-reflect` system skill; this is the host-neutral form.) |
| `audit-agent` | Audits the context quality of an existing agent or skill (a `CLAUDE.md`, a `SKILL.md`, a workflow doc) and produces an adversarially-verified, leverage-ranked improvement roadmap. | **Plain Claude Code, today** — it only needs file reads. `improvement-loop-overview.md` beside it is the design context it cites. |
| `author-opportunities` | The judgment layer between "gaps found" and "questions a customer actually answers": assess the open queue first, ask only for the delta memory doesn't hold, group into modules, rank, draft — human confirms before anything ships. | **Read as method.** It consumes artifacts produced by Bamboo's improvement loop, so it doesn't run standalone — but the method (the one rule, the dividing rule, dedup policy) is the point. |

## Try them

**Option A — Claude Code on the web** ([claude.ai/code](https://claude.ai/code)): start a new session and connect this repo (`jjwjr94/bamboo-skills-sample`). The skills load automatically for that project.

**Option B — Local Claude Code:** clone the repo (or use GitHub's **Code → Download ZIP** button above) and open Claude Code in that folder. Same auto-discovery, no manual copying.

**Option C — use them everywhere, not just this project:** copy the `.claude/skills/<name>/` folders you want into `~/.claude/skills/` — same files, just user-scoped instead of project-scoped.

Then:
- At the end of a real working session, run `/reflect-session`.
- Point `audit-agent` at any agent artifact you have — a Claude project's `CLAUDE.md`, a skill file, a written workflow. It works from a single pasted file.

## What we'd love your read on

- Does the method hold up read cold, without our context in the room?
- Where does a skill silently assume something only we would know?
- `reflect-session`: does the retro-first / max-2-saves / preview-then-confirm shape feel right in actual use, or does it get in the way?
- `audit-agent`: run it on something real of yours — is the roadmap it produces something you'd act on?

## Notes

- These are **external-share copies** (2026-08-10). Customer identifiers are anonymized ("the pilot customer"); a few internal repo paths are generalized. Method content is otherwise unchanged; each file carries its version and changelog.
- Some skills reference siblings not included here (`context-audit`, `grill-for-evals`, `author-system-context`) — those names are pointers into the wider library, safe to read past.
- Canonical versions live in Bamboo's repos and evolve there; this repo is a snapshot.
