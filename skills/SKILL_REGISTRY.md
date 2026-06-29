# SKILL_REGISTRY.md
## Persona D1 — The Quality Engineer | Skill Definitions

**Version:** 1.0.0
**Status:** Production-ready
**Date:** 2026-06-28
**Owner:** D1 The Quality Engineer
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`
**Master Strategy:** `C:\KimiWork Projects\GAI-OBSERVE-DESIGN\skills-hooks-plugins-strategy\STRATEGY.md`

---

## 1. Skill Registry Overview

All skills leveraged by D1 arms are defined in this document. Each skill has a trigger, input/output contract, procedure, quality gates, error handling, evidence requirements, and usage examples. Skills are mapped from the master strategy's skill registry to D1-specific use cases. All 10 skills are registered: SixSigmaAssessment, StatisticalProcessControl, TestGapAnalysis, CoverageOptimization, FlakyTestDetection, DefectAnalysis, QualityAuditing, ProcessCapabilityStudy, CodeReviewAudit, and QualityImprovementPlanning.

---

## 2. Skill Definitions

### SixSigmaAssessment

```yaml
skill:
  name: "SixSigmaAssessment"
  description: "Execute full DMAIC (Define, Measure, Analyze, Improve, Control) assessment with evidence gates and phase transitions. Compute sigma levels from DPMO, process capability indices (Cp, Cpk, Pp, Ppk), and generate improvement roadmaps with control plans."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_quarterly OR on_demand_quality_assessment OR event_quality_gate_failure OR chained_from_d9"
  input:
    schema: "SixSigmaAssessRequest"
    fields:
      target_id: "string (repository, process, or system ID)"
      target_type: "enum [repository, process, pipeline, module, system]"
      assessment_scope: "enum [full_dmaic, measure_only, capability_study, control_plan]"
      data_uris: "object (paths to defect logs, test results, coverage reports, process metrics)"
      standards: "object (target_coverage, target_defect_density, target_sigma_level, target_cpk)"
      historical_baseline_id: "UUID (optional)"
  output:
    schema: "SixSigmaAssessResponse"
    fields:
      assessment_id: "UUID"
      phases: "object (define, measure, analyze, improve, control status and evidence)"
      sigma_level: "float"
      dpmo: "int"
      cpk: "float"
      overall_grade: "enum [A, B, C, D, F]"
      improvement_plan: "object"
      deliverable_uris: "array[string]"
  procedure:
    - "Define: Scope boundaries, identify CTQs (Critical to Quality), and document customer requirements"
    - "Measure: Collect baseline metrics — coverage, defect density, test reliability, build time, review velocity"
    - "Measure: Validate measurement system (MSA) — repeatability, reproducibility, calibration"
    - "Measure: Compute DPMO and translate to sigma level"
    - "Analyze: Run statistical analysis — control charts, Pareto, correlation, regression"
    - "Analyze: Identify root causes with 5 Whys, fishbone diagram, and hypothesis testing"
    - "Analyze: Verify root causes with statistical tests (p < 0.05)"
    - "Improve: Generate prioritized improvement actions with effort and impact estimates"
    - "Improve: Run pilot improvements and validate with before/after comparison"
    - "Control: Establish control plan with monitoring metrics, response plans, and ownership"
    - "Control: Set control limits, deploy SPC monitoring, and train stakeholders"
  quality_gates:
    - "Phase evidence: Every phase transition requires ≥3 evidence items"
    - "Statistical validation: All root cause claims backed by p < 0.05"
    - "Sigma accuracy: DPMO-to-sigma translation matches standard tables (3.4 at 6σ)"
    - "Capability validation: Cpk computation matches AIAG standards and NumPy fixtures"
    - "R-ARM-DMAIC-4: Phase transition without evidence is blocked with explicit reason"
  error_handling:
    on_missing_evidence: "block phase transition and request specific evidence items"
    on_insufficient_data: "return partial assessment with data gaps flagged"
    on_statistical_failure: "retry with non-parametric methods; flag assumptions violated"
    on_p3_rejection: "revise claim with corrected evidence and resubmit"
  evidence:
    - "DMAIC report JSON with phase evidence"
    - "Sigma scorecard PDF"
    - "Control plan Markdown"
    - "Improvement roadmap Gantt JSON"
    - "P2 ledger hash"
  example:
    request: "{target_id: 'pipeline-alpha', target_type: 'pipeline', assessment_scope: 'full_dmaic', data_uris: {defect_logs: 's3://defects.csv'}, standards: {target_sigma: 4.0, target_cpk: 1.33}}"
    response: "{assessment_id: 'dmaic-pipeline-001', phases: {define: 'complete', measure: 'complete', analyze: 'complete', improve: 'complete', control: 'complete'}, sigma_level: 4.1, dpmo: 3400, cpk: 1.45, overall_grade: 'A-', improvement_plan: {actions: 7, effort_days: 14, expected_impact: 'sigma 4.1 -> 4.5'}}"
