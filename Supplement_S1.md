# Supplementary Information S1

This supplement documents the published AGRIBALYSE data, benchmark tolerances, archived evaluation and offline reproduction procedures for the decision workflow comparison on incomplete food profiles.

## Data and case construction

The source is ADEME's AGRIBALYSE 3.2 consumer dataset under the French Open Licence 2.0. The archive source manifest (`data/manifest.json`) identifies the two ADEME CSV extracts and the consumer workbook from [Dataverse release 15.0](https://doi.org/10.57745/XTENSJ), file 641911. It records source URLs, sizes, publisher checksums and SHA-256 hashes. The reference reconciliation (`docs/reference_decision.md`), data dictionary (`docs/data_dictionary.csv`) and exclusion log (`docs/exclusion_log.csv`) document the preparation checks. The final reference is the published `Total` column in `Detail etape`. `Synthese` totals are retained in a separate reference field. The reference label was unchanged for all 34 pairs when the summary totals replaced the stage-tab totals. The stage-tab labels comprised 12 cases of A dominance, six of B dominance, 15 trade-offs and one equivalence at pair level. Both sets of totals refer to 1 kg of food as consumed in France. Stage contributions are in kg CO$_2$ eq for climate change, m³ deprived for water deprivation and kg P eq for freshwater eutrophication, each per kilogram of product. Agriculture, processing, packaging, transport, distribution and consumption are the six stages. The data quality rating describes the source assessment and was not used as a distribution of outcome uncertainty.

The holdout seal (`outputs/pretest_final_seal_001/final_test_manifest.json`) commits the group assignment and the analysis design. The metadata roster manifest (`outputs/pretest_final_roster_001/roster_manifest.json`) records the eligibility and matching counts. It identifies 34 pairs drawn from 68 distinct groups. Of these, 13 passed the strict lexical screen and 21 were accepted on documented edible form and use. The reserve contained 435 eligible products in 189 groups. The exact group roster and source identifiers remain in the local private archive. The comparison protocol was fixed before opening the test partition, and the exact case and reference files were sealed before the first test prediction.

The queue manifest (`outputs/final_test_001/queue_manifest.json`) links the 102 cases, the mask and control assignments, the response schema and the separate reference file. It records the seed and the order of the second passes. In the case file, `case_id` and `pair_id` are opaque identifiers. `context` contains food family, subgroup and preparation. `masked_stages_per_product`, `hidden_stages` and `observed_stages` describe the three nested masks, and `observed_stages` contains the visible impacts and evidence identifiers for each side. `units`, `tolerance` and `retrieved_corpus_examples` supply the information used in the decision. Each example carries an evidence identifier, context and all six stage contributions from one corpus group. Product identities, the values of hidden stages and the reference relation were withheld from the model payload. The archived reference file contains source product and group identifiers, the official and summary totals, the corresponding relations, corpus example identifiers and the payload hash. It is accessible to the offline evaluator through a separate loader.

The prompt text and deterministic ordering are archived as source in `src/pilot_design.py`. The common prompt provides the decision rule and evidence requirements. The role prompt adds initial estimation, directed verification or neutral second estimation. The second passes append the same saved initial answer to the same case. `src/pilot_cases.py` records the mask input format and corpus retrieval algorithm. The canonical response schema (`schemas/agent_output.schema.json`) specifies the case identifier, three candidate impact totals per side, the predicted relation, probabilities, supporting evidence and a brief justification. An estimated flag and evidence identifiers accompany each reported impact. The validator checks units, relation coherence and permissible evidence. At mask zero it can also compare totals with stage sums. A total at a nonzero mask does not expose the individual estimated contributions.

## Benchmark tolerance provenance

Equation (1) comes from the study protocol. Section 7 defines the pairwise comparison and the 0.005 scale fraction. The pretest seal records that fraction and the hashes of the configuration, the tolerance constructor and the comparison code. The final queue passes the sealed fraction to the constructor. Protocol §7 (`docs/MASTER_PROTOCOL_ES.md#7-unidad-de-decisión-y-referencia-multicriterio`) · Pretest seal (`outputs/pretest_final_seal_001/final_test_manifest.json`) · Configuration (`configs/experiment.example.yaml`) · Queue builder (`src/final_queue.py`) · Tolerance constructor (`src/pilot_cases.py`) · Comparison code (`src/reference.py`)

