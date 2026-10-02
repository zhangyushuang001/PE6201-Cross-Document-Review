# Synthetic-data generation log

## Purpose and scope

- Purpose: PE6201 end-of-course project only.
- No confidential or real customer data is used.
- The corpus contains 30 synthetic cases: 24 DEV cases (C01-C24) and 6 HOLDOUT cases (C25-C30).
- The policy corpus contains 20 simulated internal policy sections (P01-P20).
- These policies are project fixtures, not real customs or trade-law advice.

## How the cases were created

The cases were built from structured Purchase Order, Commercial Invoice, Packing List
and supporting-field templates. Inconsistencies were seeded deliberately by editing
specific fields or document notes (for example quantity, supplier, currency, price,
approval references, free-text contradictions and prompt-injection text). The evaluated
foundation model was not used to decide which seeded error should count as the answer key.

Clean cases and hard negatives were also included so the system had to avoid flagging
correct documents rather than only detect planted errors.

## Ground-truth process

Expected issue type, severity, policy ID and final decision were written into a separate
ground-truth file and manually reviewed before formal model evaluation. The ground truth
was then frozen and recorded in `GROUND_TRUTH_FREEZE_RECORD.txt`.

Formal model outputs must not be used to edit the frozen answer key. If a later run exposes
a labelling or policy-specification ambiguity, it is documented as an evaluation limitation
instead of silently relabelling the case. C30 is the explicit example in this project.

## DEV and HOLDOUT discipline

- Retrieval and prompt development used DEV cases only.
- The final retrieval configuration was frozen before the HOLDOUT run.
- C25-C30 were not used for further tuning after their formal outputs were observed.
- The holdout is still synthetic and project-authored, so it supports controlled architecture
  comparison rather than production/generalisation claims.

## Reproducibility

The repository checks in the final case file, policy file, frozen ground truth, freeze
record and archived formal outputs. The notebooks can reproduce the workflow when the
same files, model configuration and pricing assumptions are used.
