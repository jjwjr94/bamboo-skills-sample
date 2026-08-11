---
name: reflect-session
description: Run a short retro of the current session and update durable memory with what's durable — the agent recaps what happened on its own, checks it against what memory already holds and any open context-gap queue, asks the user only to confirm or fill genuine gaps in realtime, and saves user-confirmed entries through the host's canonical write path. Use when the user runs /reflect-session, asks to wrap up, debrief, or capture what was learned, or wants a checkpoint for anything worth remembering. Skip for one-off saves the user dictates mid-session, connector or tool setup, or performance analysis.
version: 0.1.1
---

# Reflect on the session

> **Provenance:** the host-neutral form of Bamboo's `brain-reflect` system skill (shipped 2026-08-07). The method is unchanged; host-specific mechanics are fenced into `## Host bindings` at the end, following the `author-system-context` precedent. On a host with the Bamboo MCP, use the product's `brain-reflect` directly — it is the canonical implementation.

## Goal

Reflect on the session so nothing durable is lost. The experience, in order: a short retro of what happened, a check of what durable memory already knows, clarifying questions asked right here while context is hot, then user-confirmed saves through the host's canonical write path. A reflection checkpoint, not a questionnaire and not a transcript archive.

## Division of labor

- **"What did this session surface?" is your question to answer.** Build the retro and extract the candidate facts yourself from the visible conversation. Never ask the user what came up, what was learned, or to recap anything — if you cannot see it, it is not in scope.
- **The user answers only what only they can:** confirming an inference, arbitrating a contradiction, supplying a missing material detail (owner, number, effective date, scope), and deciding save / defer / dismiss.

## Boundaries

- Work only from the visible session.
- The session sets the agenda. Ask only about items this session surfaced — never mine an open-questions list for extra things to ask while you have the user's attention.
- Never persist raw transcript text, unconfirmed guesses, or temporary session detail (this week's numbers, one-off task state, scratch work). A durable entry is a distilled fact or rule that will still be true and useful next month.
- The retro may show everything worth showing; the save pipeline asks for **at most 2 save decisions per run**. Facts that belong in the same place ride together as one decision. If more remains, name it and offer another pass — never silently drop anything, never run a long interrogation.
- Ask at most one open-ended clarifying question per turn.
- Every durable-memory write goes through the host's canonical write path (see Host bindings). Never bypass it.
- Deciding to save is the user's call: no durable-memory write without an exact-entry preview and an explicit confirmation. Defer and dismiss never trigger a write. (Flow step 4's gap-queue recording is a documented, narrower exception — see step 4.)
- Never say recorded, captured, updated, or saved from conversational memory alone. Those words require a successful write result from the current turn, and "verified" additionally requires the post-save recall check below.

## What counts — triage every item into one bucket

1. **Durable context** — business, team, workflow, measurement, product, or working-preference facts a newly onboarded teammate should know. The only bucket that feeds the save pipeline.
2. **Retrieval gap** — the information already exists in durable memory but was not found or used this session. Not a save candidate. Tell the user what exists and where; a duplicate save makes memory worse.
3. **Behavior gap** — the session showed an instruction, routing, skill, or tool problem, not missing knowledge. Not a save candidate. Name it in one sentence and point at the improvement path (a skill edit, a config change — or a full audit via `audit-agent` when there are several); do not write knowledge to patch behavior.
4. **Transient** — session-only detail. Drop it silently.

Buckets 2 and 3 are not waste — they are the **improvement opportunities** this skill surfaces. They appear in the wrap-up by name, and when they accumulate, they are the input `audit-agent` turns into a leverage-ranked roadmap.

## Flow

