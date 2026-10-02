# Evaluation Explainer

## Evaluation goal

The formal experiment compares four approaches on the same frozen case set:

1. **Rule-only**
2. **Direct-model**
3. **Full-context**
4. **RAG-hybrid**

The primary success criterion is:

- **High-severity issue recall >= 90%**
- **Average false flags <= 1 per case**

Secondary measures include overall issue precision/recall, decision accuracy, policy grounding,
token usage, cost, latency and schema-error count.

## Formal protocol

- Model: `openai/gpt-5.6-luna`
- Temperature: `0.0`
- RAG top-k: `5`
- Embedding model: `sentence-transformers/all-MiniLM-L6-v2`
- DEV: C01-C24
- HOLDOUT: C25-C30
- Ground truth was loaded for scoring after predictions were generated.
- The holdout configuration was not tuned after holdout results were observed.
- Formal cost calculations use fixed notebook assumptions of **$0.20/M input tokens**
  and **$1.20/M output tokens**; these values are part of the archived experiment and
  are not presented as permanent/current provider pricing.

## Output files

### Development
- `formal_dev_summary.csv`
- `formal_dev_*_details.csv`
- `formal_dev_raw_results.json`

### Holdout
- `formal_holdout_summary.csv`
- `formal_holdout_*_details.csv`
- `formal_holdout_raw_results.json`

The summary CSVs contain arm-level metrics. The detail CSVs show case-level expected and
predicted issue sets. The raw JSON files preserve model outputs, token counts, latency,
retrieved policy IDs and cost.

### Case-level `success` definition

The `success` column is the experiment's predefined primary case-level criterion. A case
counts as successful when all expected HIGH-severity issues are found, false flags are
no more than one, and there is no schema error. It **does not require the final
PASS/HUMAN_REVIEW decision to match**. Decision accuracy is therefore reported separately.
This explains, for example, why C30 can have `success=True` while its decision differs
from the frozen ground truth.

## Key formal results

| Approach | DEV high-severity recall | HOLDOUT high-severity recall | HOLDOUT precision | HOLDOUT false flags/case |
|---|---:|---:|---:|---:|
| Rule-only | 100.00% | 100.00% | 100.00% | 0.000 |
| Direct-model | 88.89% | 85.71% | 77.78% | 0.333 |
| Full-context | 100.00% | 100.00% | 100.00% | 0.000 |
| RAG-hybrid | 100.00% | 100.00% | 90.00% | 0.167 |

Additional RAG-hybrid decision accuracy:
- DEV: **95.83%**
- HOLDOUT: **83.33%**

## Interpreting the metrics

- Direct-model missed the predefined high-severity recall target on both splits.
- Full-context was accurate but used more input context because all policies were sent on
  every request.
- RAG-hybrid preserved 100% high-severity recall while reducing input context. Relative
  to the frozen ground truth, it produced one holdout false positive on C30. C30 also
  exposed a specification ambiguity: the case was labelled PASS because a temperature
  range was present in supporting fields, while policy P12 stated that the packing list
  itself should contain the range. The frozen ground truth was kept unchanged.
- DEV case C18 shows a routing error that issue-level metrics alone can hide:
  RAG-hybrid correctly detected `missing_weight_field` but returned `PASS`. The archived
  result is unchanged; the proposed deployment adds a deterministic post-validation gate
  so any validated issue routes to `HUMAN_REVIEW`.
- Rule-only performed strongly on the structured synthetic checks but missed the free-text
  ambiguity case C24 in DEV.

## Evaluation limitations

The holdout contains only six cases, so individual errors move percentages substantially.
The synthetic dataset is designed for controlled comparison rather than production claims.
C30 also shows that policy wording, case schema and label definitions should be aligned
more explicitly before future evaluations. The project did not directly measure realised
reviewer time saved or business loss avoided.
