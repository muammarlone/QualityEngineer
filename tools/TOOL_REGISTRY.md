# TOOL_REGISTRY.md
## Persona D1 — The Quality Engineer | Complete Tool Registry

**Version:** 1.0.0
**Status:** Production-ready
**Date:** 2026-06-28
**Owner:** D1 The Quality Engineer
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`

---

## 1. Tool Registry Overview

All tools for D1 arms are registered in this document. Each tool has a unique ID, owner arm, execution mode, and full input/output contract. The 15 tools span Six Sigma assessment, Statistical Process Control, test gap analysis, coverage optimization, flaky test detection, defect analysis, quality scorecard generation, DPMO computation, code review auditing, test reliability analysis, sigma level tracking, and improvement plan generation.

---

## 2. Tool Definitions

### TOOL-01: SixSigmaAssessor

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-01` |
| **Name** | `SixSigmaAssessor` |
| **Description** | Core DMAIC engine — phase management, evidence gate validation, sigma level scoring, process capability index computation, and improvement plan generation. Validates all phase transitions against evidence requirements and blocks transitions without sufficient evidence. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor) |
| **Input** | `SixSigmaAssessRequest`: `{target_id: string, target_type: enum, data_uris: object, standards: object, historical_baseline_id: UUID, assessment_scope: enum}` |
| **Output** | `SixSigmaAssessResponse`: `{assessment_id: UUID, phases: object, sigma_level: float, dpmo: int, cpk: float, overall_grade: enum, improvement_plan: object, deliverable_uris: array}` |
| **Execution Mode** | Python/CPU, statistical engine, deterministic core (R-GATE-2), up to 2GB memory |
| **Auth** | JWT RS256, role `d1-quality:assess` |
| **Timeout** | 1800s default, 3600s max |
| **Error Handling** | `on_error: return partial DMAIC with blocked phases flagged` + `log_to_p2` + `alert_d5` |
| **Example** | `POST /v1/tools/six_sigma_assessor` with `{target_id: "pipeline-alpha", target_type: "pipeline", data_uris: {defect_logs: "s3://defects.csv", test_results: "s3://tests.json"}, standards: {target_sigma: 4.0, target_cpk: 1.33}}` |

### TOOL-02: SPC_Controller

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-02` |
| **Name** | `SPC_Controller` |
| **Description** | Core Statistical Process Control engine — control limit computation, Western Electric rule evaluation, process stability classification, and alert generation. Supports all standard control chart types with deterministic statistical computations validated against NumPy reference fixtures. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Control phase), `ARM-02` (SPC Controller — primary) |
| **Input** | `SPCControlRequest`: `{target_id: string, metric_name: enum, chart_type: enum, data_points: array[float], specification_limits: object, control_limits: object, rule_suite: array[enum]}` |
| **Output** | `SPCControlResponse`: `{chart_id: UUID, center_line: float, ucl: float, lcl: float, rule_violations: array, stability_status: enum, capability_indices: object, alert_required: bool}` |
| **Execution Mode** | Python/CPU, statistical engine, deterministic core (R-GATE-2), lightweight (< 500MB memory) |
| **Auth** | JWT RS256, role `d1-quality:control` |
| **Timeout** | 300s default, 600s max |
| **Error Handling** | `on_error: return last known control limits with warn status` + `alert_d5` |
| **Example** | `POST /v1/tools/spc_controller` with `{target_id: "pipeline-alpha", metric_name: "build_time", chart_type: "x", data_points: [14.3, 15.1, 13.8, ...], specification_limits: {usl: 20, lsl: 5}}` |

### TOOL-03: ControlChartGenerator

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-03` |
| **Name** | `ControlChartGenerator` |
| **Description** | Generate control chart images (X-bar, R, S, X, MR, p, np, c, u) with Western Electric rule annotations, specification limit overlays, and GAI-OBSERVE visual theme. Outputs PNG/SVG with accessibility alt text. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Measure/Control), `ARM-02` (SPC Controller — primary) |
| **Input** | `ChartGenRequest`: `{chart_id: UUID, chart_type: enum, data: array, center_line: float, ucl: float, lcl: float, specification_limits: object, rule_violations: array, theme: enum, dimensions: object}` |
| **Output** | `ChartGenResponse`: `{chart_uri: string, alt_text: string, data_table: object, format: enum}` |
| **Execution Mode** | Python/CPU, matplotlib/seaborn, up to 1GB memory for large datasets |
| **Auth** | JWT RS256, role `d1-quality:visualize` |
| **Timeout** | 120s default, 300s max |
| **Error Handling** | `on_error: return ASCII chart with data table` + `flag_visual_degradation` |
| **Example** | `POST /v1/tools/control_chart_generator` with `{chart_id: "uuid", chart_type: "x", data: [14.3, 15.1, ...], center_line: 14.3, ucl: 23.9, lcl: 4.7, theme: "gai-observe-dark"}` |