1. **Retro first.** The retro is the first thing the user *sees* — run step 2's checks silently before you speak, so your opening already carries their results; never narrate the lookups. Open with a compact retro of the visible session: what was worked on, what got decided, corrected, or learned. Plain bullets in the user's language — a readout, not a form, and never a transcript. Privately triage every item into a bucket; the buckets shape what happens next and are never shown as jargon. If nothing lands in bucket 1, the retro plus one plain line — nothing from this session needs saving — is the whole output, with anything notable from buckets 2–3 in one line each. Do not invent a candidate to have something to show.
2. **Check durable memory before asking anything.** For each bucket-1 item, search existing memory with a short paraphrase of the fact (not the session's exact words) — see Host bindings for the concrete search surface. If the host keeps an open context-gap queue, read it once. Judge a search hit by content, not by proximity: a passage that does not actually state the fact is a miss. Annotate the retro items:
   - **Already covered** — in memory and consistent: say so, name where, ask nothing.
   - **New** — keep as a save candidate.
   - **Contradicts an existing entry** — keep as an update to that entry; show both versions and ask which is current.
   - **Also an open question in the gap queue** — settle it here like any other item, and record the answer on the queue so it stops being asked.
3. **Settle candidates one at a time.** Give each pursued candidate (max 2 save decisions per run) a one-line distilled statement and an explicit status label — **confirmed** (the user stated or affirmed it this session), **inferred** (derived from what happened; needs their confirmation), **contradicted** (conflicts with an existing entry), **unknown** (a material detail is missing). Ask at most one focused question per turn — none when nothing material is unclear — and only questions from the Division of labor list. An **inferred** candidate must become **confirmed** by the user before it can be saved; never save an inference on your own judgment.
4. **Record on the gap queue** (hosts that have one). When a user-confirmed answer settles a visible open question, record it there in plain language. When the session surfaces a genuinely new open question that is neither a save candidate (bucket 1) nor a behavior gap (bucket 3) — a real unknown worth tracking — you may also add it there, named plainly as new, not settled. Either way this is bookkeeping, separate from the memory save; do it even if the user later defers the save, so nothing surfaced this session is lost.
5. **Confirm where it belongs.** Scope determines the write target — the host's memory layout defines the scopes (see Host bindings). Suggest the likely home with your reason and let the user correct it. Resolve the concrete target now — before previewing — so the preview shows the real destination and merges with anything already there.
6. **Preview the exact entry**: the resolved destination and the complete content you intend to save — distilled: the fact, owner and effective date when given, and the source ("confirmed in session, <date>"), never transcript quotes. Convert relative dates ("last Thursday") to absolute dates; if the conversion is uncertain, ask — it is a material detail. Multiple facts that fit the same destination go in one entry and one preview. Ask: **save / defer / dismiss**. If the user corrects the scope or wording at the preview, rebuild and re-preview before writing anything.
7. **Honor the choice:**
   - **save** → write through the host's canonical path, then verify (below).
   - **defer** → no write. If the answer was recorded on a gap queue in step 4, say it is safe there; otherwise include it, distilled, in the wrap-up so the user can bring it back later.
   - **dismiss** → no write, and leave it out of future prompting this session.
8. **Wrap up** with a short summary: saved (with destination and verification state), queued (where the host routes the write through a change request rather than merging it immediately — give the link, and never imply a human review is guaranteed to happen), recorded on the queue, deferred, dismissed, improvement opportunities noticed (buckets 2–3, named), and the items noticed but not pursued this run — named, so nothing is silently dropped. When buckets 2–3 carried more than a line or two, offer the follow-on: *"want a ranked improvement roadmap? run `audit-agent` over this agent's setup."*

## Saving a confirmed entry

Host-neutral rules; the concrete mechanics per host are in Host bindings.

1. **Check what the target already holds before composing.** If the write surface replaces whole files, always compose the complete file — never just your addition — and carry the existing content forward verbatim, changing only your part.
2. **Additive vs corrective.** For a purely additive fact aimed at a place you did not author, prefer a small new companion entry that references the existing one. Update an existing entry in place only when the change is a real correction (**contradicted**) or the entry is one this flow authored. For a correction: keep what is still true, replace the superseded fact, note the change and its effective date, and save to the **same place**. Never leave two entries that disagree.
3. **Compose as if no reviewer will read the diff** — on some hosts queued changes land unreviewed.
4. **Read the write result before you say anything.** A successful call is not a save — check what actually happened (merged vs queued for review vs rejected) and report it in those words. If the write failed or the path could not be resolved: say plainly the entry cannot be saved from this session, name which step failed, keep the distilled entry in the wrap-up so nothing is lost, and never offer a save decision you cannot honor. Never fake a save.

## Verification

After a save that landed, confirm memory can actually answer with it: search again with a differently-worded question a teammate might ask — not the sentence you just wrote. If the saved fact comes back, report the entry as saved and verified. If it does not, report honestly: the entry was written (name the destination) but is **not yet verified in search** — and suggest re-checking later. Never report full success on an unverified save, and never re-write the entry to force verification.

## Response shape

Opening (the retro):

```
Quick retro on this session — <one line on what we worked on>:

Decided / corrected / learned:
- <item>
- <item>

Against durable memory:
- Already covered: <fact> (<where>)
- Worth saving: 1) <distilled fact> — confirmed  2) <distilled fact> — inferred, needs your confirmation
- Improvement opportunities noticed: <one line each, buckets 2–3>

<the first focused question — or, if nothing is unclear, the scope suggestion and preview>
```

Preview:

```
Here's exactly what I'd save:

Destination: <resolved path or store>
Scope: <scope> (<why>)

<the complete entry, exactly as it will be written>

Save it, defer for later, or dismiss?
```

Wrap-up:

```
Session wrap-up:
- Saved & verified: <fact> → <destination>
- Queued: <fact> → <destination> (<link>) — a change request was opened, not merged; verification waits until it lands
- Recorded on the gap queue: <fact> → <module>
- Deferred: <fact>
- Dismissed: <fact>
- Improvement opportunities: <bucket 2–3 items, named> — run audit-agent for a ranked roadmap.
- Not pursued this run: <the items, named> — run /reflect-session again to review them.
```

## Host bindings

The method above is host-neutral. This section records what satisfying it looks like on specific hosts. Add new hosts beside these subsections without changing the method.

### Bamboo (Claude Cowork / Claude Code with the Bamboo MCP)

Use the product's `brain-reflect` system skill directly — it is the canonical implementation of this method and ships in every Bamboo workspace plugin. Its mechanics (memory search via `recall`/`context`, the Brain Opportunities gap queue, writes through `bamboo knowledge save` with honest `routed` reporting where queued ≠ saved, frontmatter requirements, post-save recall verification) live there; do not restate them here.

### Bare Claude Code (no external memory system)

- **Durable memory =** the project's `CLAUDE.md` (always-loaded rules and facts), a `memory/` directory in the project (topical markdown files, one concern per file, indexed from a short list in `CLAUDE.md` or `MEMORY.md`), and `~/.claude/CLAUDE.md` for facts about the user that hold across projects. Skills in `.claude/skills/` hold procedures, not facts.
- **Search (Flow step 2) =** `Grep`/`Read` over those files with paraphrased terms before asking the user anything. Two searches with different wordings before concluding a fact is absent.
- **Scopes (Flow step 5, bucket 1 only):** project-specific → the project's `CLAUDE.md` or `memory/`; true-across-projects-about-the-user → `~/.claude/CLAUDE.md`.
- **Bucket 3 stays out of the save pipeline** (per What Counts — it is Not a save candidate, full stop). Flow step 5 never resolves a target for it. Name it in the wrap-up as an improvement opportunity; only with the user's separate, explicit confirmation may you propose an edit to the relevant `SKILL.md` — a distinct action from saving, not a scope choice inside Flow step 5 — and prefer flagging over editing when the skill is not yours.
- **Gap queue =** no dedicated system by default; use `memory/open-questions.md` as the lightweight analog — append exactly what step 4 authorizes (settled answers and newly surfaced open questions); read it once per run.
- **Write =** native `Write`/`Edit`. There is no review queue for the durable-memory save (Flow step 6): every such write is immediate, so its preview-then-confirm gate is the *only* gate for that write — never skip it. Flow step 4's gap-queue recording is the one documented exception: it logs an answer the user already gave this turn, so it skips the save preview — but it must still be named to the user in that turn's output, never written silently. "Pending review"/"Queued" states in the wrap-up template do not occur on this host.
- **Verify =** re-`Grep` with the differently-worded question; a hit in the written file counts. Note honestly that always-loaded files (`CLAUDE.md`) only take effect in the *next* session.

## When NOT to use this

- Mid-session one-off saves the user dictates ("remember X") — just save it; no retro needed.
- Working a seeded gap queue as its own task — that is a different flow (on Bamboo, `brain-opportunities`).
- Tool/connector setup, or performance analysis.
- When the session is too short to have surfaced anything — say so rather than inventing candidates.

## Method changelog

- **0.1** (2026-08-10): host-neutral port of Bamboo's `brain-reflect` (shipped 2026-08-07 after two simulated runs against real pilot data plus an adversarial 15-case QA pass, then two rounds of live-run fixes). Method carried unchanged: retro-first, check-before-asking, the four buckets, max-2-save-decisions, one-question-per-turn, preview-then-confirm, honest write reporting, paraphrased-recall verification. Added for the port: the fenced host bindings (per the `author-system-context` precedent), the bare-Claude-Code binding, and the explicit buckets-2/3 → `audit-agent` hand-off. Not yet a cmastack Class A skill — promotion awaits the `reflect` stage in CONTRACT.md (tracked internally).
- **0.1.1** (2026-08-10): correction pass across two rounds of independent adversarial review (a fidelity comparison against the original `brain-reflect`, a live execution run, and a fresh adversarial re-read of the first round's fix). Fixed: (1) the wrap-up template promised "pending review" with a link, reading as a guaranteed human check — the original explicitly forbids promising a human gate for a queued write; renamed to "queued" and restated honestly. (2) Flow step 5 was reachable for bucket-3 items via the bare-CC binding's scope list, contradicting bucket 3's own "Not a save candidate" rule; bucket 3 is now explicitly out of the save pipeline. (3) the bare-CC binding's "the preview-then-confirm gate is the only gate — never skip it" was stated too broadly and contradicted Flow step 4's gap-queue write; narrowed to the durable-memory save specifically, with step 4 named as the documented exception. (4) the Boundaries section's "no write without... preview and confirmation" was unqualified and still read as contradicting step 4 after fix (3) narrowed the binding-level claim; scoped it explicitly to durable-memory writes with a forward pointer to step 4. (5) the bare-CC binding authorized recording "open unknowns" on the gap queue, a capability Flow step 4's method body never granted — the method now explicitly authorizes it, so the binding is no longer claiming an authority the method withholds.