```

### StatisticalProcessControl

```yaml
skill:
  name: "StatisticalProcessControl"
  description: "Complete SPC workflow — control chart selection, limit computation, Western Electric rule evaluation, and process stability classification. Supports all standard chart types and all 8 Western Electric rules with deterministic validation."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_spc_scan OR real_time_metric_stream OR on_demand_stability_check OR event_threshold_breach"
  input:
    schema: "SPCRequest"
    fields:
      target_id: "string"
      metric_name: "enum [build_time, test_duration, defect_count, coverage, error_rate, latency, throughput]"
      chart_type: "enum [xbar, r, s, x, mr, p, np, c, u]"
      data_points: "array[float]"
      specification_limits: "object {usl, lsl} (optional)"
      subgroup_size: "int (default: 1)"
      rule_suite: "array[enum] (default: all 8)"
  output:
    schema: "SPCResponse"
    fields:
      chart_id: "UUID"
      center_line: "float"
      ucl: "float"
      lcl: "float"
      rule_violations: "array[RuleViolation]"
      stability_status: "enum [STABLE, UNSTABLE, TRENDING, CYCLICAL, UNKNOWN]"
      capability_indices: "object (optional)"
      alert_required: "boolean"
      chart_uri: "string (optional)"
  procedure:
    - "Select control chart type based on data type (continuous vs. attribute) and subgroup size"
    - "Compute center line (mean or median) and control limits (±3σ)"
    - "Validate control limits against specification limits if provided"
    - "Plot data points and identify patterns (trends, cycles, shifts)"
    - "Apply all 8 Western Electric rules to control chart data"
    - "Classify process stability based on rule violations and pattern analysis"
    - "Compute process capability indices (Cp, Cpk) if specification limits available"
    - "Generate control chart image with rule annotations"
    - "Dispatch alerts if critical rule violations detected"
    - "Update SPC dashboard with new data points and stability status"
  quality_gates:
    - "Chart type match: selected chart type matches data characteristics"
    - "Control limit accuracy: limits within ±3σ, validated against NumPy fixtures"
    - "Rule completeness: all 8 Western Electric rules implemented and evaluated"
    - "Deterministic core: R-GATE-2 — same input produces same output"
    - "Pattern accuracy: stability classification matches expert assessment >90%"
  error_handling:
    on_insufficient_data: "require minimum data points (n≥20 for X, n≥10 for X-bar/R)"
    on_non_normal_data: "apply transformation or use non-parametric limits"
    on_chart_type_mismatch: "recommend appropriate chart type and restart"
    on_control_limit_failure: "fallback to specification limits or historical limits"
  evidence:
    - "Control chart image with rule annotations"
    - "Rule violation report JSON"
    - "Process stability verdict"
    - "Capability study (if applicable)"
    - "P2 ledger hash"
  example:
    request: "{target_id: 'pipeline-alpha', metric_name: 'build_time', chart_type: 'x', data_points: [14.3, 15.1, 13.8, ...], specification_limits: {usl: 20, lsl: 5}}"
    response: "{chart_id: 'uuid', center_line: 14.3, ucl: 23.9, lcl: 4.7, rule_violations: [{rule: 'WE_1', point: 42, value: 28.5}], stability_status: 'UNSTABLE', alert_required: true}"