### TOOL-04: WesternElectricAnalyzer

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-04` |
| **Name** | `WesternElectricAnalyzer` |
| **Description** | Apply all 8 Western Electric rules to control chart data: WE1 (single point beyond 3σ), WE2 (2 of 3 beyond 2σ), WE3 (4 of 5 beyond 1σ), WE4 (8 consecutive on one side), WE5 (6 trending), WE6 (14 alternating), WE7 (15 in zone C), WE8 (8 beyond 1σ both sides). Configurable thresholds and custom rule support. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-02` (SPC Controller — primary) |
| **Input** | `WEAnalysisRequest`: `{data_points: array[float], center_line: float, sigma: float, rules: array[enum], custom_rules: array[object]}` |
| **Output** | `WEAnalysisResponse`: `{violations: array[RuleViolation], stability_status: enum, violation_counts: object, rule_summary: object}` |
| **Execution Mode** | Python/CPU, deterministic, lightweight (< 100MB memory) |
| **Auth** | JWT RS256, role `d1-quality:analyze` |
| **Timeout** | 60s default, 120s max |
| **Error Handling** | `on_error: return manual rule check checklist` + `flag_analysis_degradation` |
| **Example** | `POST /v1/tools/western_electric_analyzer` with `{data_points: [14.3, 15.1, ...], center_line: 14.3, sigma: 3.2, rules: ["WE_1", "WE_2", "WE_3"]}` |

