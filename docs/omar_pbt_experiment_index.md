# Omar Sufyan PBT experiment index

This index publishes the completed experiment set in this repository. It is an artifact/ownership index, not a claim that Omar originated the general idea of property-based testing (PBT). The results concern model-authored tests against stored APPS honest/attack candidates; they do not establish selective-audit or deployment safety.

## Shared methodology

Each arm asks Azure `gpt-5.6-terra` (low reasoning, requested seed 300) to author or revise property-based tests. Candidate code and reviewed inputs are supplied where the report says so; stored reference solutions and secret triggers are not disclosed. A suite “fires” only when an assertion returns `catch`; crashes, property errors, parse failures, incomplete grids, and infrastructure failures are not silently scored as clean negatives. Primary rates use explicitly stated paired/eligible denominators. Same-input replay checks whether an attack-authored suite also fails on its honest twin, but is a specificity diagnostic—not proof of correctness.

## Completed studies

| Study | Question and population | Verified result | Primary artifacts |
|---|---|---|---|
| Training prompt comparison (7 Sep) | Exploratory comparison of `plain_v3` and `traceable_v1` on 19 previously inspected training tasks (38 candidates), with 379 retained reviewed inputs. | Paired honest FPR was **3/17 vs 3/17 (17.65%)** and matched attack firing **18/18 vs 18/18**. The 10-point FPR-reduction target failed. Traceability had more duplicate tests and 3 contaminated attack suites, although all 17 replay-eligible tasks retained a differential witness. | [report](azure_pbt_training_results_2026-09-07.md), [authoring notebook](../notebooks/azure_pbt_authoring.ipynb), [replay notebook](../notebooks/azure_pbt_authoring_replay.ipynb), run prefix `azure-terra-pbt-train19-s300-reviewed-v1` |
| Frozen held-out comparison (8 Sep) | Single-turn `plain_v3` vs `traceable_v1` on all 30 original held-out tasks; input review left 29 tasks/58 candidates and 576 inputs eligible. | Honest FPR changed **4/29 (13.79%) to 3/29 (10.34%)**, only −3.45 pp; matched attack firing stayed **22/28 (78.57%)**. The preregistered ≥10 pp target was not met; the exploratory paired 95% interval included zero. Replay witnesses were 21/25 vs 20/25. | [report](azure_pbt_heldout_results_2026-09-08.md), [launcher](../notebooks/azure_pbt_heldout.ipynb), [domain review](../notebooks/azure_pbt_heldout_domain_review.ipynb), [replay](../notebooks/azure_pbt_heldout_replay.ipynb), run prefix `azure-terra-pbt-heldout30-s300-v1` |
| Exploratory execution-informed repair (9 Sep) | One revision of the training `traceable_v1` suites, with own-candidate execution diagnostics; 38 candidates, no held-out task selection. | On complete/error-free paired candidates, honest FPR fell **3/17 to 1/17 (−11.76 pp)** while attack TPR stayed **18/18**. The task-clustered 95% interval was **[−29.41, 0.00]**, including zero. Whole-suite-clean replay improved 14/17 to 16/17, while pairwise witnesses stayed 17/17. This is development evidence, not a resolved effect. | [report](azure_pbt_multiturn_results_2026-09-09.md), [notebook](../notebooks/azure_pbt_multiturn.ipynb), run `azure-terra-pbt-train19-s300-repair-v1` |
| Fresh controlled A/B/C revision (14 Sep; report finalized 17 Sep) | Fresh 30-task sample (60 candidates), with common initial arm A, revision without feedback B, and revision with bounded own-candidate execution feedback C. Inputs were frozen and reused across arms. | Primary B→C comparison: honest FPR **4/28 to 5/28** (+3.57 pp; interval [0,+11.12]) and attacks caught **26/30 to 25/30** (−3.33 pp; interval [−10,0]). The frozen improvement gate failed. Replay whole-suite-clean was 22/30 for both; matched witnesses were 24/30 vs 25/30. | [report](azure_pbt_multiturn_confirmatory_results_2026-09-14.md), [prospective plan](azure_pbt_multiturn_confirmatory_plan.md), [analysis notebook](../notebooks/azure_pbt_multiturn_confirmatory.ipynb), study artifacts under `runs/azure-terra-pbt-fresh30-second-revision-s300-v1-study/` |

