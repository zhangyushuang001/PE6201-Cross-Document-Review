# AI-Assisted Cross-Document Review for Import and Export Documentation Compliance

PE6201 Emerging AI Technologies — End-of-Course Project

## Project goal

This project evaluates a lightweight pre-shipment review workflow for trading SMEs.
It cross-checks a Purchase Order, Commercial Invoice and Packing List against each
other and against simulated internal policies. The system is decision support only:
legal decisions, customs filing and autonomous approval are out of scope.

The predefined primary target is:

- **High-severity issue recall >= 90%**
- **Average false flags <= 1 per case**

## Product documentation

### Persona
Primary user: an import/export operations executive in a trading SME who needs a
pre-shipment review of Purchase Orders, Commercial Invoices and Packing Lists.

### Inputs
- Purchase Order
- Commercial Invoice
- Packing List
- Supporting fields such as approval, insurance, SDS and temperature references
- Simulated internal policy corpus

### Outputs
The review returns structured JSON containing:
- detected issue type(s)
- severity
- supporting evidence
- policy ID when policy grounding is used
- final decision: `PASS` or `HUMAN_REVIEW`

### High-level architecture

The formal experiment evaluates four approaches separately. Based on those results,
the proposed deployment is a layered workflow:

```text
Documents + Supporting Fields
            |
            +-------------------------------+
            |                               |
            v                               v
 Deterministic checks                 RAG-hybrid review
 for stable structured                observable-fact routing
 comparisons and thresholds           + semantic retrieval
                                            |
                                            v
                                   Top-5 relevant policies
                                            |
                                            v
                                      GPT-5.6 Luna
            |                               |
            +---------------+---------------+
                            |
                            v
           Structured issues + evidence + policy ID
                            |
                            v
                 Schema / output validation
                            |
                            v
             Deterministic routing gate
                            |
                            v
                  PASS / HUMAN_REVIEW
```

The repository contains four formal comparison arms: rule-only, direct-model,
full-context and RAG-hybrid. Their predictions were evaluated independently; rule-only
predictions were not merged into the RAG-hybrid arm. The deterministic routing gate
shown above is a proposed deployment improvement identified after the frozen formal run;
the archived formal outputs were not retroactively changed.

### Metrics targeted
- High-severity issue recall **>= 90%**
- Average false flags **<= 1 per case**
- Secondary measures: overall issue precision/recall, decision accuracy,
  policy grounding, input/output tokens, cost and latency

### Metrics reached
- RAG-hybrid DEV high-severity recall: **100%**
- RAG-hybrid HOLDOUT high-severity recall: **100%**
- RAG-hybrid HOLDOUT overall issue recall: **100%**
- RAG-hybrid HOLDOUT precision: **90%**
- RAG-hybrid HOLDOUT false flags per case: **0.167**
- RAG-hybrid DEV decision accuracy: **95.83%**
- RAG-hybrid HOLDOUT decision accuracy: **83.33%**
- Direct-model missed the high-severity recall target on both DEV (**88.89%**)
  and HOLDOUT (**85.71%**)

See `data/README.md` for the dataset explanation and
`results/README.md` for the evaluation protocol and metric definitions.

## Compared approaches

1. **Rule-only** — deterministic checks for structured fields and explicit thresholds.
2. **Direct-model** — GPT-5.6 Luna reviews the case without internal policy context.
3. **Full-context** — GPT-5.6 Luna receives all 20 simulated internal policies.
4. **RAG-hybrid** — observable-fact routing plus semantic retrieval selects the top 5
   relevant policies before GPT-5.6 Luna review.

## Frozen formal configuration

- Model: `openai/gpt-5.6-luna`
- Temperature: `0.0`
- RAG top-k: `5`
- Embedding model: `sentence-transformers/all-MiniLM-L6-v2`
- Dataset: 30 synthetic cases
  - DEV: C01-C24
  - HOLDOUT: C25-C30