```

### TestGapAnalysis

```yaml
skill:
  name: "TestGapAnalysis"
  description: "Comprehensive coverage gap analysis with prioritized test plan generation. Ingests coverage reports, maps gaps to source code AST, identifies untested branches/paths/edge cases, and produces risk-scored test plans with effort estimates."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_nightly OR ci_pipeline_completion OR on_demand_repo_check OR chained_from_d9"
  input:
    schema: "TestGapRequest"
    fields:
      repository_id: "string"
      branch: "string"
      coverage_report_uri: "string (S3/MinIO)"
      source_code_uri: "string (S3/MinIO)"
      report_format: "enum [coverage.xml, lcov, jacoco, cobertura, clover]"
      target_coverage: "float (default: 0.80)"
      include_branch_coverage: "boolean (default: true)"
      include_mutation_score: "boolean (default: false)"
  output:
    schema: "TestGapResponse"
    fields:
      analysis_id: "UUID"
      current_coverage: "float (line)"
      current_branch_coverage: "float"
      gap_modules: "array[GapModule]"
      total_gaps: "int"
      risk_summary: "object"
      test_plan: "object (prioritized tests, effort estimates, expected coverage)"
      flaky_tests_detected: "int"
  procedure:
    - "Ingest coverage report and validate format"
    - "Map coverage metrics to source code modules and files"
    - "Parse source code AST to identify branches, conditions, and edge cases"
    - "Cross-reference coverage with AST to identify untested branches and paths"
    - "Compute risk scores per module (complexity × security × data integrity)"
    - "Detect flaky tests from run history if available"
    - "Generate prioritized test plan with effort estimates and expected coverage"
    - "Validate test plan against target coverage and flag unattainable targets"
    - "Generate coverage gap map visualization"
    - "Update coverage dashboard and quality scorecard"
  quality_gates:
    - "Coverage accuracy: gap map matches coverage.xml exactly (R-ARM-REPRO-1)"
    - "AST validation: untested branches identified with file and line numbers"
    - "Risk scoring: all risk scores between 0 and 10 with documented factors"
    - "Effort accuracy: estimates within ±20% of historical data"
    - "Target feasibility: expected coverage after implementation ≥ target"
  error_handling:
    on_invalid_coverage_format: "attempt fallback parsers; flag unsupported format"
    on_ast_parsing_failure: "fallback to line-level gaps with manual review flag"
    on_insufficient_run_history: "require minimum 10 runs for flaky detection"
    on_target_unattainable: "flag with recommended achievable target"
  evidence:
    - "Coverage gap map JSON and HTML"
    - "Prioritized test plan Markdown and Excel"
    - "Flaky test report (if applicable)"
    - "Risk score justification per module"
    - "P2 ledger hash"
  example:
    request: "{repository_id: 'KnowledgeWorker', branch: 'main', coverage_report_uri: 's3://coverage.xml', target_coverage: 0.80}"
    response: "{analysis_id: 'gap-001', current_coverage: 0.71, gap_modules: [{module: 'auth.py', coverage: 0.34, risk_score: 9.2, recommended_tests: 6, effort_days: 2}], test_plan: {total_tests: 13, total_effort: 4, expected_coverage: 0.83}}"
```

### CoverageOptimization

```yaml
skill:
  name: "CoverageOptimization"
  description: "Optimize test suite for maximum coverage with minimum execution cost. Uses genetic algorithm and greedy selection to identify redundant tests, reorder for fast failure, and select minimal subset achieving target coverage while preserving mutation score."
  owner: "D1 The Quality Engineer"
  trigger: "on_demand_optimization OR scheduled_monthly OR event_execution_time_breach"
  input:
    schema: "CoverageOptRequest"
    fields:
      test_suite_id: "string"
      target_coverage: "float"
      execution_time_budget: "float (seconds)"
      optimization_strategy: "enum [genetic, greedy, mixed]"
      preserve_mutation_score: "boolean (default: true)"
      include_tests: "array[string] (required tests to preserve)"
  output:
    schema: "CoverageOptResponse"
    fields:
      optimized_suite_id: "UUID"
      coverage_achieved: "float"
      execution_time_reduction: "float (percentage)"
      redundant_tests: "array[string]"
      test_order: "array[string]"
      mutation_score: "float"
      fault_detection_score: "float"
  procedure:
    - "Ingest test suite data (coverage matrix, execution times, test names)"
    - "Validate coverage matrix completeness and consistency"
    - "Run genetic algorithm (NSGA2) with objectives: max coverage, min time, preserve mutation"
    - "Identify redundant tests that contribute no unique coverage"
    - "Reorder tests for fast failure (high failure probability first)"
    - "Validate optimized suite against target coverage and time budget"
    - "Run mutation testing if preserve_mutation_score=true"
    - "Compare fault detection score before and after optimization"
    - "Generate optimization report with before/after comparison"
  quality_gates:
    - "Coverage maintained: optimized suite achieves target coverage"
    - "Time reduced: execution time reduced by ≥20% or within budget"
    - "Mutation preserved: mutation score maintained within ±5%"
    - "No regression: fault detection score ≥ original"
    - "Deterministic: same input produces same output (R-GATE-2)"
  error_handling:
    on_optimization_timeout: "return best solution found so far with degradation flag"
    on_coverage_unattainable: "flag achievable coverage and recommend additional tests"
    on_mutation_regression: "re-run with stricter constraints or flag manual review"
    on_invalid_matrix: "fallback to greedy algorithm with warning"
  evidence:
    - "Optimized test suite JSON"
    - "Before/after comparison report"
    - "Mutation score comparison"
    - "Execution time benchmark results"
    - "P2 ledger hash"
  example:
    request: "{test_suite_id: 'suite-123', target_coverage: 0.80, execution_time_budget: 300, optimization_strategy: 'genetic'}"
    response: "{optimized_suite_id: 'opt-001', coverage_achieved: 0.82, execution_time_reduction: 0.35, redundant_tests: ['test_a', 'test_b'], mutation_score: 0.78, fault_detection_score: 0.95}"
