---
name: author-opportunities
description: Assess the brand's Brain Opportunities queue and author the next attainable bites — re-check every open module against what changed, harvest new gaps from the evidence the loop already produces, and draft grouped, answerable asks a human confirms before anything reaches the customer. Use on a brain structure change, after a batch of opportunity answers lands, or when an escape traces to a knowledge gap. Never on a calendar.
version: 0.1
stage: run
emits: [opportunity-drafts]
consumes: [audit-findings, run-outputs, brain-diff, opportunity-answers, annotations]
---

# Author opportunities

> **External-share copy** (2026-08-10): customer identifiers and person names anonymized. Method content unchanged from v0.1. Consumes artifacts produced by Bamboo's improvement loop — read as method; not runnable standalone.

The producer for the Brain Opportunities queue. `audit-agent` finds gaps; the customer's
`brain-opportunities` skill walks them through answering; **this skill is the judgment in
between** — deciding what is worth asking, in what shape, and just as often deciding what
to *retire*.

The goal, in the owner's words: **give the customer attainable bites — improvements they
can make or questions they can answer.** Not a health score, not a checklist, not a
homework packet.

```
evidence (findings · runs · diffs · answers) → [this skill: assess → harvest → shape → rank → draft] → human confirms → queue → customer answers → B4 fills
```

## The one rule

**State what the brain already has, then ask only for the gap.**

Every one of the seven rows this skill is codified from does this, and it is why they read
as tasks instead of homework:

> *"Brain already defines ROAS, CAC/blended CAC, and LTV at a high level"* → ask only for
> nCAC, MER, LTV:CAC, churn, repeat rate, active subscriber.

> *"The updated growth chart names Morgan as Retention Lead; an older view names Riley.
> Should the older assignment be marked superseded?"*

A generic gap-finder emits *"unit economics: incomplete"* — a score. These emit a question
someone finishes tonight — a task. **That difference is the product.** An opportunity that
asks for something the brain already holds teaches the customer the queue doesn't know her
business, and she stops reading it. This rule costs a corpus read per draft, and it is
never optional.

## What counts as an opportunity — the dividing rule

> **An opportunity is something only the CUSTOMER can resolve. Everything else is
> observability.**

A missing metric definition, an unresolved owner, a contradiction between two pages, a
stale forecast — only she can settle those. A dark drain, a failed skill sync, a connector
500, an ingestion bug — ours, and putting them in her queue trains her to ignore it. One
narrow exception: a *stalled* ingest means she believes we hold a document we don't —
that fact is hers to know, even though the fix is ours.

## Inputs — where opportunities come from

In rough order of value. All but the last exist today; none require new plumbing.

| Input | What it yields |
|---|---|
| **`[GAP]` markers in run outputs** | The agent names its own gaps *while answering a real question* — demand-shaped without a demand read. Measured 2026-08-06: one passing run named five (`Amazon owner, Analytics owner, Promotions owner, numeric budget threshold, formal tie-break rule`), all machine-readable, none harvested |
| **Brain diffs** | A deletion is content, not just a trigger: *"these claims are now unbacked"* is itself an ask. 4 pages deleted 2026-08-07 were exactly one module's corpus |
| **Answers to prior opportunities** | An answer closes a subquestion, can invalidate a sibling, and often surfaces a new gap. Her questions *back* are the highest-quality demand signal available |
| **Contradictions between pages** | A static corpus check, distinct from a diff — the Morgan/Riley row came from one |
| **Annotations** | `eval_labels` + annotation-task dismissal reasons. "Failure but NOT an agent defect" dismissals are often knowledge gaps with no home |
| **Audit findings** | The classic source — sourced 6 of the 7 first rows |
| **Live-run feedback + `ask` answers** | What actually broke or surfaced in her real work |
| **Quiz per-module scores** | Module-granularity coverage, once the batch is complete — never interpret a partial one |
| **Escapes** | If the cause was a knowledge gap, it is an opportunity as well as a golden case |
| *Real customer questions (OTEL)* | *Dark — webhook deleted. Future input, not a blocker* |

## Phase 1 — Assess before you author

**Re-assessment is half the job, and it runs first.** The most valuable run of this skill
may *shorten* the queue.

For every open opportunity:

1. **Still valid?** Did a corpus change invalidate it? (The 4 deleted claims-and-compliance
   pages sit exactly under one open module.)
2. **Already answered elsewhere?** An answer that arrived through a different channel —
   a brain save, an `ask`, a meeting note — closes the subquestion; nothing does that
   automatically.
3. **Declined or ignored?** If it has sat unanswered across multiple touches, that is
   information: the ask may be too big, too vague, or not hers. Propose reshaping or
   retiring it, with the reason.
4. **Sibling effects.** One answer often settles part of another module — mark those,
   don't re-ask them.

⚠ The whole queue sitting untouched is not a reason to produce more. Measured: 7 modules,
~25 subquestions, zero answered in three days. The annotation queue died the same way —
83 rows, all eventually dismissed. **Producing into an undrained queue is how a demand
signal becomes a backlog.**

## Phase 2 — Harvest and shape

For each candidate gap that survives the dividing rule:

- **Run the one rule.** Read the corpus first. What does the brain already hold on this?
  Quote it in the draft. Ask only for the delta.
- **Group into modules.** ~25 loose questions is a wall; the same 25 grouped into named
  modules ("Metrics and source rules", "Owners and approvals") with keyed subquestions is
  workable — the module is the unit the customer engages with, the subquestion is the unit
  of progress. Every subquestion carries `purpose` and `required`, so a partial answer is
  representable and still moves things.