- Ground truth was frozen before formal testing.
- The holdout configuration was not tuned after seeing holdout results.
- Formal token-cost calculations use the fixed notebook assumptions of
  **$0.20 per million input tokens** and **$1.20 per million output tokens**.
  These are experiment assumptions and should be repriced for any later deployment.

## Formal results

| Approach | DEV high-severity recall | HOLDOUT high-severity recall | HOLDOUT precision | HOLDOUT false flags/case | HOLDOUT cost |
|---|---:|---:|---:|---:|---:|
| Rule-only | 100.00% | 100.00% | 100.00% | 0.000 | $0.000000 |
| Direct-model | 88.89% | 85.71% | 77.78% | 0.333 | $0.002136 |
| Full-context | 100.00% | 100.00% | 100.00% | 0.000 | $0.003506 |
| RAG-hybrid | 100.00% | 100.00% | 90.00% | 0.167 | $0.002565 |

Key observations:

- Direct-model did not meet the >=90% high-severity recall target on either DEV or HOLDOUT.
- Full-context was accurate but used substantially more input tokens because all policies
  were sent on every request.
- RAG-hybrid preserved 100% high-severity recall on both DEV and HOLDOUT while using
  less policy context than Full-context.
- Relative to the frozen ground truth, RAG-hybrid produced one holdout false positive
  on C30 and routed it to HUMAN_REVIEW. C30 also exposed a specification ambiguity:
  the case was designed as a clean negative with a temperature range in supporting fields,
  while policy P12 stated that the packing list itself should contain that range. The frozen
  ground truth was kept unchanged rather than relabelled after formal testing.
- DEV case C18 exposed a separate routing failure: RAG-hybrid correctly detected
  `missing_weight_field` but returned `PASS`. This is why DEV issue precision/recall can
  be 100% while decision accuracy is 95.83%. The proposed deployment therefore adds a
  deterministic post-validation gate so any validated issue routes to `HUMAN_REVIEW`.
- Rule-only performed strongly on the synthetic structured checks but missed the DEV
  free-text ambiguity case C24.

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 01_formal_DEV_C01_C24.ipynb
│   └── 02_formal_HOLDOUT_C25_C30.ipynb
├── data/
│   ├── README.md
│   ├── policies_final.json
│   ├── cases_30_final.json
│   ├── ground_truth_30_FROZEN.json
│   ├── GROUND_TRUTH_FREEZE_RECORD.txt
│   └── GENERATION_LOG.md
└── results/
    ├── README.md
    ├── formal_dev_summary.csv
    ├── formal_dev_*_details.csv
    ├── formal_dev_raw_results.json
    ├── formal_holdout_summary.csv
    ├── formal_holdout_*_details.csv
    └── formal_holdout_raw_results.json
```

## How to run in Google Colab

1. Open a notebook from `notebooks/`.
2. Upload the three required JSON files from `data/`:
   - `policies_final.json`
   - `cases_30_final.json`
   - `ground_truth_30_FROZEN.json`
3. Add an OpenRouter key to Colab Secrets using the name:
   `OPENROUTER_API_KEY`
4. Enable notebook access to that secret.
5. Run the notebook from top to bottom.

**Do not hard-code or commit the API key.**

The DEV notebook makes 24 x 3 = 72 model calls.
The HOLDOUT notebook makes 6 x 3 = 18 model calls.

## Responsible-use boundary

The prototype is intended for pre-shipment decision support. In the proposed deployment,
validated issues or ambiguity are routed to `HUMAN_REVIEW` through a deterministic
post-processing gate; this gate is an improvement identified from the frozen C18 result
and was not added retroactively to the formal experiment. The evaluation uses synthetic
data, so results should not be interpreted as production performance on real trade documents.

## Reproducibility note

The CSV and JSON files under `results/` are the archived outputs from the formal runs.
They are included so the reported metrics can be inspected without re-running paid API calls.