```

### FlakyTestDetection

```yaml
skill:
  name: "FlakyTestDetection"
  description: "Statistical detection of flaky tests from run history with root cause classification. Requires minimum 10 runs for reliable detection. Computes pass rate variance, binomial confidence intervals, and classifies root causes into race condition, timeout, external dependency, data contamination, or environment variance."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_weekly OR event_ci_failure_pattern OR on_demand_flaky_check"
  input:
    schema: "FlakyDetectRequest"
    fields:
      test_suite_id: "string"
      run_history_uri: "string (S3/MinIO)"
      run_count: "int"
      flaky_threshold: "float (default: 0.05 variance)"
      min_runs: "int (default: 10)"
      classification_model: "enum [statistical_heuristic, ml_classifier]"
  output:
    schema: "FlakyDetectResponse"
    fields:
      flaky_tests: "array[FlakyTest]"
      total_flaky: "int"
      flaky_rate: "float"
      root_cause_distribution: "object"
      stabilization_recommendations: "array[string]"
      confidence: "float"
  procedure:
    - "Ingest test run history (≥10 runs per test)"
    - "Validate data format and completeness"
    - "Compute pass rate per test and variance across runs"
    - "Calculate binomial confidence intervals for pass rates"
    - "Flag tests with variance > threshold as flaky candidates"
    - "Run chi-square test for independence across runs"
    - "Classify root cause based on failure patterns, error messages, and timing"
    - "Generate stabilization recommendations per root cause type"
    - "Validate flaky verdict against R-ARM-REPRO-2 (backed by run statistics)"
    - "Update test reliability scorecard and quality metrics"
  quality_gates:
    - "Minimum runs: ≥10 runs for reliable flaky verdict"
    - "Variance threshold: flaky if variance > 0.05 (configurable)"
    - "Root cause accuracy: classification accuracy >85% on validated dataset"
    - "Statistical backing: all verdicts supported by p-values and confidence intervals"
    - "R-ARM-REPRO-2: flaky verdict backed by run statistics"
  error_handling:
    on_insufficient_runs: "require minimum 10 runs; flag if <10 available"
    on_classification_uncertainty: "flag manual review with confidence score"
    on_data_corruption: "fallback to pass rate only; flag data quality issue"
    on_ml_classifier_unavailable: "fallback to statistical heuristic"
  evidence:
    - "Flaky test report with run statistics"
    - "Root cause classification with confidence scores"
    - "Stabilization recommendations per test"
    - "P2 ledger hash"
  example:
    request: "{test_suite_id: 'suite-123', run_history_uri: 's3://runs.csv', run_count: 50, flaky_threshold: 0.05}"
    response: "{flaky_tests: [{test_name: 'test_token_refresh', pass_rate: 0.72, variance: 0.18, root_cause: 'race_condition', confidence: 0.91}], total_flaky: 3, flaky_rate: 0.06}"
