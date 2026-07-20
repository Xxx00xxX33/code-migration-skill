---
name: ai-code-migration
description: >
 Use when planning, executing, reviewing, or rescuing large AI-assisted code migrations and refactors: language ports, framework upgrades, architecture-preserving rewrites, or parity-driven modernization. Covers judge/harness creation, rulebooks, dependency maps, gap inventories, stress tests, mechanical queues, adversarial review, compile/smoke/parity loops, and cost-aware model orchestration. Do not use for small one-file refactors or routine bug fixes.
version: 1.0.0
author: Lucien
license: MIT
metadata:
 hermes:
 tags: [code-migration, refactoring, ai-agents, parity-testing, rulebook, modernization]
 source: https://claude.com/blog/ai-code-migration
---

# AI Code Migration Skill

## Source and Core Insight

This skill distills Anthropic's article **How Anthropic runs large-scale code migrations with Claude Code** (2026-07-16): https://claude.com/blog/ai-code-migration

Core principle: **do not hand-fix migrated code one failure at a time; fix the process loop that produced the code.** Repeated failures become rulebook updates, queue items, or validator improvements.

## When to Use

Use for migrations where correctness depends on many coordinated changes:

- Language ports, e.g. Python → TypeScript, Zig/C/C++ → Rust.
- Framework/runtime migrations with broad behavior parity requirements.
- Architecture-preserving rewrites where source and target can be compared file-by-file.
- Redesign migrations where the old system is the behavioral oracle.
- Large refactors that need subagents, phase gates, objective verification, and resumable queues.

Do **not** use for small edits where a direct patch and narrow test is cheaper than creating migration infrastructure.

## Success Contract

Before implementation, define the business case, exit judge, parity boundary, rollback policy, and cost posture. High-volume implementers may use smaller models; strongest models should write rules, review, and resolve ambiguity.

Failure signals:

- No objective judge or parity harness exists.
- Review findings cite taste rather than a written rule.
- Agents repeatedly fix the same symptom without updating the rulebook.
- The queue depends on human memory instead of disk/build/test state.
- Expensive build/test operations run concurrently and fight each other.

## Phase 0 — Build the Judge First

A migration without a judge has no exit condition.

1. Categorize existing tests into portable external tests and internal tests that will not survive the port.
2. Rewrite portable tests into cross-codebase assertions.
3. Use adversarial reviewers to ensure rewritten tests did not weaken assertions.
4. Validate the judge against original code and deliberately broken code.
5. If no portable suite exists, build a parity harness from real user scenarios and diff source vs target outputs.

Acceptance: the judge catches known breakage and can compare both systems fairly.

## Phase 1 — Create Rulebook, Dependency Map, and Gap Inventory

Order matters: **rulebook first, gap inventory second**. The inventory captures places where rulebook defaults are insufficient.

- Structure-preserving migration: rulebook is mostly lookup tables for types, APIs, idioms, file layout, error handling, ownership, async behavior, etc.
- Redesign migration: rulebook becomes a design document with contracts, boundaries, and intentional differences.

Create deterministic scripts to map dependency order and batching boundaries. Identify leaf units, high-fanout/shared units, and files that must stay in the same batch.

List source-language assumptions that the target language/framework requires explicitly: ownership/lifetimes, explicit interfaces/contracts, module boundaries, static types, schemas, or config.

Acceptance: rulebook, dependency map, and gap inventory are audited together; gaps are defined as rulebook defaults that are not enough.

## Phase 2 — Stress-Test the Rules, Then Throw Away the Output

For structure-preserving migrations, have one agent translate representative files using the rulebook, another translate like a senior target-language engineer, and a third diff outputs to propose missing rules. For redesign migrations, attack the design document with adversarial reviewers and run a disposable end-to-end attempt.

Keep lessons and rule patches; delete generated files. The goal is to refine the loop, not make incremental progress.

## Phase 3 — Translate Everything with a Mechanical Queue

Use an implement → review → fix loop.

1. Rebuild the work queue from disk every run.
2. Slice pending work into dependency-aware batches.
3. Fan out implementer agents and require complete batches.
4. Mark uncertainty as `TODO(port): <reason>`.
5. Run independent adversarial reviewers in separate contexts.
6. Route reviewer disagreement to a tie-breaker.
7. Repeated findings update the rulebook, then affected batches are regenerated.