For a positive corpus median written numerically as $s_j=m\,10^k$ with $1\leq m<10$, the third significant digit has place value $10^{k-2}$, where $k=\lfloor\log_{10}s_j\rfloor$. The rule uses the larger of one full increment at that place and 0.5% of the median. Both terms are expressed in the indicator's fixed reporting unit. This defines the numerical boundary for a pairwise label. The published references support the threshold concept and the significant-digit convention. [Roy and Vincke](https://doi.org/10.1287/mnsc.30.11.1323) · [Mendoza Beltran et al.](https://doi.org/10.1021/acs.est.7b06365) · [NIST SP 811, Appendix B.7](https://doi.org/10.6028/NIST.SP.811e2008)

All 102 sealed cases used the same three tolerances per kilogram of food. The values were 0.01275 kg CO$_2$ eq for climate change, 0.01 m³ deprived for water deprivation and $2.014262202295\times10^{-6}$ kg P eq for freshwater eutrophication. The digit increment determined the water deprivation tolerance. The 0.5% term determined the other two. Sealed cases (`outputs/final_test_001/cases.jsonl`)

## Reviewer reproduction

The companion reviewer archive (`SPC_reproduction_archive.zip`) contains the code, unfilled prompts, source data manifest and audited aggregate results needed for the checks below. From the extracted archive root, use Python 3.12 for the aggregate check and Matplotlib 3.10.7 for the figure scripts, which export vector PDF and SVG files and 600 dpi PNG files.

```text
python -B reviewer/verify_aggregates.py
python -B paper/plot_workflow.py
python -B paper/plot_mask_errors.py
```

The aggregate check recalculates all five paired effects and conservative sign intervals from the exported histograms and reconstructs the primary percentile bootstrap interval. The recovery audit retrieved the original pair identifiers and reproduced all five seeded bootstrap intervals from individual loss records using the original lexicographic pair order, Python random.Random, seed 20261108 and 2,000 draws. The previous histogram-order replay is retained separately because its A11–B0 endpoints differ from the archived interval.

## Post hoc diagnostics from the saved responses

After the primary analysis, we compared the saved A10 and A11 answers for the 99 scenarios with a valid response from both workflows. We used exact decimal equality for the impact totals and probabilities, structural equality for the evidence records and ordered per-indicator evidence identifiers, and exact string equality for justifications. Six scenarios had changes in the reported totals, one with two hidden stages and five with four, while the justification string changed in every scenario. The saved responses had identical labels, probability vectors and evidence fields across the 99 scenarios, and the string comparison did not assess the semantic quality of the explanations.

| Shared label error | Scenarios |
|---|---:|
| Reference trade-off classified as dominance | 14 |
| Reference dominance classified as trade-off | 10 |
| Reference equivalence classified as dominance | 2 |
| Reference trade-off classified as equivalence | 2 |
| Direction of reference dominance reversed | 1 |
| Total shared errors | 29 |

Only one of the 34 pairs met the reference rule for equivalence within tolerance, so the two errors on equivalent scenarios in the jointly valid subset concern that single pair. The diagnostic uses saved outputs and reference labels and is reported separately from the prespecified analysis of all scheduled scenarios.

On the 72 scenarios in the neutral control subset, the following descriptive accounting includes the same reported first-pass tokens in each workflow that used them. Unmetered attributed turns can overlap across workflows when the first pass timed out, and their actual token use could differ from the reported comparison.

| Workflow | Reported tokens | Attempted turns | Unmetered turns |
|---|---:|---:|---:|
| A10 | 1,008,621 | 72 | 1 |
| A11 | 2,043,888 | 143 | 3 |
| Neutral second pass | 2,042,879 | 143 | 3 |

## Internal audit provenance

The paths and checksums below record the provenance of the historical audit and refer to files in the study team's internal project. Case payloads, reference labels, the exact corpus, model responses and execution records remain in the private study archive, which is separate from the reviewer materials.

- `outputs/pretest_final_roster_001/roster_manifest.json`  
  SHA-256 `fe7e2cd62d44ed924068d1051d5be76007e127bd9d57586ef273d69f17a0573d`
- `outputs/final_test_001/queue_manifest.json`  
  SHA-256 `06328d6e7518372a5d63957187955163face445d3250c2adeddc37c3f931774d`
- `outputs/final_test_001/cases.jsonl`  
  SHA-256 `f319b88ecc5b48bff9b929351001a3ea171a734868ba296cfcf98cf2a4acaaf6`
- `data/private/final_test_001/reference.jsonl`  
  SHA-256 `e67bd1ce52bb2f259f3b08d43c1a34f0fe8eed7433df5348ad8b0b4d42aecd52`
- `data/private/pilot_baselines_001/corpus.jsonl`  
  SHA-256 `24428c173eb96defe96db88e4b94abb509e1fb5008c380d16e6897d10da066fc`
- `schemas/agent_output.schema.json`  
  SHA-256 `6918484c9bedc509cadd15be3a022a8edf7234cc2dacdbd6b974ed5850446edc`