```

### DefectAnalysis

```yaml
skill:
  name: "DefectAnalysis"
  description: "Defect density computation, trend analysis, root cause classification, and Pareto analysis. Computes defects per KLOC, DPMO, and classifies defects as systemic vs. sporadic. Identifies top modules and types via 80/20 Pareto analysis."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_monthly OR event_defect_spike OR on_demand_defect_analysis"
  input:
    schema: "DefectAnalysisRequest"
    fields:
      defect_log_uri: "string (S3/MinIO)"
      source_code_uri: "string (S3/MinIO)"
      classification_schema: "array[enum] (type, severity, module, root_cause, detection_phase)"
      analysis_period: "object {start, end}"
      include_pareto: "boolean (default: true)"
      include_trend_forecast: "boolean (default: true)"
  output:
    schema: "DefectAnalysisResponse"
    fields:
      defect_density: "float (defects/KLOC)"
      dpmo: "int"
      total_defects: "int"
      kloc: "float"
      by_module: "object"
      by_type: "object"
      pareto_chart_uri: "string"
      trend_direction: "enum [improving, stable, declining, volatile]"
      forecast: "object (optional)"
      systemic_defects: "array[string]"
  procedure:
    - "Ingest defect logs from issue tracker or CSV"
    - "Classify defects by type, severity, module, and root cause"
    - "Ingest source code and compute KLOC per module"
    - "Compute defect density per module and overall"
    - "Compute DPMO from defect count and opportunities"
    - "Run Pareto analysis (80/20) for modules and defect types"
    - "Identify systemic vs. sporadic defects using frequency and pattern analysis"
    - "Run trend analysis with time-series decomposition"
    - "Forecast defect trend using ARIMA or exponential smoothing"
    - "Generate Pareto chart and trend visualization"
    - "Update defect scorecard and quality metrics"
  quality_gates:
    - "KLOC accuracy: source code lines counted consistently with standard tool"
    - "Pareto validation: top 20% of categories account for ≥80% of defects"
    - "Trend significance: trend direction statistically significant (p < 0.05)"
    - "DPMO accuracy: DPMO computation validated against NumPy fixtures"
    - "Systemic identification: systemic defects confirmed by ≥3 occurrences in same module/type"
  error_handling:
    on_missing_source_code: "compute density from total KLOC estimate; flag uncertainty"
    on_classification_ambiguity: "flag manual review; use default classification"
    on_insufficient_history: "skip trend forecast; provide current snapshot only"
    on_data_format_error: "attempt fallback parsers; flag unsupported format"
  evidence:
    - "Defect density report with module breakdown"
    - "Pareto chart PNG"
    - "Trend analysis with forecast"
    - "Systemic defect identification with evidence"
    - "P2 ledger hash"
  example:
    request: "{defect_log_uri: 's3://defects.csv', source_code_uri: 's3://src/', include_pareto: true, include_trend_forecast: true}"
    response: "{defect_density: 0.12, dpmo: 4500, total_defects: 14, kloc: 117, by_module: {auth.py: 4, api.py: 3}, pareto_chart_uri: 's3://pareto.png', trend_direction: 'improving', systemic_defects: ['auth_bypass']}"
```

### QualityAuditing

```yaml
skill:
  name: "QualityAuditing"
  description: "Multi-dimensional quality audit with scorecard generation. Evaluates coverage, test reliability, defect density, process capability, review velocity, review thoroughness, DPMO, and sigma level. Computes weighted overall score (0-100) with grade (A-F). Supports configurable weights per audience."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_monthly OR event_pre_release OR on_demand_scorecard OR chained_from_g1"
  input:
    schema: "QualityAuditRequest"
    fields:
      target_id: "string"
      target_type: "enum [repository, team, project, system]"
      dimensions: "object (per-dimension metric values)"
      weights: "object (optional, per-dimension weights)"
      audience: "enum [executive, technical, regulatory, board]"
      standards: "object (per-dimension targets)"
      include_trends: "boolean (default: true)"
  output:
    schema: "QualityAuditResponse"
    fields:
      audit_id: "UUID"
      overall_score: "float (0-100)"
      overall_grade: "enum [A, B, C, D, F]"
      dimension_scores: "object"
      priority_gaps: "array[string]"
      recommendations: "array[string]"
      certification_recommendation: "enum [CERTIFIED, CONDITIONAL, NOT_CERTIFIED]"
      trend_chart_uri: "string"
  procedure:
    - "Collect quality metrics across all dimensions"
    - "Validate metric values are within valid ranges"
    - "Normalize metrics to 0-100 scale using target-based normalization"
    - "Apply audience-appropriate weights (executive: balanced, technical: coverage/reliability heavy)"
    - "Compute weighted overall score"
    - "Assign grade based on standard scale (A≥90, B≥80, C≥70, D≥60, F<60)"
    - "Identify priority gaps (dimensions below target with highest weight)"
    - "Generate actionable recommendations per gap"
    - "Determine certification recommendation based on grade and critical gaps"
    - "Generate trend chart if historical data available"
    - "Update quality scorecard dashboard"
  quality_gates:
    - "Transparency: score computation formula documented and reproducible"
    - "Weight validity: all weights sum to 1.0, each between 0 and 1"
    - "Metric validity: all sub-metrics within plausible ranges"
    - "Grade consistency: same score always yields same grade"
    - "Audience fit: technical depth matches specified audience"
  error_handling:
    on_missing_dimensions: "fallback to unweighted average of available dimensions; flag gaps"
    on_invalid_weights: "fallback to default weights for audience; flag override"
    on_grade_contest: "escalate to G1 with full computation trace"
    on_insufficient_history: "skip trend analysis; provide current snapshot"
  evidence:
    - "Quality scorecard PDF and Excel"
    - "Dimension score breakdown with formulas"
    - "Trend chart (if applicable)"
    - "Certification recommendation with justification"
    - "P2 ledger hash"
  example:
    request: "{target_id: 'GitSentinental', target_type: 'repository', dimensions: {coverage: 87, test_reliability: 94, defect_density: 0.12, cpk: 1.2, review_velocity: 2.3, review_thoroughness: 4.2}, audience: 'technical'}"
    response: "{audit_id: 'audit-001', overall_score: 87, overall_grade: 'B+', dimension_scores: {coverage: 87, test_reliability: 94, defect_density: 92, cpk: 78, review_velocity: 88, review_thoroughness: 95}, priority_gaps: ['test_reliability', 'cpk'], certification_recommendation: 'CONDITIONAL'}"
