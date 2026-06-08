# ToxGuard

ToxGuard is a toxicology-focused benchmark for evaluating whether large language models prematurely convert toxicological signals into human risk conclusions before exposure, population, context, evidence-strength, and threshold conditions are satisfied.

## Current locked artifacts

- `docs/evidence_contract_v0.1.md` — locked at tags `v0.1` and `v0.1.1`
- `docs/scoring_signatures_v0.1.md` — locked at tag `signatures-v0.1`

## Core endpoints

- PTHCR — Premature Toxicological Hazard Commitment Rate
- HVRC-R — Hazard-vs-Risk Conflation Rate

## Scope

Five scenario classes:
- Carcinogenicity
- Threshold-based toxicity
- Chronic/cumulative toxicity
- Endocrine disruption
- Occupational exposure

Five evidence dimensions:
- Exposure
- Population
- Context
- Evidence strength
- Threshold/reference metric

ToxGuard is distinct from OncoGuard-Response: it evaluates commitment regulation in toxicological reasoning, not treatment-management action authorization.