Key rule: **do not hand-patch around a broken rule**. Amend the rule and regenerate files the rule touched.

## Phase 4 — Compile / Typecheck Loop

Choose where the compiler belongs based on cost:

- Fast compiler/typechecker: run inside every implementer or fixer loop.
- Slow compiler/build: run centrally through an orchestrator or build daemon.

Convert compiler errors into a queue, classify systemic clusters, fan out fixer agents, review fixes, rebuild, and repeat. If one error class repeats, fix the rulebook or dependency-boundary logic.

## Phase 5 — Smoke-Test Runtime Behavior

Run minimal executable scenarios. Group failures by root cause, not by file. Assign each cluster to a fixer agent, review fixes adversarially, rerun smoke tests, and move repeated root causes upstream into rules, harnesses, or boundary decisions.

## Phase 6 — Match Behavior Against the Original

The original codebase is ground truth unless an intentional difference is documented.

1. Run the portable judge or parity harness against both systems.
2. Shard the test/parity suite if needed.
3. Assign each failure to a fixer agent that inspects both codebases and explains the delta.
4. Use adversarial reviewers to check the fix and ensure parity is not weakened.
5. Serialize expensive rebuilds through one build daemon.
6. If many tests fail the same way, amend the generating rule and regenerate only touched files.

If no full suite exists, create real-world command/API scenarios and diff outputs. Then let agents design additional end-to-end tests to catch paper cuts.

## Review Model

Use adversarial review and mechanical verification together:

- Mechanical verifier: compiler, tests, diff harness, smoke scripts, static analyzers.
- Adversarial reviewers: check rule compliance, weakened assertions, hidden drift, unsafe shortcuts, and repeated patterns.
- Strongest model: rulebook authoring, reviewer/tie-breaker, systemic diagnosis.
- Smaller/cheaper models: high-volume translation and routine fixer batches.

Reviewer output should include finding, cited rule, evidence, scope, and recommended action.

## Mechanical Queue Design

A good queue is resumable and reproducible:

- Source of truth is disk/build/test state, not a hand-edited spreadsheet.
- Each item has input path, output path, rulebook version, status, last verifier result, and owning batch.
- Rebuild the queue each run from current files and verifier outputs.
- Store logs/artifacts so future agents can resume without asking the user.

Statuses: `pending`, `translated`, `compile_failed`, `smoke_failed`, `parity_failed`, `accepted`, `blocked`.

## Prompt Skeletons

### Migration Contract Prompt

```text
We are migrating <source system> from <source tech> to <target tech>.
Original behavior is the ground truth unless listed under Intentional Differences.
Create a migration contract with: business case, parity boundary, judge plan,
rulebook outline, dependency-map strategy, gap-inventory categories, phase gates,
mechanical queues, model allocation, and rollback policy.
```

### Rulebook Reviewer Prompt

```text
Review this migration rulebook adversarially.
Find ambiguity that will create inconsistent generated code.
For every finding, cite the exact rule or missing rule, give a concrete source example,
explain how it would fail in the target system, and propose one concise rulebook patch.
Do not review style unless it affects parity, safety, or build correctness.
```

### Fixer Agent Prompt

```text
You are fixing one migration queue item.
Inputs: source path, target path, rulebook, gap inventory, verifier failure, relevant logs.
Goal: make the smallest change that satisfies the verifier without weakening parity.
If the same failure appears systemic, stop and propose a rulebook update instead of patching around it.
Return: patch summary, tests run, remaining risk, and whether regeneration is needed.
```

## Verification Checklist

- [ ] Business case and non-goals are explicit.
- [ ] Judge/parity harness passes original and fails deliberately broken code.
- [ ] Rulebook exists before gap inventory is finalized.
- [ ] Dependency map controls batching and parallelism.
- [ ] Stress-test output was discarded after rules were updated.
- [ ] Queue is rebuilt from disk/verifier state and is resumable.
- [ ] Review is adversarial and findings cite rules.
- [ ] Repeated failures update process/rules, not just individual files.
- [ ] Compile/typecheck, smoke, and parity loops have recorded outputs.
- [ ] Expensive rebuild/test operations are serialized through one orchestrator/daemon.
- [ ] Final behavior is matched against original or intentional differences are documented.

## Near-Miss Anti-Triggers

Do not load this skill for a one-file cleanup, ordinary bug fix, pure planning task without code parity concerns, or generic coding style work unrelated to migration loops.