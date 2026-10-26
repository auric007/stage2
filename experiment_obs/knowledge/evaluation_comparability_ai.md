# Cross-project saved-report comparability audit

Author: Codex. Human companion: [evaluation_comparability.md](evaluation_comparability.md).
Updated: 26 October 2026, 19:50 IST. Task: `RN-20261026-vlm-stage2-report-audit`. Correlation: `RN-20261026-week-start`.

The researcher supplied [the prediction guide](test_predictions_guide_ai.md) for a comparison with the latest four-class class-only VLM. Completed a metadata and aggregate-report audit, documented in the [primary VLM audit](/sfs/vlm_qwen/experiment_obs/knowledge/evaluation_comparability_ai.md). No Stage2 code, predictions, protocol, weights or BEST pointer changed. No inference, rematching or instance-level outcome comparison ran.

Observed: both original manifests name the same 80 scenes; the clean VLM subset contains 67. The three Stage2 export folders cover all 80 names. The guide's aggregate validation values are arithmetically consistent. They are not directly comparable with C1 headline scores: candidate sets, GT inclusion/taxonomy, IoU and assignment rules differ. Draft P04 v2 and per-class model/confidence choices were selected on test80, so their scores are exploratory rather than a new held-out estimate.

The fixed m0001 reference is itself a two-backbone ensemble; best_per_class combines four model sources. These are different baselines. No ranking, shared-miss count or integration benefit is established by this audit.

Source code HEAD: `e5af44c4bcc6763cb5031887bc9c56b071ad4508`; pre-existing dirty state: `True`. No research code changes were made. See the primary audit for input hashes and verification details.
