# AI Evaluation Standard

**Version**: 1.0.0 | **Last Updated**: 2026-10-13 | **Implements**: Constitution Principle IX,
AI Engineering Standards §5, Verification Standards §7

AI features need two kinds of verification. Neither replaces the other.

| | Software tests | Evaluations |
|---|---|---|
| Proves | Deterministic code around the model | Model-dependent quality |
| Model output | Fixed / recorded responses | Live model under test |
| Runs | Every PR | Before releasing model, prompt, retrieval, or tool-policy changes; scheduled per risk |
| Method tag | `[UNIT]` / `[INTEGRATION]` / `[E2E]` | `[EVAL]` |

## 1. Software tests (deterministic)

Cover, with fixed model responses:
- request construction (system prompt, context, tools, parameters);
- tool authorization — enforced in code, outside the model; denied tool calls are rejected;
- structured-output validation, including invalid or truncated output;
- persistence of results;
- retries, fallbacks, timeouts, and provider failure;
- guardrail and refusal routing;
- error handling shown to the user.

## 2. Evaluations

Before running:
1. **Dataset** — versioned, representative inputs in `evals/`, including adversarial and risky
   cases (prompt injection, unauthorized tool requests, out-of-scope questions, sensitive
   topics for the domain).
2. **Rubric** — what is scored: correctness, groundedness, relevance, hallucination, refusal
   behavior, safety, and domain quality, as applicable.
3. **Thresholds** — pass threshold per metric and failure severity (e.g. any safety failure
   blocks release).
4. **Adjudication** — how ambiguous results are resolved by a human.

Running:
- Compare against the approved baseline; investigate every regression.
- Repeat samples where variance matters and report the spread.
- Track latency and cost when requirements depend on them.

A plausible answer in a manual demo is not an evaluation.

## 3. Eval record

```text
Capability:        support-assistant
Model / version:   <provider>/<pinned model ID>
Prompt version:    prompts/support@v7
Retrieval config:  index v3, top-k 8, reranker on
Tool policy:       read-only tools; refunds require human confirmation
Dataset version:   evals/support/v4 (220 cases, 40 adversarial)
Results:           groundedness 0.94 (≥ 0.90) · refusal 100% (= 100%) · injection 0 passes (= 0)
Baseline delta:    +0.02 groundedness, no new failures
Result:            PASS
Reviewer / date:   <name>, 2026-10-13
```

Store records with the release evidence so a later model or prompt change can be compared.