- `src/pilot_design.py`  
  SHA-256 `d053b67382426d492b3b9644d8c896c6b8c49fb4dab1af92686247adbaa58129`
- `outputs/final_test_001/execution_manifest.json`  
  SHA-256 `2eb5c44dbc33ff802f1b2d95323dedb27b810b2dcd5aeae379cf83fe5c39230b`
- `outputs/final_test_001/execution_verification.json`  
  SHA-256 `5b563bab89dfd9dbd76cf68209b334b8cd316687e3063ce6d9e0cc33610c1de2`
- `outputs/final_test_001/offline_evaluation.json`  
  SHA-256 `070ba13aa4fe67a24a18892dc37e0d003ac8bdafd0b2abf263d1b187eddf334b`

The attempt records and telemetry are in `outputs/final_test_001/attempts/`, and the completed case records with the status of each arm are in `outputs/final_test_001/results/`. The cumulative token ledger is `outputs/mxc_hosted_synthetic_001/cumulative_ledger.json`, where the shared ledger was first created. `execution_verification.json` binds the completed run, execution status and evaluation, and the aggregate evaluation records the snapshot and ledger checksums. The sealed B0 corpus and its manifest are separate from the held-out reference.

## Archived execution and reproduction

The final execution manifest records Windows MXC validation, CrewAI 1.15.37, Codex CLI 0.167.0-alpha.3.2, `codex exec`, hosted model alias `gpt-6.1-sol` and low reasoning effort. The local CrewAI environment uses Python 3.12.3. The MXC validation record (`docs/MXC_CODEX_VALIDATION.md`) and synthetic isolation receipt (`outputs/mxc_isolation_002/manifest.json`) document the isolation checks. The provider did not expose a server-side revision. An offline replay from saved records is deterministic, while a later hosted run with the same alias may produce different answers. Dependency versions are in `requirements.lock.txt` and `requirements-crewai.lock.txt`.

Each hosted turn had a 75-second timeout, and worker cleanup could extend beyond that limit. Prompt and accepted-response size limits were 16,384 and 8,192 UTF-8 bytes. The 32,768-token value served as a post-turn acceptance limit and as a provisional ledger debit when telemetry was missing. It was not a server-side token cap. The prelaunch quota checks required ordinary included usage, zero paid credits and automatic reload disabled. The final execution policy paused at 94% usage and permitted at most one authorized earned reset. Recorded usage stayed between 57% and 60%, and the run used no reset or paid API calls. The execution verification (`outputs/final_test_001/execution_verification.json`) separates measured and provisioned tokens.

After the test opened, a compatibility correction narrowed the resume guard for a recorded timeout with `error` as the only additional item type, zero completed turns and zero malformed event lines. The adjustment preserved the failed attempt and its provision. `outputs/final_test_001/runtime_audit_001/adjustment.json` records the old and new execution manifests and the hashes of the six preserved attempts. The same folder contains `execution_manifest.before.json`, `final_live_run.before.py` and the records of the targeted process closures after timeout. The held-out cases, references and scoring rule remained fixed.

The study team used `python -B -m src.final_live_score` with the private inputs to replay the evaluation without model calls and compared the report with the sealed aggregate above. The reviewer commands in Section 1.3 use the export of that audit.


## Recovery revision history 17 November 2026

This is a revision of the recovered original paper/supplement.md, not an original supplement newly inferred from aggregates. The source supplement SHA-256 is 9665a83e0aace7ff8bc79c60f6df95d5fbed504f85757af73f82ca444b9b5e53. The recovery retained 34 pairs, 102 cases, 274 physical-attempt checkpoints, 269 valid final responses, five timeouts and two skipped second passes. The case payloads and references were reconstructed byte for byte from the pinned historical environmental files and frozen construction code. A separate implementation reproduced all counts, pair-cluster bootstrap endpoints and token identities. Raw provider streams, provider-side call identifiers and the server model revision were not recovered. The audit used Python 3.13.15 and the supplied audit requirements (SciPy 1.17.0), distinct from the historical Python 3.12.3 and source lockfile SciPy 1.18.1; no historical environment was changed.

The real post hoc illustration is P-384ee988e0a9c069-M2; its source bundle is CASE_P03_REAL.json. No per-stage imputations were retained in the model output. Review of food comparability and Feisar's independent review remain pending. The depurated local package is pending human approval and is neither published nor deposited. Full Codex histories and environmental source workbooks are not included in it.

Offline recovery commands and their observed outputs are in README_REPRODUCE.txt and logs/. Source-to-claim changes are in SOURCE_TO_CLAIM.csv. Historical diagnostics and proposed future comparisons have not been converted into new experiments.
