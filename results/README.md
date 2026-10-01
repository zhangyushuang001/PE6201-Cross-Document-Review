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

## Key formal results

| Approach | DEV high-severity recall | HOLDOUT high-severity recall | HOLDOUT precision | HOLDOUT false flags/case |
|---|---:|---:|---:|---:|
| Rule-only | 100.00% | 100.00% | 100.00% | 0.000 |
| Direct-model | 88.89% | 85.71% | 77.78% | 0.333 |
| Full-context | 100.00% | 100.00% | 100.00% | 0.000 |
| RAG-hybrid | 100.00% | 100.00% | 90.00% | 0.167 |

## Interpreting the metrics

- Direct-model missed the predefined high-severity recall target on both splits.
- Full-context was accurate but used more input context because all policies were sent on
  every request.
- RAG-hybrid preserved 100% high-severity recall while reducing input context, but produced
  one holdout false positive on C30.
- Rule-only performed strongly on the structured synthetic checks but missed the free-text
  ambiguity case C24 in DEV.

## Evaluation limitations

The holdout contains only six cases, so individual errors move percentages substantially.
The synthetic dataset is designed for controlled comparison rather than production claims.
The project also did not directly measure realised reviewer time saved or business loss avoided.
