# Method Summary: AI-Assisted Code Migration

Source: https://claude.com/blog/ai-code-migration

## Thesis

Large migrations become tractable when agents operate inside objective, repeatable loops. Do not manually fix every generated-code failure. When failures repeat, update the rulebook, queue logic, verifier, or batching process that produced them.

## Six-Step Process

1. **Build the judge first.** Create portable tests or a parity harness that can compare original and target behavior. Validate that it passes original code and fails deliberately broken code.
2. **Create the rulebook, dependency map, and gap inventory.** Decide structure-preserving vs redesign; document ambiguity; map dependencies for parallel batches; inventory places where target tech requires explicit contracts the source did not.
3. **Stress-test the rules.** Run a disposable mini-migration or adversarial design review. Keep the lessons; throw away the generated code.
4. **Translate everything with a mechanical queue.** Rebuild the queue from disk/verifier state; fan out implementers; use adversarial reviewers; repeated findings amend rules and trigger regeneration.
5. **Compile/typecheck and smoke-test in loops.** Convert compiler and runtime failures into fix queues. Centralize expensive builds through an orchestrator/build daemon.
6. **Match behavior against the original.** Run parity tests/diffs across both systems; fix behavioral deltas; document intentional differences.

## Durable Practices

- Make verification mechanical and review adversarial.
- Use smaller models for high-volume implementation and strongest models for rule-writing/review.
- Front-load human effort into rulebook and stress testing.
- Make work queues mechanical, resumable, and derived from state.
- Review loop results and failure patterns rather than reading every generated file.