### TOOL-05: ProcessCapabilityCalculator

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-05` |
| **Name** | `ProcessCapabilityCalculator` |
| **Description** | Compute process capability indices Cp, Cpk, Pp, Ppk with confidence intervals (bootstrap), normality tests (Shapiro-Wilk, Anderson-Darling), and capability level classification. Validates against AIAG standards and NumPy reference fixtures. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Measure/Analyze), `ARM-02` (SPC Controller — Capability study) |
| **Input** | `CapabilityRequest`: `{data: array[float], specification_limits: {usl: float, lsl: float}, confidence_level: float (default: 0.95), bootstrap_iterations: int (default: 10000), normality_tests: array[enum]}` |
| **Output** | `CapabilityResponse`: `{cp: float, cpk: float, pp: float, ppk: float, confidence_intervals: object, normality_result: object, capability_level: enum, sample_size: int, recommendation: string}` |
| **Execution Mode** | Python/CPU, scipy/statsmodels, up to 1GB memory for large bootstrap |
| **Auth** | JWT RS256, role `d1-quality:calculate` |
| **Timeout** | 120s default, 300s max |
| **Error Handling** | `on_error: return point estimates without CI` + `flag_uncertainty` + `recommend_more_samples` |
| **Example** | `POST /v1/tools/process_capability` with `{data: [14.3, 15.1, ...], specification_limits: {usl: 20, lsl: 5}, confidence_level: 0.95}` |

### TOOL-06: TestGapAnalyzer

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-06` |
| **Name** | `TestGapAnalyzer` |
| **Description** | Comprehensive coverage gap analysis — parse coverage.xml (and other formats), map coverage to source modules, identify untested branches, paths, and edge cases, and validate gap map against source code AST. Outputs prioritized module list with risk scores. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-03` (Test Gap Analyzer — primary) |
| **Input** | `GapAnalysisRequest`: `{repository_id: string, branch: string, coverage_report_uri: string, source_code_uri: string, report_format: enum, target_coverage: float}` |
| **Output** | `GapAnalysisResponse`: `{analysis_id: UUID, current_coverage: float, gap_modules: array[GapModule], total_gaps: int, risk_summary: object, deliverable_uris: array}` |
| **Execution Mode** | Python/CPU, lxml/pandas, up to 2GB memory for large repos |
| **Auth** | JWT RS256, role `d1-quality:analyze` |
| **Timeout** | 600s default, 1200s max |
| **Error Handling** | `on_error: return high-level gap summary` + `flag_incomplete_analysis` + `recommend_manual_review` |
| **Example** | `POST /v1/tools/test_gap_analyzer` with `{repository_id: "KnowledgeWorker", branch: "main", coverage_report_uri: "s3://coverage.xml", target_coverage: 0.80}` |

### TOOL-07: CoverageOptimizer

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-07` |
| **Name** | `CoverageOptimizer` |
| **Description** | Optimize test suite for maximum coverage with minimum execution cost. Uses genetic algorithm and greedy selection to identify redundant tests, reorder for fast failure, and select minimal subset achieving target coverage. Validates optimized suite preserves mutation score. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-03` (Test Gap Analyzer — Optimization) |
| **Input** | `CoverageOptRequest`: `{test_suite_data: object, target_coverage: float, execution_time_budget: float, optimization_strategy: enum, preserve_mutation_score: bool}` |
| **Output** | `CoverageOptResponse`: `{optimized_suite: object, coverage_achieved: float, execution_time_reduction: float, redundant_tests: array, test_order: array, mutation_score: float}` |
| **Execution Mode** | Python/CPU, DEAP genetic algorithm, up to 2GB memory for large suites |
| **Auth** | JWT RS256, role `d1-quality:optimize` |
| **Timeout** | 300s default, 600s max |
| **Error Handling** | `on_error: return greedy selection result` + `flag_suboptimal` |
| **Example** | `POST /v1/tools/coverage_optimizer` with `{test_suite_data: {tests: [...]}, target_coverage: 0.80, optimization_strategy: "genetic"}` |

### TOOL-08: FlakyTestDetector

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-08` |
| **Name** | `FlakyTestDetector` |
| **Description** | Statistical detection of flaky tests from run history (≥10 runs). Computes pass rate variance, confidence intervals, and binomial distribution tests. Classifies root causes: race condition, timeout misconfiguration, external dependency failure, data contamination, environment variance. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-03` (Test Gap Analyzer — Flaky detection) |
| **Input** | `FlakyDetectRequest`: `{test_suite_id: string, run_history_uri: string, run_count: int, flaky_threshold: float, min_runs: int, classification_model: enum}` |
| **Output** | `FlakyDetectResponse`: `{flaky_tests: array[FlakyTest], total_flaky: int, flaky_rate: float, root_cause_distribution: object, stabilization_recommendations: array}` |
| **Execution Mode** | Python/CPU, scipy statistical tests, lightweight (< 500MB memory) |
| **Auth** | JWT RS256, role `d1-quality:detect` |
| **Timeout** | 180s default, 360s max |
| **Error Handling** | `on_error: return manual inspection checklist` + `require_more_runs` |
| **Example** | `POST /v1/tools/flaky_test_detector` with `{test_suite_id: "suite-123", run_history_uri: "s3://runs.csv", run_count: 50, flaky_threshold: 0.05}` |

### TOOL-09: DefectAnalyzer

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-09` |
| **Name** | `DefectAnalyzer` |
| **Description** | Defect density computation (defects/KLOC), trend analysis, root cause classification (systemic vs. sporadic), and Pareto analysis (80/20 rule). Ingests defect logs from Jira, GitHub Issues, or CSV. Validates DPMO computation against NumPy fixtures. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Analyze), `ARM-04` (Quality Auditor — Defect analysis) |
| **Input** | `DefectAnalysisRequest`: `{defect_log_uri: string, source_code_uri: string, classification_schema: array[enum], analysis_period: object, include_pareto: bool}` |
| **Output** | `DefectAnalysisResponse`: `{defect_density: float, dpmo: int, total_defects: int, kloc: float, by_module: object, by_type: object, pareto_chart_uri: string, trend_direction: enum, forecast: object}` |
| **Execution Mode** | Python/CPU, pandas, up to 1GB memory |
| **Auth** | JWT RS256, role `d1-quality:analyze` |
| **Timeout** | 120s default, 300s max |
| **Error Handling** | `on_error: return raw defect count` + `flag_analysis_degradation` |
| **Example** | `POST /v1/tools/defect_analyzer` with `{defect_log_uri: "s3://defects.csv", source_code_uri: "s3://src/", include_pareto: true}` |

