---
name: audit-agent
description: Audit the context quality of an existing marketing agent or skill (a customer's Cowork project/SKILL.md, a client-authored workflow doc) and produce an adversarially-verified, leverage-ranked list of improvement candidates. Use when reviewing a customer- or client-authored agent/skill, deciding what to improve first, or re-auditing after changes land.
version: 0.6
stage: audit
emits: [audit-roadmap]
consumes: []
---

# Audit an agent's context quality

> **External-share copy** (2026-08-10): customer identifiers anonymized (“the pilot customer/engagement”), internal engagement-home paths generalized to `<engagement-home>`, and the internal design-doc pointer redirected to the bundled excerpt. Method content unchanged from v0.5.

The entry component of **CMAstack**. This file is the **executable method only**.

**The design — the loop, the A/B mechanism, and the surrounding product — lives in an internal design doc; a short excerpt ships beside this file as `improvement-loop-overview.md`.** Read it before acting on findings.

What you need from it to run this audit:

- The subject is **context quality**, not structural completeness. "It has no brain" is known and boring; grade what exists and how it's wired.
- The destination is a **compass, not a conformance checklist**. An agent doesn't have to arrive at all of it to be better than it was, and the audit may conclude the compass is wrong for a specific brand.
- Findings feed an **A/B engine that already exists** — you are producing candidates for it, not designing a test harness.

---

## Step 1 — Extract everything, including what's hidden

**Start from whatever exists.** A whole runtime (Cowork project with CLAUDE.md, connectors) or a single pasted skill file. Don't gate on a rich artifact set. When an artifact type is absent, note the structural gap once and move on; areas referencing a missing artifact collapse into the nearest present one.

If it's a `.docx`/`.pdf`, **don't trust a plain-text export** — check embedded comments and tracked changes (docx: unzip, read `word/comments.xml`, and map each via `commentRangeStart`/`commentRangeEnd` in `word/document.xml`; PDF: annotations). Authors leave the real signal there.

**Mapping a comment to its anchor changes its meaning, not just its location — so it is not optional.** In run 2, a comment reading only *"would need to see this"* was anchored on the exact line `python build_dashboard.py`; that anchor is what turned a vague note into the audit's #1 finding. Another anchored on a byline was correctly demoted to a *direction* item rather than a defect. Unanchored, both are noise.

Capture: trigger conditions (could they misfire — too broad/narrow, bundling unrelated intents); durable facts embedded as prose; workflow steps in order, flagging anything irreversible or externally visible and **where** it sits; hardcoded thresholds; any script the artifact drives and whether its data shape is documented twice (code + prose kept in sync by hand); existing human-review points and where they *actually* occur; references out to tools and data; and **what's already working**.

## Step 2 — Finding areas

1. **CLAUDE.md altitude & load-safety** — is always-loaded context lean and correctly scoped, or bloated with what should be fetched on demand? → route to `context-audit` (detect) + `author-system-context` (fix).
2. **Skill craft** — facts-vs-procedure separation, thresholds as config, trigger precision, sequencing around irreversible actions.
3. **Reference wiring** — are tools and data sources referenced *correctly*: named (not "the report"), live (not stale IDs / retired tools), and actionable by the **runtime's** connectors rather than assuming manual work?
4. **Cross-layer consistency** — no duplicated facts drifting across CLAUDE.md / skill / brain; no contradictions.
5. **Output grounding & verification** — does output cite the number and show its work, or assert conclusions? Are there deterministic checks (reconciliation, anomaly bounds, freshness tags) before external/irreversible actions?
6. **Correction capture** — does a human's correction become a durable improvement, or evaporate?
7. **Operability** — **can the person who owns this agent run and change it without an engineer?** Look for: dependencies only one person can satisfy (a script on someone's laptop, a file in a personal Drive, a credential in one head); tunables that require editing code; steps that assume access the owner doesn't have; anything that breaks when that person is on holiday. A skill nobody but its author can operate is a single point of failure wearing a process costume — and it is usually invisible in the artifact, because the author never had to write down what only they can do.

   ⚠ This area exists because it was **missed** in the first real audit: the pilot customer's weekly-dashboard skill depends on a `build_dashboard.py` living on the operator's laptop. Bamboo can't see it, version it, or ship it — and an ablation run in an isolated dir won't have it either, so the skill can't be A/B-tested at all. The artifact says "the build script is alongside this SKILL.md," which reads fine until you ask *where is it actually*. **Ask that question explicitly; it doesn't surface on its own.** *(Run 2 validated the lens: given only the artifact, it promoted this from "optional cleanup" to finding #1 and traced 4 other findings back to it.)*

   **Escape hatch — emit a question, never infer.** Operability answers usually are **not in the artifact** (can this person edit Python? do they have repo access? who else can run this?). Do NOT guess. Record it in the required *"Not knowable from the artifact"* section (Step 6) as a question for the human. That section is a first-class output: it tells you which areas genuinely require an interview, and it's how the audit and `grill-for-evals` divide labour instead of duplicating it.

For 5/6, distinguish gate shapes — don't prescribe a generic "add a review step":
- **Interactive** (human live): the conversation *is* the review. Fix = sequencing (pause before the irreversible step) + surface check-flags in the reply.
- **Scheduled/async** (nobody watching): needs a real hold-for-review or accept-and-annotate policy, chosen deliberately.

These areas were generalized from one real audit. If a finding fits none, **add an area** rather than force it, and say you did.

## Suppressions — DO NOT flag these

The counterweight to refute-by-default (Step 5). A skeptic loose on a brand artifact full of *deliberate* choices will over-flag, and noise is what makes marketers distrust the tool. Keep a standing list, each entry carrying a precedent so it grows by evidence:

- A threshold or target a human set on purpose — thresholds get tuned; comments rot.
- An interactive gate that happens *in conversation* — a checklist reviewer misreads it as a missing review step.
- Brand-voice quirks that read as errors but are the client's chosen tone.
- Anything already addressed elsewhere in the artifact set.

Add to it whenever a real audit produces a false positive; cite that audit as the precedent.

## Step 3 — Rank into a roadmap (= the test queue)

⚠ **Do NOT use a Tier 1-4 ladder.** An earlier version did, and it collapsed: "Tier 1 = cheap-now" makes tier a *scheduling bucket* while the finding template called it *severity*. Run 2 produced **14 Tier-1 findings out of 25** — accurate on both readings, useless as a priority signal, and unable to distinguish a small bundleable fix from a standalone project. Rank on **Lane + Leverage** instead.

**Lane — what kind of work is this?** This is the distinction the test queue exists to make: some findings bundle into one candidate, others are projects that must stand alone.

| Lane | Meaning |
|---|---|
| **now** | Small, self-contained, bundles with other `now` findings into a single candidate |
| **next** | Real work, still a skill/config edit, but wants its own candidate to stay attributable |
| **spike** | The fix isn't known yet — investigate before estimating |
| **runtime** | Standalone project outside the skill (wiring connectors, getting a data source pulling correctly). **Never bundle these.** They're too big to lump in, and lumping them is what makes a queue useless |

**Leverage — a 3-value scale, not a formula.** ⚠ `impact ÷ effort × confidence` was theatre: nobody can produce that number, so it degrades into a vibe word — exactly what "anchors, not vibes" forbids.

- **high** — fixes something that makes the agent *unusable or untrusted* (it's wrong, or slow enough to abandon, or the human must verify every run)
- **medium** — fixes a real defect the human works around
- **low** — correctness or hygiene with no user-visible consequence today

**Leverage is standalone severity only — never what a finding unlocks.** Whether a fix unlocks other findings is Step 3.5's `unlocks`/`blocked-by` edges, not this field; a foundational fix can be `low` leverage and still run first because it's a gate, and that must never be smuggled into a `high` to justify the ordering.

⚠ **Calibration: default to `medium`, escalate to `high` only with a named, falsifiable reason** (e.g. "this failure reaches the customer verbatim," not "this feels important"). If more than roughly a quarter of findings land `high`, stop and recheck against the definition and against Suppressions — that ratio is itself the collapse Step 3's tier-ladder warning describes. Don't rely on Step 5 to catch what Step 3 should have calibrated.

**A finding whose real leverage depends on a fact not yet confirmed does not average across the possible outcomes.** (A referenced tool's leverage is nil if it resolves, high if it doesn't — one number can't hold both.) Name both leverages in the finding and mark its verdict **PLAUSIBLE**; let Step 5 or the human's answer resolve which applies.

**Effort** keeps its anchors: *cheap* (single-file edit, under an hour) · *moderate* (new function/config/test set, under a day) · *big* (new integration or cross-system plumbing).

⚠ **Calibration: a prose/label/doc edit is `cheap` by default** — escalate only with a named reason. In run 2 all three reviewers flagged the same systematic inflation (doc edits drafted moderate/big). Effort drives sequencing, and sequencing is this audit's whole output, so inflated effort silently corrupts the roadmap.

Rank by **leverage, then effort ascending**, within lane. Present *direction* findings (the compass is wrong for this brand) separately from *problems*.

**Tag each finding `ab_testable: yes | no`.** Eval runs exclude irreversible/external actions, so any finding *about* those actions is invisible to ablation — there is nothing to observe. Findings tagged `no` ship as small human-reviewed diffs, judged directly. **Without this tag the roadmap misrepresents itself as a test queue**, and someone will queue an ablation that can only ever return flat.

**Write each item as a self-contained handoff spec** — the implementer and the next re-auditor are context-less by construction, so "tighten stale references" is not enough. Each carries: the current-state quote, the exact section to change, the convention to match, a machine-checkable done criterion, and out-of-scope boundaries. Tag each fix **mechanical** (safe to bake in) / **taste** (build it, surface at the gate) / **user-challenge** (overrides something the human deliberately chose — never auto-apply; present as *what they chose / what we'd recommend / what we might be missing / the cost if we're wrong*).

## Step 3.5 — Sequence into waves (the dependency pass)

⚠ **Ranking is not sequencing.** Step 3 sorts by leverage-then-effort within lane; that orders findings by *value*, not by what can actually be done first. Run 2 needed a second pass the sorted list can't express — **group the ranked findings into ordered WAVES using dependency reasoning**, because some findings unlock others and some invalidate the measurement of others. This pass was hand-authored in the pilot engagement; it's repeatable method.

The rules that set wave order, most-binding first:

- **(a) Foundation unlocks first.** A finding that *other* findings depend on goes in wave 1 even when its own leverage is modest. In run 2 a single portable-build fix (commit the operator's local script into the versioned skill folder) was the precondition for nine downstream items — nothing script-touching can be tested, shipped, or even seen until the code travels with the skill. Its own leverage was scored on severity alone; it still ran first, because it's a gate.
- **(b) Source-swaps before the accuracy work they'd re-measure.** Connector / data-source changes **re-baseline the eval set** — the numbers legitimately shift when the source changes. So a data-source swap precedes any accuracy fix that would otherwise be measured against a baseline about to move, or you grade the same work twice against two different rulers. In run 2 this put *connectors-before-accuracy*: the manual-upload → connector cutover leads, and the estimate-tightening rides behind it.
- **(c) Within a wave, order by leverage-then-effort** — Step 3's rank, unchanged, applied *inside* each wave once dependencies have set the wave boundaries.
- **(d) The wave IS the trial-lane unit.** A wave of bundled `now` findings is exactly one A/B candidate — **packaging waves = packaging candidates**. Respect the lane rules from Step 3: a `runtime` finding is its own wave-of-one (never bundled), and `ab_testable: no` findings ride in a wave as reviewed diffs, not as ablation arms.
- **(e) Name the dependency edges explicitly.** Where X unlocks Y, say so on the finding (`unlocks: <ids>` / `blocked-by: <id>`) — an edge is machine-checkable trend signal for the next run, the way durable IDs are; an implied edge is not. Findings with no edges fall to their leverage-then-effort slot.

⚠ **Waves are not tiers reborn.** A wave is a *dependency-ordered batch*, not a severity bucket — the exact collapse Step 3 warns about. If your waves sort cleanly by leverage rather than by unlocks-edges, you've re-derived the ranking and skipped the pass; the tell is a wave 1 that isn't a foundation.

## Step 4 — Per-finding regression cases

Every finding's `Failure scenario` **is** its regression case. **These are not the eval bank** — the bank seeds from the customer's real task distribution (interview or manual), never from the artifact, or you hillclimb a distribution the customer doesn't run. See the design doc §7.

Prefer **deterministic** criteria (reconciliation math, freshness tag present, anomaly within bound) over **judged** ones.

## Step 5 — Adversarial review (maker ≠ checker)

**The pass that makes the list trustworthy. Not optional.** The agent that produced the findings must not certify them. Hand the artifact + draft findings to an **independent reviewer** — a fresh sub-agent given only those two things, not the authoring conversation — and have it try to refute each: real and accurately cited (not inferred)? tier/effort/impact honest or inflated? fix correct and *minimal*, or a rewrite in disguise? what did the audit **miss**?

Verdicts: **CONFIRMED** / **ADJUSTED** (lane/fix/scope corrected) / **REFUTED** (dropped, reason kept) / **PLAUSIBLE** (real *if* an artifact nobody has seen says what we think — e.g. a referenced script). CONFIRMED, ADJUSTED and PLAUSIBLE become the roadmap; PLAUSIBLE carries the condition that would settle it.

**Two lenses are the MINIMUM, not an escalation.** ⚠ Corrected after run 2: the refutation lens and the completeness critic returned nearly **disjoint** sets — refutation killed 3 and narrowed 7; completeness *added* 9, and **3 of the top 5 findings came only from the critic**. One lens is half a review.

1. **Refutation lens** — try to kill each draft finding. **Refute by default**: it survives only if it *can't* be refuted. Check against **Suppressions** first so this doesn't manufacture noise from deliberate choices. *(In run 2 this correctly killed two run-1 findings, one of which was a suppression case the author flagged anyway.)*
2. **Completeness critic** — "what did the audit MISS?" (triggers that could misfire, bundled concerns, data-exposure on external outputs, methodology bugs, silent degradation).
3. **Refute the critic's additions too.** ⚠ They arrive *after* the refutation pass and would otherwise ship uncontested. Run them back through lens 1 — in run 2 this downgraded 6 of 9.

**Sub-agents here are for independence, not breadth.** ⚠ This sits beside Orchestration's "don't fan out across one small artifact," and the two read as contradictory. They aren't: **breadth** (many artifacts at once) is unwarranted on a single skill file; **independence** (a checker who doesn't share the author's head) is required regardless of size. Fresh context is the point, not parallelism.

**When two reviewers disagree**, don't let refute-by-default hand it to whichever is more aggressive. Keep it **PLAUSIBLE**, record both positions, and name the evidence that would settle it. An unresolved disagreement is information; a coin-flip is not.

**Maker ≠ checker on the stop** — the audit is done when the independent checker certifies the list is complete, not when the author says so.

## Step 6 — Output

Order: **one-line summary + counts by lane, verdict, and leverage (high/medium/low)** → **Trend** (if a prior audit exists: fixed / still open / new / regressed / **reversed** — a prior finding now refuted) → **"Considered & rejected"** (REFUTED findings, so they aren't re-litigated) → **verified findings, by leverage then effort, within lane** → **Not knowable from the artifact** (required) → **What's already working** → **Eval-set status** → a **Completion Status** line (DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT). **Stamp the report with the artifact revision audited** (SHA or content hash) **AND the method version** (this file's `version:` frontmatter) so the next audit can tell "fixed" from "rewritten underneath us" — and artifact-change from method-change. ⚠ Run 1→2's trend was dominated by re-classification ("the artifact barely moved — the method did"); without the method stamp that signal is unreadable.

**Finding IDs are durable across runs.** A carried-forward finding keeps its ID from the prior run; only genuinely new findings get new IDs (`<run>-<n>` or next-free-integer — pick once per engagement and stick to it). Run 1→2's trend table required a hand-built #1→#7 / #4→#6 mapping; with stable IDs, trend becomes mechanical instead of optional.

**Output home: durable and private, never session scratch.** The report contains real customer data (names, emails, revenue). Write it to the engagement's durable home (e.g. `<engagement-home>/<customer>-audit-run-<n>.md`, or the engagement repo if one exists) — never only to a session scratchpad, which evaporates and forced a "where is the audit?" hunt in the pilot engagement. Client-facing renders (roadmap, implementation table) live beside it, derived from it, and say so.

**"Not knowable from the artifact" is a required section, not a caveat list.** Each entry is a **question for the human**, not a hedge. It is how the audit hands off to `grill-for-evals` cleanly instead of guessing — and in run 2 it was one of the most useful outputs, because 8 of 9 entries traced to a single missing file.

```
### <id>. <one-line finding>                # id is DURABLE across runs — see Step 6
- Area: <1-7>
- Lane: now / next / spike / runtime      # runtime = standalone project, NEVER bundled
- Leverage: high / medium / low
- Effort: cheap / moderate / big
- ab_testable: yes / no                   # no → ships as a reviewed diff, not an ablation
- Routes to: brain / skill / script / config / connector   # which surface the fix edits
- Decision class: mechanical / taste / user-challenge
- Impact: <one sentence — what breaks if unfixed>
- Failure scenario: <concrete inputs/state → wrong output — this IS the regression case>
- Fix (handoff spec): <current-state quote · exact section · convention to match · machine-checkable done criterion · out-of-scope>
- Evidence: <file/line/comment/quote>
- Verdict: CONFIRMED / ADJUSTED (<what changed>) / PLAUSIBLE (<what would settle it>)
```

**`Routes to` uses the compass's surface split** — *facts→brain · procedure→skill · logic→script · numbers/data→config · pulls→connector*. It's what lets the per-item implementation plan (the "how do we actually update the skill" table the client sees) be **derived** from the audit instead of hand-authored after it — in the pilot engagement that translation was done by hand, twice.

### Client renders — DERIVED from the report, never hand-authored

⚠ **Second hand-authoring is the trigger.** The pilot engagement needed two client-facing artifacts on top of the internal report, and both were written by hand — the codify signal. At v0.4 they are **explicit derivation steps**, mechanical from the fields the report already carries. They live **beside** the audit report in the engagement home, and each **states it is derived and from which run + method version** (same stamp as the report) so a stale client render can't outlive the audit it came from.

- **Client summary/roadmap** = three parts, all already in the report: **What's already working** (the protect list, verbatim) → **the waves in client register** (Step 3.5's ordering, rendered in outcome language — no lanes, no leverage/effort tags, no finding IDs, no internal ticket refs; a wave becomes "First — make the build portable (unlocks everything below)", not "Wave 1, `runtime`, leverage-high") → **Not knowable from the artifact** (Step 6), rendered as *a few things to check with you*. It is the internal report with the jargon and the customer PII-adjacent internals stripped, not a fresh document.
- **Per-item implementation table** = one row per verified finding, all columns derived: **Built in** ← `Routes to` (*facts→brain · procedure→skill · logic→script · numbers→config · pulls→connector*); **Ready** ← effort + `ab_testable` rendered as **now-vs-test-first** — a cheap prose/label/doc edit is ✅ **now**; anything that touches numbers, the script, or a data source (i.e. `ab_testable: yes`, or Routes-to script/config/connector) is 🧪 **test first** (A/B where observable, reviewed diff where not); **What changes** ← the handoff spec's plain-language core. The waves from Step 3.5 are the table's section headers, so foundation-unlocks-first survives into what the client reads.

A finding you can't state as concrete inputs→wrong-output probably isn't verifiable. Finding and regression case are authored in one motion.

## Orchestration

**Interview the human** only for **taste** and **user-challenge** calls — which findings matter, what "better" means for this brand, whether a threshold was deliberate. Never interview for facts you can extract from the artifact yourself.

**Sub-agents** are for **breadth or verification**, not mechanical triage. Warranted: auditing many agents in parallel, or the Step-5 adversarial pass. Not warranted: fanning out across one small artifact — a single skill file is an inline read.

## Where to start — data sources

1. **The artifact** — zero setup, always available.
2. **The human's existing outputs** — past deliverables they didn't push back on. Real signal, no instrumentation.
3. **Telemetry** — what it carries is **environmental state that changes; do not trust any restated version of it (including a prior audit's)**. Check the current internal design doc for the capture posture before leaning on it (not included in this bundle). ⚠ Precedent: v0.2 of this file stated "content is dropped to length + sha256" as fact; the capture-everything decision superseded it within days — the exact prose-restated-fact rot this audit flags in customer artifacts (one source per fact applies to us too).
4. **Full runtime interview** — richest, most complex. Deferred.

## Hand-off

**You now have:** `audit-roadmap` at `<engagement-home>/<customer>-audit-run-<n>.md`, with client renders beside it when produced.
**Next is usually:** `grill-for-evals` — because the roadmap's **"Not knowable from the artifact"** section is written as priority questions for the human, and **"What's already working"** tells the interviewer what to protect.
**Unless the audit is BLOCKED or NEEDS_CONTEXT:** resolve the missing artifact or access first.
**Append to engagement state:**

    stage: audit
    emitted: audit-roadmap -> <engagement-home>/<customer>-audit-run-<n>.md
    open: questions from "Not knowable from the artifact" plus any PLAUSIBLE findings needing evidence
    blocked: missing artifact/access needed for the next stage, or "nothing"

## Method changelog

- **0.6** (2026-08-10): Leverage-axis correction. A live test run (21 findings on a planted-defect artifact) drafted 8 of 17 pre-adversarial findings as `high` — reproducing, in miniature, exactly the "accurate on both readings, useless as a priority signal" collapse this file's own Step 3 warns against for tier ladders; only the mandatory Step 5 pass pulled it to 5 of 21, and always in the same direction, which is the signature of a biased anchor, not noise. Root cause: §3.5(a)'s own worked example scored a foundational fix as "low standalone leverage... because it's a gate" — smuggling unlock-value (Step 3.5's job) into a field defined as standalone severity. Fixed: Leverage's definition now explicitly excludes unlock value; added a default-to-`medium`/named-reason-to-escalate calibration rule mirroring the one already proven for Effort; conditional findings (leverage that depends on an unconfirmed fact) now route to PLAUSIBLE instead of forcing an average; Step 6 now reports the leverage distribution alongside lane/verdict counts so drift is visible without a full adversarial pass.
- **0.5** (canonicalization pass, 2026-08-03 — adopted CONTRACT v0.1 frontmatter + hand-off).
- **0.4** (2026-07-23): added Step 3.5 wave/dependency sequencing (foundation-unlocks-first · source-swaps-before-the-accuracy-they-re-baseline · leverage-then-effort within wave · the wave IS the A/B trial-lane bundling unit · explicit `unlocks`/`blocked-by` edges) + the client-render derivation rules (client summary + per-item implementation table derived from the report, never hand-authored — Routes-to → *Built in*, effort+`ab_testable` → now-vs-test-first). Trigger = second hand-authoring in the pilot engagement (the client summary + implementation docs).
- **0.3** (2026-07-23, post-run-2 + client-plan work): method-version stamp on reports; durable finding IDs across runs; `Routes to` surface field (makes the client implementation table derivable); effort-calibration rule (prose edits = cheap by default — all three run-2 reviewers flagged the same inflation); durable+private output home (no session scratch); telemetry facts by reference, not restatement.
- **0.2** (post-run-2): Tier ladder → Lane + Leverage; three-pass review floor (refute → completeness critic → refute the critic); `ab_testable` tag; Area 7 (Operability); artifact-SHA stamping; "Not knowable" as required section.
- **0.1**: initial method (pilot run 1).
