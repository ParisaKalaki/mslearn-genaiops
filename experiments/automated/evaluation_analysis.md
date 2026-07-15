# Cloud Evaluation Analysis: Trail Guide Agent

## Evaluation Summary

Evaluated: 89 test cases
Time: ~52 minutes (3103 seconds)
Scoring: gpt-oss-120b as an LLM judge (1-5 scale)

| Evaluator         | Scored Items | Pass Rate         | Assessment                                                   |
| ----------------- | ------------ | ----------------- | ------------------------------------------------------------ |
| Intent Resolution | 5 / 89       | 100% (5/5 scored) | Most items errored; too few valid scores to draw conclusions |
| Relevance         | 72 / 89      | 1% (1/72)         | Overwhelming majority of scored items failed threshold       |
| Groundedness      | 0 / 89       | N/A (0/0)         | No items successfully scored                                 |

## Key Findings

### Critical Issue: Judge Model Incompatibility

The vast majority of evaluation items returned "Error" rather than a numeric score, across all three evaluators. This is very likely because the configured judge model, gpt-oss-120b, does not reliably produce the structured output format Microsoft Foundry's built-in evaluators expect (typically a JSON object with a score and reasoning field). This is a plausible consequence of using an open-weight model (OpenAI-OSS format) instead of a native OpenAI reasoning/chat model like gpt-5.1 or gpt-4o, which the evaluators were likely designed and tested against.

### Why This Happened

Earlier in this project, gpt-5.1 (the lab's recommended default) could not be deployed in this subscription due to a DeploymentModelNotSupported error unrelated to quota - gpt-5.1 and several other proprietary reasoning models were rejected outright despite confirmed available quota, likely due to Limited Access gating. gpt-oss-120b was the only model that passed all validation checks (catalog availability, lifecycle status, SKU support, and quota) and was used as a substitute for both the agent and the evaluation judge model.

### Recommended Next Steps

1. Do not treat these results as representative of actual agent quality - the failure is in the evaluator/judge-model pipeline, not necessarily the agent's response quality itself.
2. Investigate whether a different, evaluator-compatible model can be granted access for use specifically as the judge model (e.g., requesting Limited Access approval for gpt-5.1 or gpt-4o through Azure AI Foundry, separate from the agent's own serving model).
3. Manually review a sample of the "Error" rows to confirm whether the underlying agent responses were reasonable (spot-checking suggests they were, based on earlier manual scoring in the baseline/optimization experiments) - this would confirm the issue is isolated to the judge model's output format, not agent quality.
4. Re-run the evaluation once a properly supported judge model is available, and compare against this baseline attempt.

## Automated Evaluation Benefits (Conceptual, Pending Working Judge Model)

- Scales to hundreds/thousands of items efficiently
- Consistent scoring criteria across all evaluations
- Fast turnaround for large test sets
- Repeatable and trackable over time
- CI/CD ready for integration into deployment pipelines

## Lesson Learned

Automated LLM-as-judge evaluation is sensitive to the specific judge model's output format compliance - not every capable chat model is automatically compatible with a given evaluation framework's parsing expectations. This is a distinct consideration from raw model quality or quota availability, and should be validated with a small test batch before running a full-scale evaluation.