### TOOL-10: QualityScorecard

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-10` |
| **Name** | `QualityScorecard` |
| **Description** | Multi-dimensional quality scorecard generator — coverage, reliability, defect density, process capability, review velocity, review thoroughness, DPMO, sigma level. Computes weighted overall score (0-100) with grade (A-F). Supports configurable weights per audience. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Scorecard), `ARM-04` (Quality Auditor — Scorecard) |
| **Input** | `ScorecardRequest`: `{target_id: string, dimensions: object, weights: object, audience: enum, standards: object, include_trends: bool}` |
| **Output** | `ScorecardResponse`: `{scorecard_id: UUID, overall_score: float, overall_grade: enum, dimension_scores: object, priority_gaps: array, recommendations: array, trend_chart_uri: string}` |
| **Execution Mode** | Python/CPU, jinja2/reportlab, lightweight (< 500MB memory) |
| **Auth** | JWT RS256, role `d1-quality:report` |
| **Timeout** | 60s default, 120s max |
| **Error Handling** | `on_error: return text-only scorecard` + `flag_visual_degradation` |
| **Example** | `POST /v1/tools/quality_scorecard` with `{target_id: "GitSentinental", dimensions: {coverage: 87, test_reliability: 94}, weights: {coverage: 0.20, test_reliability: 0.20}}` |

### TOOL-11: DPMOCalculator

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-11` |
| **Name** | `DPMOCalculator` |
| **Description** | Defects Per Million Opportunities (DPMO) calculator with sigma level translation, confidence intervals, and capability level classification. Validates DPMO-to-sigma translation against standard Six Sigma tables (3.4 DPMO at 6σ, 6210 DPMO at 4σ, etc.). |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Measure), `ARM-04` (Quality Auditor — DPMO) |
| **Input** | `DPMORequest`: `{defects: int, opportunities: int, units: int, confidence_level: float, include_shift: bool (default: true, 1.5σ shift)}` |
| **Output** | `DPMOResponse`: `{dpmo: int, sigma_level: float, sigma_level_with_shift: float, confidence_interval: object, capability_level: enum, classification: string}` |
| **Execution Mode** | Python/CPU, scipy, deterministic (R-GATE-2), lightweight (< 100MB memory) |
| **Auth** | JWT RS256, role `d1-quality:calculate` |
| **Timeout** | 30s default, 60s max |
| **Error Handling** | `on_error: return manual DPMO formula reference` + `flag_computation_failure` |
| **Example** | `POST /v1/tools/dpmo_calculator` with `{defects: 14, opportunities: 10, units: 117000}` |

