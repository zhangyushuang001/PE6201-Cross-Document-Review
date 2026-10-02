# Data Explainer

## What is included

This folder contains the frozen synthetic data used in the PE6201 formal experiment.

- `policies_final.json` — 20 simulated internal policies used for policy-grounded review.
- `cases_30_final.json` — 30 synthetic cross-document cases containing Purchase Orders,
  Commercial Invoices, Packing Lists and supporting fields.
- `ground_truth_30_FROZEN.json` — expected issue labels, severities, policy IDs and final
  decisions for all 30 cases.
- `GROUND_TRUTH_FREEZE_RECORD.txt` — record that the ground truth was frozen before
  formal evaluation.
- `GENERATION_LOG.md` — notes describing how the synthetic data was created.

## Split

- **DEV:** C01-C24
- **HOLDOUT:** C25-C30

The holdout set was not used for further tuning after the final configuration was frozen.

## Why synthetic data

The project avoids confidential company information. Synthetic cases also make it possible
to seed known inconsistencies and evaluate against explicit ground truth. The cases were
built from structured document templates and deliberately edited to introduce known
mismatches or policy conditions; the evaluated model was not used to define the answer key.
See `GENERATION_LOG.md` for the generation and freeze process.

## Case coverage

The dataset includes structured mismatches and policy-dependent cases such as:

- quantity, supplier, currency and Incoterm mismatches
- wrong PO references and unit-price variance
- missing manager approval, insurance, SDS and temperature-range references
- duplicate invoice and bank-detail change
- missing country of origin and missing line items
- free-text ambiguity
- prompt-injection text
- clean negative cases

## Important limitation

These cases are useful for architecture comparison, not as evidence of production accuracy.
The six-case holdout is project-authored rather than an external benchmark, and C30 exposed
a policy/label specification ambiguity that is documented rather than retroactively relabelled.
Real documents may introduce OCR errors, layout variation, incomplete fields, policy-version
conflicts and organisation-specific wording.