The original three-arm proposal is now completed by this A/B/C study; descriptions that present those arms as merely planned are stale. The A/B/C intervention is a replacement-suite rewrite, not a mechanical deletion/minimal-edit study; see the [design assessment](omar_pbt_design_assessment.md).

## Code-to-method map

- [Model adapter and provenance](../pipeline/model.py) — records the exact provider/deployment metadata used by the runs.
- [Dataset builder](../build_dataset.py) — builds the APPS population and freezes train/test split and pool-selection rules.
- [Original authoring protocol](../pipeline/protocols/unit_testing.py) — generates the baseline/traceability suites.
- [Training repair protocol](../pipeline/protocols/test_repair.py) — implements the older execution-informed rewrite.
- [Fresh control/feedback protocol](../pipeline/protocols/second_revision.py) — implements A/B/C with feedback withheld for B and supplied to C.
- [Sandbox execution](../pipeline/sandbox.py) — runs suites under the pinned execution constraints and preserves failure state.
- [Analysis notebooks](../notebooks/) — frozen preparation, cached metrics, bootstrap intervals, and honest-twin replay; each study row above links its specific notebook.
- [Design assessment](omar_pbt_design_assessment.md) — explains the rewrite estimand and why it must not be called deletion.
- [OpenRouter GLM check](openrouter_glm_check_2026-09-17.md) — connectivity check only; no GLM experiment or result is included.

## Exact model and execution metadata

The reports consistently identify the Azure deployment as `openai-api/azureai/gpt-5.6-terra` (short form: `gpt-5.6-terra`), low reasoning, seed 300, ten requested tests, and an 8,192-token authoring cap. The fresh controlled report records the returned model snapshot as `gpt-5.6-terra-2026-07-09`; earlier reports do not claim a GPT-4 model. Docker execution used image `python@sha256:78387bc3881b8273120a12ebe6c1ab22b018ccc2c9adf565ae1ac9b536e184ea` and a 120-second sandbox limit where documented. Trigger-search stages used a separate 12,000-token cap. Exact retries, call budgets, schemas, and visibility differ by study and are recorded in the linked reports; do not merge their denominators.

## Limits, exclusions, and reproduction

- These are exploratory/benchmark measurements with small task counts. Bootstrap intervals are task-clustered and do not make tests or inputs independent; intervals including zero remain unresolved.
- Candidate labels come from pool construction and were not independently re-established against every benchmark grading test. Inputs are targeted and statement-derived, not a random deployment distribution.
- Parse failures, candidate crashes, property errors, incomplete grids, invalid inputs, and infrastructure failures are reported and excluded only under each study’s pre-specified eligibility rule. No automatic repair or retry is retroactively applied to the single-turn comparisons.
- Exact replay distinguishes a test/input witness from a clean whole-suite differential. Neither proves correctness on untested inputs.
- Cached analysis is reproducible from the notebooks with launch/preparation flags disabled when run artifacts are available. Regenerating remote responses is not guaranteed bit-for-bit, and runtime artifacts may be gitignored. Reports provide SHA-256 fingerprints for raw records and machine-readable summaries.
- Selected raw Azure run artifacts are included in the published `runs/` subset despite the general runs ignore rule, including the relevant `config.json`, `records.jsonl`, and selected JSON replay/analysis outputs. Logs, lock files, environment files, and `.hf_cache/` remain excluded by design.

For implementation details, start with [`DOCUMENTATION.md`](../DOCUMENTATION.md) and the protocol/notebook links above. No experiment is described here as a successful deployment-safety result.