```

### ProcessCapabilityStudy

```yaml
skill:
  name: "ProcessCapabilityStudy"
  description: "Statistical process capability study (Cp, Cpk, Pp, Ppk) with normality tests, confidence intervals, and capability level classification. Validates against AIAG standards and requires minimum 30 samples for reliable Cpk computation."
  owner: "D1 The Quality Engineer"
  trigger: "on_demand_capability_study OR scheduled_quarterly OR event_specification_breach"
  input:
    schema: "CapabilityStudyRequest"
    fields:
      target_id: "string"
      metric_name: "string"
      data: "array[float]"
      specification_limits: "object {usl, lsl}"
      confidence_level: "float (default: 0.95)"
      bootstrap_iterations: "int (default: 10000)"
      include_normality_tests: "boolean (default: true)"
  output:
    schema: "CapabilityStudyResponse"
    fields:
      study_id: "UUID"
      cp: "float"
      cpk: "float"
      pp: "float"
      ppk: "float"
      confidence_intervals: "object"
      normality_result: "object"
      capability_level: "enum [excellent, capable, marginally_capable, incapable, critical]"
      sample_size: "int"
      recommendation: "string"
      deliverable_uri: "string"
  procedure:
    - "Collect process data (minimum 30 samples for reliable Cpk)"
    - "Validate data completeness and remove outliers using IQR method"
    - "Run normality tests (Shapiro-Wilk, Anderson-Darling, Kolmogorov-Smirnov)"
    - "If non-normal, apply Box-Cox transformation or use non-parametric methods"
    - "Compute Cp and Cpk using standard formulas"
    - "Compute Pp and Ppk using overall standard deviation"
    - "Generate bootstrap confidence intervals (10,000 iterations)"
    - "Classify capability level based on Cpk value"
    - "Compare to specification limits and identify gaps"
    - "Generate capability study report with control charts"
    - "Provide improvement recommendations if Cpk < 1.33"
  quality_gates:
    - "Sample size: n ≥ 30 for reliable Cpk; flag if n < 30"
    - "Normality tests: all tests implemented correctly; p-values accurate"
    - "Cpk accuracy: computation matches AIAG standards; validated against NumPy fixtures"
    - "CI validity: bootstrap confidence intervals achieve specified coverage"
    - "Classification consistency: same Cpk always yields same capability level"
  error_handling:
    on_insufficient_samples: "require minimum 30 samples; flag uncertainty if n < 30"
    on_non_normal_data: "apply Box-Cox transformation; fallback to Pp/Ppk if transformation fails"
    on_specification_limits_missing: "flag requirement; compute Cp/Pp only if one-sided"
    on_computation_failure: "fallback to point estimates without CI; flag uncertainty"
  evidence:
    - "Capability study PDF with control charts"
    - "Normality test results with p-values"
    - "Bootstrap confidence interval data"
    - "P2 ledger hash"
  example:
    request: "{target_id: 'pipeline-alpha', metric_name: 'build_time', data: [14.3, 15.1, ...], specification_limits: {usl: 20, lsl: 5}, confidence_level: 0.95}"
    response: "{study_id: 'cap-001', cp: 1.1, cpk: 0.85, pp: 1.05, ppk: 0.82, confidence_intervals: {cpk: [0.72, 0.98]}, normality_result: {test: 'shapiro_wilk', p_value: 0.12, is_normal: true}, capability_level: 'marginally_capable', recommendation: 'Improve process centering to increase Cpk to 1.33'}"
