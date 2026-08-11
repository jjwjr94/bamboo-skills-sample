---
name: author-opportunities
description: Turn audit findings, session notes, failed runs, or unresolved repo gaps into a ranked opportunities roadmap. Scan available evidence first, group the remaining decisions into modules, and guide the owner through one question at a time. Use after an audit, retro, meaningful context change, or escaped failure.
version: 0.3
stage: run
emits: [opportunities-roadmap]
consumes: [audit-findings, session-notes, run-outputs, context-diff, prior-roadmap]
---

# Author opportunities

Turn messy findings into attainable improvements and answerable decisions. The default
output is a proposed `opportunities.md` in the current project.

The governing rule:

> State what the project already establishes, then ask only for the unresolved delta.

## Workflow

### 1. Inspect the project

Read the evidence available in the current environment before asking questions:

- project instructions and documentation
- existing roadmaps, issues, and decision records
- relevant skills, configuration, tests, and examples
- audit findings, session notes, run outputs, logs, and tool results
- prior answers, declined questions, and superseded decisions

Keep the scan proportional. Stop when the remaining uncertainty is clearly a human
decision, a missing fact, or more expensive to research than to ask.

### 2. Reassess existing opportunities

When a roadmap already exists, review it before adding anything. For each open item,
choose one verdict and record the reason:

- **Keep**: still unresolved and still useful.
- **Reshape**: valid, but too broad, vague, or assigned to the wrong person.
- **Close**: answered elsewhere or completed.
- **Retire**: no longer relevant, declined, or superseded.

Apply new evidence to related items so one answer does not leave stale sibling questions.
If the existing roadmap is full and untouched, prioritize reshaping or retiring it over
adding more.

### 3. Classify each surviving gap

An opportunity belongs in the roadmap when the intended owner must supply a fact, make a
judgment, resolve a contradiction, or approve a change.

Handle observable implementation failures separately. Broken automation, failed syncs,
connector errors, and product defects are engineering or operational work, not questions
for the project owner. A roadmap entry may mention that work only when an owner decision
is required to unblock it.

### 4. Group and rank

Group related gaps into named modules. Each module must identify:

- what the project already establishes
- the unresolved questions or decisions
- why resolving them matters
- the artifact or behavior that will change
- the supporting evidence, referenced by path, heading, issue, run, or finding

Rank modules using judgment across three factors: impact, observed demand, and the next
step unlocked. State when a ranking is judgment rather than measured evidence.

Deduplicate by the underlying gap and evidence, not by wording. Preserve prior answers and
declines. A recurring question must have an explicit reason and review horizon.

### 5. Draft the roadmap

Create or update `opportunities.md`. If the project already uses `roadmap.md`, update that
instead. Treat the result as a proposal until the owner confirms it.

Use this shape:

```markdown
# Opportunities roadmap

## Existing items
- **Keep | Reshape | Close | Retire:** <item> — <reason>

## <priority>. <module title>

**Why it matters:** <outcome in project language>

**Expected change:** <artifact or behavior that changes>

**Evidence:**
- <path, heading, issue, run, or finding reference>

**Already established:**
- <evidence-backed fact>

**Open questions:**
- [ ] [required] <one answerable question> — Purpose: <what it unlocks>
- [ ] [optional] <one answerable question> — Purpose: <what it improves>

**Priority:** <high | medium | low> — <brief rationale>
```

Point to source material instead of copying findings into the roadmap. Write questions
that can be answered directly. Remove trivia whose answer would change no artifact,
decision, or behavior.

### 6. Guide the Q&A

After drafting, offer to work through the roadmap. Start with the highest-priority
incomplete module unless the owner chooses another.

Before each question, provide only this compact orientation:

```markdown
Module: <title>
Already established: <relevant fact or facts>
Still unresolved: <gap this question closes>
```

Ask one focused question and wait for the answer. Do not present the rest of the module's
questions at the same time.

After each answer:

1. Confirm the captured fact or decision briefly.
2. Update the roadmap and mark the question `answered`, `deferred`, `skipped`, or
   `unavailable`.
3. Update any related questions the answer resolves or invalidates.
4. Ask the next highest-leverage question, or stop when the owner is done.

## Completion criteria

The run is complete when:

- every candidate gap is either resolved from evidence, routed as implementation work,
  or represented once in the roadmap;
- every roadmap module cites evidence and names an expected change;
- existing items have explicit keep, reshape, close, or retire verdicts;
- the owner is shown no more than one unanswered question at a time; and
- no action beyond drafting or updating the roadmap occurs without human confirmation.