- **Name the artifact.** `expected_artifacts` says what exists when this closes. If
  answering it wouldn't change what the agent does, it is trivia — drop it.
- **Point at evidence by ID, never by restatement.** Findings already live in
  `audit_findings`, `eval_results_v2`, `eval_labels`, `annotation_tasks`, `judgments`,
  `investigations`. **The finding is derived — reference it. The ask is authored — that is
  the only original content this skill produces.** A queue that restates findings is a
  seventh copy of six other tables, and it will drift like every other copy.

## Phase 3 — Dedup: never ask twice, except on purpose

The DB blocks exact re-creation (`unique (org_id, brand_id, opportunity_key)`). That is
necessary and nowhere near sufficient. Before drafting, check:

| Case | Rule |
|---|---|
| Same gap, different wording | Key on the **underlying finding**, not the phrasing. If the evidence IDs overlap an existing row, it is the same opportunity |
| Previously answered | Never resurface. Note: `answered` is per-subquestion — check the parts, not just the parent |
| Asked and declined | Do not re-ask without a reason *she* would accept. Record the decline and why |
| Legitimately recurring | Some questions are *supposed* to return — *"is Scenario 2 still the operating forecast?"* goes stale annually. Mark these explicitly as recurring with a horizon, so their return is a policy, not a bug |

The precedent for getting this wrong is measured: annotation-task creation ignored scope
history and 78% of open tasks pointed at paused cases. **Check history before
creating. Every time.**

## Phase 4 — Rank, and say the ranking is judgment

`gap severity × observed demand × step unlocked` is the frame. **Observed demand is
currently dark** (no usage read), so the ranking is expert judgment — the draft must say
so out loud rather than implying measurement. A criteria gap nobody ever asks about
outranks nothing.

Tiebreak toward: unblocks a workflow step (the delegation ladder) > settles a
contradiction > fills a definition > refreshes recency.

## Phase 5 — Draft, and stop

**The skill drafts; a human confirms before anything reaches the customer.** An
opportunity spends the customer's attention — a bad one costs trust, not just a cycle.
Drafting is judgment; sending is a decision.

Emit per module: `title` · `summary` · `why_it_matters` (customer language, never
rubric language) · `expected_artifacts` · keyed `subquestions` with `purpose` +
`required` · `evidence` as source-row references · `priority` with the one-line judgment
behind it. Plus the assessment verdicts from Phase 1: keep / reshape / retire per open
row, each with its reason.

## Anti-patterns

- **Asking for what the brain already has.** The one rule. Everything else here is
  recoverable; this is how the queue loses the customer.
- **Producing into an undrained queue.** Assessment first, always.
- **Restating findings.** Point at them. The ask is the only authored content.
- **A health score wearing a question mark.** "Improve your metrics coverage (currently
  D+)" is not an opportunity.
- **Ungrouped question lists.** 25 loose questions is a wall; the module is the unit.
- **Re-asking a declined question because the finding fired again.** The finding firing
  is expected — she declined it, and that decision stands until something material changes.
- **Mixing our bugs into her queue.** The dividing rule.
- **Implying the ranking is measured.** It is judgment until a demand read exists.
- **Auto-sending.** Never.

## Hand-off

**You now have:** `opportunity-drafts` — new/reshaped modules plus keep/reshape/retire
verdicts on existing rows, awaiting human confirmation.
**Next is usually:** the human confirms → rows land in `brain_opportunities` → the
customer report / `brain-opportunities` skill carries them to her → answers flow back →
**B4 fills the gaps** and the change-triggered quiz measures the delta.
**Unless assessment dominated:** if the run mostly retired/reshaped, the next step is the
customer touch that drains what remains — not another authoring pass.
**Append to engagement state:**

    stage: opportunities
    emitted: opportunity-drafts -> awaiting confirmation (n new, n reshaped, n retire-proposed)
    open: modules awaiting customer answers; declined items with reasons
    blocked: missing human confirmation, or "nothing"

## When NOT to use this

- **On a calendar.** Triggered only: brain structure change, a batch of answers landing,
  an escape tracing to a knowledge gap, or an audit run completing. `OPERATIONS.md`:
  *"tuned on triggers. Never on a calendar."*
- **When the queue is full and untouched.** Run Phase 1 alone, or nothing. The blocker is
  drainage, not supply.
- **To report our own system failures.** Dividing rule — that's observability.
- **To re-rank for its own sake.** Ranking churn with no new evidence is noise.

## Method changelog

- **v0.1** (2026-08-07) — first written form. **Run count: 0 — but NOT evidence-free.**
  Codified from one full hand-run of the loop this skill automates: the 7 pilot-customer
  modules (authored 2026-08-04, live in prod), the 2026-08-03 customer report that
  rendered them, and the session that audited both. The one rule, the module
  grouping, the subquestion shape, and the evidence-source list are all lifted from those
  artifacts, not invented. The dividing rule and the derived-vs-authored rule come from
  the same session's review of ten candidate input sources. Deliberately deferred, to be
  earned from real runs rather than decided now: the priority formula, the
  absence-state taxonomy, the customer/operator swimlane, and whether active
  modules need a bound beyond grouping — the owner's call is grouping-over-cap, and the
  first drained round tests it. Expect Phase 1's assessment verdicts and Phase 3's
  recurring-question policy to move most; neither has survived contact with a customer.