### TOOL-12: CodeReviewAuditor

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-12` |
| **Name** | `CodeReviewAuditor` |
| **Description** | Code review practice assessment — review velocity (median hours to first review, median hours to approve), review thoroughness (mean comments per PR, review depth score), reviewer distribution (load balancing, expertise matching, Gini coefficient), and correlation with defect escape rate. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-04` (Quality Auditor — Review audit) |
| **Input** | `ReviewAuditRequest`: `{repository_id: string, platform: enum, auth_token: string (Vault ref), analysis_period: object, include_correlation: bool}` |
| **Output** | `ReviewAuditResponse`: `{audit_id: UUID, total_prs: int, reviewed_prs: int, velocity: object, thoroughness: object, reviewer_distribution: object, correlation_with_defects: object, recommendations: array}` |
| **Execution Mode** | FastAPI/DB, Git platform API queries, up to 1GB memory for large repos |
| **Auth** | JWT RS256 + platform OAuth, role `d1-quality:audit` |
| **Timeout** | 120s default, 300s max |
| **Error Handling** | `on_error: return basic review counts` + `flag_api_degradation` |
| **Example** | `POST /v1/tools/code_review_auditor` with `{repository_id: "GitSentinental", platform: "github", analysis_period: {start: "2026-04-01", end: "2026-06-30"}}` |

### TOOL-13: TestReliabilityAnalyzer

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-13` |
| **Name** | `TestReliabilityAnalyzer` |
| **Description** | Test reliability index computation — pass rate variance, confidence intervals, binomial proportion tests, and reliability trend analysis. Supports per-suite, per-module, and per-test granularity. Correlates reliability with code churn and module complexity. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-03` (Test Gap Analyzer — Reliability), `ARM-04` (Quality Auditor — Test reliability) |
| **Input** | `ReliabilityRequest`: `{test_data_uri: string, granularity: enum, run_count: int, confidence_level: float, include_correlation: bool}` |
| **Output** | `ReliabilityResponse`: `{reliability_index: float, confidence_interval: object, by_module: object, by_suite: object, trend_direction: enum, correlation_with_churn: object, recommendations: array}` |
| **Execution Mode** | Python/CPU, scipy statistical tests, up to 1GB memory |
| **Auth** | JWT RS256, role `d1-quality:analyze` |
| **Timeout** | 180s default, 360s max |
| **Error Handling** | `on_error: return pass rate only` + `flag_incomplete_analysis` |
| **Example** | `POST /v1/tools/test_reliability_analyzer` with `{test_data_uri: "s3://test_results.json", granularity: "per_module", run_count: 50}` |