```

### CodeReviewAudit

```yaml
skill:
  name: "CodeReviewAudit"
  description: "Assessment of code review practices and their quality impact. Computes review velocity (median time to first review and approval), review thoroughness (mean comments per PR), reviewer distribution (Gini coefficient for load balancing), and correlates review metrics with defect escape rate."
  owner: "D1 The Quality Engineer"
  trigger: "scheduled_weekly OR event_review_backlog OR on_demand_review_audit"
  input:
    schema: "ReviewAuditRequest"
    fields:
      repository_id: "string"
      platform: "enum [github, gitlab, bitbucket]"
      auth_token: "string (Vault reference)"
      analysis_period: "object {start, end}"
      include_correlation: "boolean (default: true)"
      reviewer_anonymization: "boolean (default: true)"
  output:
    schema: "ReviewAuditResponse"
    fields:
      audit_id: "UUID"
      total_prs: "int"
      reviewed_prs: "int"
      velocity: "object"
      thoroughness: "object"
      reviewer_distribution: "object"
      correlation_with_defects: "object"
      recommendations: "array[string]"
      deliverable_uri: "string"
  procedure:
    - "Ingest PR/MR data from Git platform API"
    - "Validate data completeness and filter out bot/ automated PRs"
    - "Compute review velocity metrics (median time to first review, median time to approve)"
    - "Compute review thoroughness metrics (mean comments per PR, review depth score)"
    - "Analyze reviewer distribution (load balancing, expertise matching, Gini coefficient)"
    - "Ingest defect escape data and correlate with review metrics"
    - "Identify review bottlenecks and quality gaps"
    - "Generate recommendations for review practice improvement"
    - "Anonymize reviewer data if configured"
    - "Generate review audit report with visualizations"
    - "Update quality scorecard with review dimension"
  quality_gates:
    - "Data accuracy: metrics match Git platform API response exactly"
    - "Velocity median: use median not mean to avoid outlier skew"
    - "Correlation significance: p < 0.05 for correlation claims"
    - "Anonymity preservation: no reviewer names in external reports if anonymization enabled"
    - "Completeness: all PRs in period analyzed; flag if API pagination incomplete"
  error_handling:
    on_api_failure: "retry with exponential backoff; fallback to cached data with staleness warning"
    on_insufficient_prs: "flag sample size < 30; provide metrics with uncertainty intervals"
    on_correlation_failure: "skip correlation analysis; provide descriptive statistics only"
    on_anonymization_failure: "abort external report generation; flag for manual review"
  evidence:
    - "Review audit report with metrics and visualizations"
    - "Correlation analysis with p-values"
    - "Reviewer distribution chart (anonymized)"
    - "P2 ledger hash"
  example:
    request: "{repository_id: 'GitSentinental', platform: 'github', analysis_period: {start: '2026-04-01', end: '2026-06-30'}, include_correlation: true}"
    response: "{audit_id: 'review-001', total_prs: 142, reviewed_prs: 138, velocity: {median_first_review_hours: 18, median_approve_hours: 55}, thoroughness: {mean_comments_per_pr: 4.2}, reviewer_distribution: {gini_coefficient: 0.34}, correlation_with_defects: {pearson_r: -0.42, p_value: 0.003}, recommendations: ['Add reviewer rotation schedule', 'Set SLA for first review within 24h']}"
```

### QualityImprovementPlanning

```yaml
skill:
  name: "QualityImprovementPlanning"
  description: "Generate prioritized improvement plans with effort estimates, expected impact, and ROI computation. Uses Pareto optimization to select highest-impact, lowest-effort improvements. Creates phased roadmaps with milestones and integrates with D3 Delivery Captain for scheduling."
  owner: "D1 The Quality Engineer"
  trigger: "event_audit_complete OR on_demand_improvement_plan OR chained_from_g1"
  input:
    schema: "ImprovementPlanRequest"
    fields:
      target_id: "string"
      findings: "array[Finding]"
      constraints: "object (budget, timeline, resources)"
      historical_effort_data_uri: "string (S3/MinIO)"
      max_effort_days: "float"
      min_roi: "float (default: 1.5)"
      include_control_plan: "boolean (default: true)"
  output:
    schema: "ImprovementPlanResponse"
    fields:
      plan_id: "UUID"
      improvements: "array[ImprovementItem]"
      total_effort_days: "float"
      expected_impact: "object"
      roi: "float"
      phased_roadmap: "object"
      control_plan: "object"
      deliverable_uris: "array[string]"
  procedure:
    - "Consolidate findings from all quality analyses (audit, gap, SPC, defect)"
    - "Score each improvement by effort (days), impact (score improvement), and risk (0-1)"
    - "Run Pareto optimization to find optimal improvement set"
    - "Filter by constraints (max effort, min ROI, resource availability)"
    - "Group improvements into phases with dependencies and milestones"
    - "Estimate effort using historical median with 1.3× buffer"
    - "Compute expected impact on each quality dimension"
    - "Calculate overall ROI (expected benefit / effort cost)"
    - "Generate control plan for each improvement item"
    - "Create Gantt-style phased roadmap"
    - "Integrate with D3 Delivery Captain for scheduling"
    - "Generate improvement plan report with before/after projections"
  quality_gates:
    - "Effort accuracy: estimates within ±30% of historical data"
    - "Impact completeness: plan covers all fail/marginal dimensions"
    - "ROI positive: all recommended improvements have ROI ≥ min_roi"
    - "Milestone feasibility: all phases achievable within constraints"
    - "Control plan completeness: each improvement has monitoring and response plan"
  error_handling:
    on_missing_historical_data: "use industry standard estimates; flag higher uncertainty"
    on_constraint_violation: "flag infeasible improvements; recommend constraint relaxation"
    on_roi_negative: "exclude improvement; explain why not recommended"
    on_scheduling_failure: "fallback to unscheduled plan with dependency graph"
  evidence:
    - "Improvement plan PDF with phased roadmap"
    - "Pareto optimization trace"
    - "Control plan per improvement item"
    - "Effort estimation justification"
    - "P2 ledger hash"
  example:
    request: "{target_id: 'pipeline-alpha', findings: [{dimension: 'test_reliability', current: 83, target: 97}, {dimension: 'cpk', current: 0.85, target: 1.33}], max_effort_days: 20, min_roi: 2.0}"
    response: "{plan_id: 'imp-001', improvements: [{item: 'Fix flaky tests in auth module', effort_days: 3, expected_impact: {test_reliability: +8}, roi: 3.5}, {item: 'Add mock external dependencies', effort_days: 2, expected_impact: {test_reliability: +6}, roi: 4.0}], total_effort_days: 14, expected_impact: {test_reliability: 97, cpk: 1.45}, roi: 3.2, phased_roadmap: {phase_1: 'Week 1: Fix flaky tests', phase_2: 'Week 2: Add mocks and recompute capability'}}"
```

---

## 3. Skill-to-Arm Mapping

| Skill | ARM-01 | ARM-02 | ARM-03 | ARM-04 | Primary Use |
|-------|--------|--------|--------|--------|-------------|
| SixSigmaAssessment | ✅ Core | ❌ | ❌ | ✅ Aux | DMAIC assessments, sigma scoring, control plans |
| StatisticalProcessControl | ✅ Aux (Control) | ✅ Core | ❌ | ❌ | Control charts, Western Electric rules, stability |
| TestGapAnalysis | ❌ | ❌ | ✅ Core | ❌ | Coverage gaps, test plans, risk scoring |
| CoverageOptimization | ❌ | ❌ | ✅ Core | ❌ | Test suite optimization, redundancy removal |
| FlakyTestDetection | ❌ | ❌ | ✅ Core | ❌ | Flaky test detection, root cause classification |
| DefectAnalysis | ✅ Core | ❌ | ❌ | ✅ Core | Defect density, Pareto, trends, DPMO |
| QualityAuditing | ✅ Core | ❌ | ❌ | ✅ Core | Scorecard generation, multi-dimensional grading |
| ProcessCapabilityStudy | ✅ Core | ✅ Aux | ❌ | ❌ | Cp/Cpk/Pp/Ppk computation, capability classification |
| CodeReviewAudit | ❌ | ❌ | ❌ | ✅ Core | Review velocity, thoroughness, distribution |
| QualityImprovementPlanning | ✅ Core | ❌ | ❌ | ✅ Core | Prioritized plans, effort estimates, ROI, roadmaps |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team
**Next Review:** 2026-07-28
**Classification:** Internal — Skill Registry