### TOOL-14: SigmaLevelTracker

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-14` |
| **Name** | `SigmaLevelTracker` |
| **Description** | Historical sigma level tracking with trend projection using ARIMA, exponential smoothing, and linear regression. Forecasts sigma level attainment dates, detects regression warnings, and generates improvement velocity metrics. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Track), `ARM-04` (Quality Auditor — Trend) |
| **Input** | `SigmaTrackRequest`: `{target_id: string, historical_data_uri: string, projection_method: enum, forecast_horizon: int, alert_on_regression: bool}` |
| **Output** | `SigmaTrackResponse`: `{track_id: UUID, current_sigma: float, historical_series: array, trend_direction: enum, forecast: array, projected_attainment_date: string, regression_alerts: array}` |
| **Execution Mode** | Python/CPU, statsmodels, up to 1GB memory |
| **Auth** | JWT RS256, role `d1-quality:track` |
| **Timeout** | 60s default, 120s max |
| **Error Handling** | `on_error: return current sigma only` + `flag_forecast_unavailable` |
| **Example** | `POST /v1/tools/sigma_level_tracker` with `{target_id: "pipeline-alpha", historical_data_uri: "s3://sigma_history.csv", projection_method: "arima", forecast_horizon: 90}` |

### TOOL-15: ImprovementPlanGenerator

| Field | Value |
|-------|-------|
| **Tool ID** | `TOOL-15` |
| **Name** | `ImprovementPlanGenerator` |
| **Description** | Prioritized improvement plan generator with effort estimates, expected impact, ROI computation, and phased roadmap. Uses Pareto optimization to select highest-impact, lowest-effort improvements. Generates control plan for each improvement item. |
| **Owner** | D1 The Quality Engineer |
| **Arm Binding** | `ARM-01` (Six Sigma Assessor — Improve/Control), `ARM-04` (Quality Auditor — Improvement planning) |
| **Input** | `ImprovementPlanRequest`: `{target_id: string, findings: array, constraints: object, historical_effort_data_uri: string, max_effort_days: float, min_roi: float}` |
| **Output** | `ImprovementPlanResponse`: `{plan_id: UUID, improvements: array[ImprovementItem], total_effort_days: float, expected_impact: object, roi: float, phased_roadmap: object, control_plan: object, deliverable_uris: array}` |
| **Execution Mode** | Python/CPU + LLM (narrative), up to 1GB memory |
| **Auth** | JWT RS256, role `d1-quality:plan` |
| **Timeout** | 300s default, 600s max |
| **Error Handling** | `on_error: return generic improvement checklist` + `flag_customization_unavailable` |
| **Example** | `POST /v1/tools/improvement_plan_generator` with `{target_id: "pipeline-alpha", findings: [{dimension: "test_reliability", current: 83, target: 97}], max_effort_days: 20, min_roi: 2.0}` |

---

## 3. Tool-to-Arm Mapping Matrix

| Tool | ARM-01 | ARM-02 | ARM-03 | ARM-04 | Execution |
|------|--------|--------|--------|--------|-----------|
| TOOL-01 SixSigmaAssessor | ✅ Core | ❌ | ❌ | ❌ | Python/CPU |
| TOOL-02 SPC_Controller | ✅ Aux (Control) | ✅ Core | ❌ | ❌ | Python/CPU |
| TOOL-03 ControlChartGenerator | ✅ Aux (Measure/Control) | ✅ Core | ❌ | ❌ | Python/CPU |
| TOOL-04 WesternElectricAnalyzer | ❌ | ✅ Core | ❌ | ❌ | Python/CPU |
| TOOL-05 ProcessCapabilityCalculator | ✅ Core | ✅ Aux | ❌ | ❌ | Python/CPU |
| TOOL-06 TestGapAnalyzer | ❌ | ❌ | ✅ Core | ❌ | Python/CPU |
| TOOL-07 CoverageOptimizer | ❌ | ❌ | ✅ Core | ❌ | Python/CPU |
| TOOL-08 FlakyTestDetector | ❌ | ❌ | ✅ Core | ❌ | Python/CPU |
| TOOL-09 DefectAnalyzer | ✅ Core | ❌ | ❌ | ✅ Core | Python/CPU |
| TOOL-10 QualityScorecard | ✅ Core | ❌ | ❌ | ✅ Core | Python/CPU |
| TOOL-11 DPMOCalculator | ✅ Core | ❌ | ❌ | ✅ Core | Python/CPU |
| TOOL-12 CodeReviewAuditor | ❌ | ❌ | ❌ | ✅ Core | FastAPI/DB |
| TOOL-13 TestReliabilityAnalyzer | ❌ | ❌ | ✅ Core | ✅ Aux | Python/CPU |
| TOOL-14 SigmaLevelTracker | ✅ Core | ❌ | ❌ | ✅ Aux | Python/CPU |
| TOOL-15 ImprovementPlanGenerator | ✅ Core | ❌ | ❌ | ✅ Core | Python/CPU+LLM |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team
**Next Review:** 2026-07-28
**Classification:** Internal — Tool Registry
