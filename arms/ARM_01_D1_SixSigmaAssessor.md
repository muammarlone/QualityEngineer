# ARM_01_D1_SixSigmaAssessor.md
## Persona D1 — The Quality Engineer | Primary Arm 1: Six Sigma Assessor

**Arm ID:** `ARM-01`
**Name:** `six_sigma_assessor`
**Type:** Primary
**Status:** Production-ready
**Version:** 1.0.0
**Owner:** D1 The Quality Engineer
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`

---

## 1. Identity

```yaml
arm_identity:
  id: ARM-01
  name: "Six Sigma Assessor"
  persona_id: D1
  version: "1.0.0"
  purpose: >
    Execute complete DMAIC (Define, Measure, Analyze, Improve, Control) assessments of processes,
    products, pipelines, and systems. Compute sigma levels from DPMO, process capability indices
    (Cp, Cpk, Pp, Ppk), and generate control plans with improvement roadmaps. Validates all
    statistical computations against NumPy reference fixtures. Every assessment is evidence-backed —
    phase transitions without evidence are blocked.
```

---

## 2. Sensors

| Sensor ID | Name | Trigger | Filter | Arm Action | Handler Module |
|-----------|------|---------|--------|------------|---------------|
| `SENS-D1-01A` | `dmaic_request_sensor` | On-demand quality assessment request | `assessment_type` in [process, product, pipeline, system] | Ingest data, run DMAIC, emit scorecard | `sensors.dmaic_request_handler` |
| `SENS-D1-01B` | `quarterly_review_sensor` | Cron: `0 0 1 */3 *` UTC | Active projects with `dmaic_status=ongoing` | Update DMAIC with new data, check phase gates | `sensors.quarterly_review_handler` |
| `SENS-D1-01C` | `quality_gate_failure_sensor` | CI/CD quality gate failure event | `gate_status=fail` AND `severity>=high` | Trigger emergency DMAIC, skip to Analyze phase | `sensors.gate_failure_handler` |
| `SENS-D1-01D` | `cross_persona_chained_sensor` | Hook: `d9_to_d1_quality_v1` | D9 reports quality gap (coverage < threshold or complexity breach) | Run DMAIC on affected module, prioritize improvement | `sensors.hook_handler` |
| `SENS-D1-01E` | `process_capability_trigger` | Manufacturing or process data ingestion | `data_type=process_metrics` AND `sample_size>=30` | Compute Cp/Cpk/Pp/Ppk, generate control plan | `sensors.process_capability_handler` |
| `SENS-D1-01F` | `sigma_level_tracker` | Scheduled DPMO computation | `defect_data` updated in last 24h | Recompute sigma level, trend analysis | `sensors.sigma_tracker_handler` |

### Sensor Event Schemas

```yaml
dmaic_request_event:
  assessment_id: "UUID"
  target_id: "string (repo, process, or system ID)"
  target_type: "enum [repository, process, pipeline, module, system]"
  assessment_scope: "enum [full_dmaic, measure_only, capability_study, control_plan]"
  data_uris:
    - defect_logs: "string (S3/MinIO path)"
    - test_results: "string (S3/MinIO path)"
    - coverage_reports: "string (S3/MinIO path)"
    - process_metrics: "string (S3/MinIO path, optional)"
  standards:
    target_coverage: "float (default: 0.80)"
    target_defect_density: "float (default: 0.2 defects/KLOC)"
    target_sigma_level: "float (default: 4.0)"
    target_cpk: "float (default: 1.33)"
  historical_baseline_id: "UUID (optional)"
  priority: "enum [p0, p1, p2, p3]"
  callback_url: "string (optional)"
```

---

## 3. Tools

| Tool ID | Name | Description | Input Schema | Output Schema | Timeout | Retry | Permissions | Error Handling |
|---------|------|-------------|-------------|-------------|---------|-------|-------------|---------------|
| `TOOL-D1-01` | `SixSigmaAssessor` | Core DMAIC engine — phase management, evidence gate validation, improvement plan generation | `d1/tools/six_sigma_input.json` | `d1/tools/six_sigma_output.json` | 1800s | 3× exponential | `d1-quality:assess` | Fallback: partial DMAIC with blocked phases flagged |
| `TOOL-D1-02` | `SPC_Controller` | Control chart generation and Western Electric rule evaluation (shared with ARM-02) | `d1/tools/spc_input.json` | `d1/tools/spc_output.json` | 300s | 3× exponential | `d1-quality:control` | Fallback: last known control limits with warn status |
| `TOOL-D1-03` | `ControlChartGenerator` | Generate X-bar, R, S, X, MR, p, np, c, u charts with GAI-OBSERVE theme | `d1/tools/chart_gen_input.json` | `d1/tools/chart_gen_output.json` | 120s | 2× exponential | `d1-quality:visualize` | Fallback: ASCII chart with data table |
| `TOOL-D1-04` | `WesternElectricAnalyzer` | Apply all 8 Western Electric rules to control chart data | `d1/tools/we_input.json` | `d1/tools/we_output.json` | 60s | 2× fixed | `d1-quality:analyze` | Fallback: manual rule check checklist |
| `TOOL-D1-05` | `ProcessCapabilityCalculator` | Compute Cp, Cpk, Pp, Ppk with confidence intervals and normality tests | `d1/tools/capability_input.json` | `d1/tools/capability_output.json` | 120s | 3× exponential | `d1-quality:calculate` | Fallback: point estimates without CI |
| `TOOL-D1-06` | `TestGapAnalyzer` | Coverage gap analysis and prioritized test planning (shared with ARM-03) | `d1/tools/gap_input.json` | `d1/tools/gap_output.json` | 600s | 3× exponential | `d1-quality:analyze` | Fallback: high-level gap summary |
| `TOOL-D1-07` | `CoverageOptimizer` | Optimize test suite for maximum coverage with minimum execution cost | `d1/tools/coverage_opt_input.json` | `d1/tools/coverage_opt_output.json` | 300s | 2× exponential | `d1-quality:optimize` | Fallback: greedy coverage algorithm |
| `TOOL-D1-08` | `FlakyTestDetector` | Statistical detection of flaky tests from run history | `d1/tools/flaky_input.json` | `d1/tools/flaky_output.json` | 180s | 2× exponential | `d1-quality:detect` | Fallback: manual inspection checklist |
| `TOOL-D1-09` | `DefectAnalyzer` | Defect density, trend, root cause classification, and Pareto analysis | `d1/tools/defect_input.json` | `d1/tools/defect_output.json` | 120s | 2× exponential | `d1-quality:analyze` | Fallback: raw defect count |
| `TOOL-D1-10` | `QualityScorecard` | Multi-dimensional quality scorecard generator | `d1/tools/scorecard_input.json` | `d1/tools/scorecard_output.json` | 60s | 2× fixed | `d1-quality:report` | Fallback: text-only scorecard |
| `TOOL-D1-11` | `DPMOCalculator` | Defects Per Million Opportunities calculator with sigma level translation | `d1/tools/dpmo_input.json` | `d1/tools/dpmo_output.json` | 30s | 2× fixed | `d1-quality:calculate` | Fallback: manual DPMO formula reference |
| `TOOL-D1-12` | `CodeReviewAuditor` | Code review practice assessment — velocity, thoroughness, distribution | `d1/tools/review_audit_input.json` | `d1/tools/review_audit_output.json` | 120s | 2× exponential | `d1-quality:audit` | Fallback: basic review count |
| `TOOL-D1-13` | `TestReliabilityAnalyzer` | Test reliability index, confidence intervals, and trend analysis | `d1/tools/reliability_input.json` | `d1/tools/reliability_output.json` | 180s | 2× exponential | `d1-quality:analyze` | Fallback: pass rate only |
| `TOOL-D1-14` | `SigmaLevelTracker` | Historical sigma level tracking with trend projection | `d1/tools/sigma_track_input.json` | `d1/tools/sigma_track_output.json` | 60s | 2× fixed | `d1-quality:track` | Fallback: current sigma only |
| `TOOL-D1-15` | `ImprovementPlanGenerator` | Prioritized improvement plan with effort estimates and expected impact | `d1/tools/improvement_input.json` | `d1/tools/improvement_output.json` | 300s | 3× exponential | `d1-quality:plan` | Fallback: generic improvement checklist |

---

## 4. Skills

| Skill | Name | Description | Steps | Validation | Error Handling |
|-------|------|-------------|-------|------------|---------------|
| `SKILL-D1-01` | `SixSigmaAssessment` | Execute full DMAIC assessment with evidence gates and phase transitions | 1. Define scope, boundaries, CTQs. 2. Measure current state (DPMO, coverage, defect density). 3. Analyze root causes with statistical tools. 4. Generate improvement plan with prioritized actions. 5. Establish control plan with monitoring. | Phase gates require evidence; sigma computation matches NumPy fixtures; Cpk > 1.0 for "capable"; all claims have statistical backing | Block phase transition without evidence; retry analysis with different tools; escalate to D3 if improvement requires scheduling |
| `SKILL-D1-02` | `StatisticalProcessControl` | Apply SPC methodology with control charts and Western Electric rules | 1. Select appropriate control chart type. 2. Compute center line and control limits. 3. Plot data points. 4. Apply all 8 Western Electric rules. 5. Classify process stability. 6. Generate control plan. | Control limits within ±3σ; all 8 rules implemented; chart type matches data type; R-GATE-2 deterministic math | Fallback to simpler chart type; alert D5 for data quality issues; escalate if process fundamentally unstable |
| `SKILL-D1-03` | `TestGapAnalysis` | Comprehensive coverage gap analysis with prioritized test plan | 1. Ingest coverage.xml and test data. 2. Map coverage to source modules. 3. Identify untested branches, paths, and edge cases. 4. Compute risk scores per module. 5. Generate prioritized test plan with effort estimates. 6. Validate against target coverage. | Gap map matches coverage.xml exactly; effort estimates within ±20% of historical; risk scores based on complexity + security + data integrity | Retry with different coverage tool; manual review flag for complex modules; escalate to D7 for implementation |
| `SKILL-D1-04` | `CoverageOptimization` | Optimize test suite for maximum coverage with minimum execution cost | 1. Analyze test execution times and coverage contributions. 2. Identify redundant tests. 3. Reorder tests for fast failure. 4. Select minimal subset for target coverage. 5. Validate optimized suite preserves fault detection. | Optimized suite achieves target coverage; execution time reduced ≥20%; no fault detection regression; mutation score maintained | Fallback to original suite; flag redundant tests; escalate if coverage unattainable with available tests |
| `SKILL-D1-05` | `FlakyTestDetection` | Statistical detection of flaky tests from run history with root cause classification | 1. Collect test run history (≥10 runs). 2. Compute pass rate variance and confidence intervals. 3. Identify tests with non-deterministic outcomes. 4. Classify root cause (race, timeout, dependency, data). 5. Generate stabilization recommendations. | Flaky verdict backed by ≥10 runs; variance threshold matches R-ARM-REPRO-2; root cause classification accuracy >85% | Require more runs if <10 available; flag manual inspection if classification uncertain; escalate to D9 for race condition fixes |
| `SKILL-D1-06` | `DefectAnalysis` | Defect density, trend, root cause classification, and Pareto analysis | 1. Ingest defect logs from issue tracker. 2. Classify defects by type, severity, module. 3. Compute defect density (defects/KLOC). 4. Run Pareto analysis (80/20 rule). 5. Identify systemic vs. sporadic defects. 6. Trend analysis with forecast. | Defect density matches KLOC denominator; Pareto chart validated; trends statistically significant (p < 0.05); DPMO computation correct | Fallback to manual defect count; flag data quality issues; escalate if defect classification ambiguous |
| `SKILL-D1-07` | `QualityAuditing` | Multi-dimensional quality audit with scorecard generation | 1. Collect quality metrics across all dimensions. 2. Normalize metrics to 0-100 scale. 3. Apply weighting based on target audience. 4. Compute overall score with grade. 5. Identify priority gaps. 6. Generate actionable recommendations. | Score computation transparent and reproducible; weights configurable; all sub-metrics within valid ranges; grades match standard scale (A≥90, B≥80, etc.) | Fallback to unweighted average; flag missing metrics; escalate to G1 if certification grade contested |
| `SKILL-D1-08` | `ProcessCapabilityStudy` | Statistical process capability study (Cp, Cpk, Pp, Ppk) with normality tests | 1. Collect process data (n≥30). 2. Test normality (Shapiro-Wilk, Anderson-Darling). 3. Compute Cp, Cpk, Pp, Ppk. 4. Generate confidence intervals (bootstrap). 5. Compare to specification limits. 6. Classify capability level. | Sample size ≥30 for reliable Cpk; normality tests correct; Cpk computation matches AIAG standards; bootstrap CI valid | Require more samples if n<30; transform data if non-normal; fallback to Pp/Ppk only if Cpk unreliable |
| `SKILL-D1-09` | `CodeReviewAudit` | Assessment of code review practices and their quality impact | 1. Ingest PR data from Git platform. 2. Compute review velocity (median time to first review, time to approve). 3. Compute review thoroughness (comments per PR, review depth). 4. Analyze reviewer distribution (load balancing, expertise matching). 5. Correlate review metrics with defect escape rate. | Data matches Git API response; velocity metrics use median not mean; correlation statistically significant; reviewer anonymity preserved | Fallback to basic counts; flag API access issues; escalate to D9 for review bot improvements |
| `SKILL-D1-10` | `QualityImprovementPlanning` | Generate prioritized improvement plans with effort estimates and ROI | 1. Consolidate findings from all quality analyses. 2. Score improvements by effort, impact, and risk. 3. Generate Pareto-optimal improvement set. 4. Create phased roadmap with milestones. 5. Estimate effort using historical data. 6. Define success metrics and control plan. | Effort estimates within ±30% of historical; improvement plan covers all fail/marginal metrics; milestones achievable; ROI positive for all recommended items | Fallback to generic recommendations; flag missing historical data; escalate to D3 for resource planning |

---

## 5. Plugins

| Plugin | Name | Type | Config | Version | Dependencies |
|--------|------|------|--------|---------|-------------|
| `PLUGIN-D1-01` | `DMAIC_Framework` | methodology_engine | DMAIC phase manager, evidence gate validator, control plan template engine | 1.0.0 | `pydantic>=2.0`, `numpy>=1.26.0`, `scipy>=1.12.0` |
| `PLUGIN-D1-02` | `DevAgent` | quality_integration | DevAgent quality arm integration for code review and defect tracking | 1.0.0 | `gitpython>=3.1.0`, `pygithub>=2.0.0`, `jira>=3.5.0` |
| `PLUGIN-D1-03` | `ReproSense` | reproducibility_engine | ReproSense coverage and flaky test integration | 1.0.0 | `coverage>=7.3.0`, `pytest>=7.4.0`, `pytest-rerunfailures>=12.0` |
| `PLUGIN-D1-04` | `SPC_Tool` | statistical_engine | SPC computation engine with all 8 Western Electric rules | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0`, `statsmodels>=0.14.0` |
| `PLUGIN-D1-05` | `Control_Chart` | visualization | Control chart generator with Western Electric rule annotations | 1.0.0 | `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `theme-factory>=1.0.0` |
| `PLUGIN-D1-06` | `Western_Electric` | rule_engine | Western Electric rule engine with configurable thresholds | 1.0.0 | `numpy>=1.26.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-07` | `Coverage_Analyzer` | coverage_engine | Coverage XML parser, gap mapper, and optimizer | 1.0.0 | `coverage>=7.3.0`, `lxml>=4.9.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-08` | `Flaky_Test_Detector` | flaky_engine | Flaky test statistical detection and classification | 1.0.0 | `scipy>=1.12.0`, `numpy>=1.26.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-09` | `Defect_Tracker` | defect_integration | Issue tracker integration for defect ingestion and analysis | 1.0.0 | `jira>=3.5.0`, `github>=2.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-10` | `Quality_Scorecard` | reporting | Multi-dimensional quality scorecard generator | 1.0.0 | `jinja2>=3.1.0`, `reportlab>=4.0.0`, `openpyxl>=3.1.0` |
| `PLUGIN-D1-11` | `DPMO_Calculator` | dpmo_engine | DPMO and sigma level translation engine | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0` |
| `PLUGIN-D1-12` | `Review_Auditor` | review_integration | Git platform PR data ingestion and review metrics | 1.0.0 | `pygithub>=2.0.0`, `gitlab-python>=4.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-13` | `Test_Optimizer` | optimization | Test suite optimizer using genetic algorithm and greedy selection | 1.0.0 | `deap>=1.4.0`, `numpy>=1.26.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-14` | `Sigma_Tracker` | tracking_engine | Historical sigma level tracking with trend projection | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0`, `statsmodels>=0.14.0` |
| `PLUGIN-D1-15` | `Improvement_Planner` | planning_engine | Improvement plan generator with effort estimation and ROI | 1.0.0 | `numpy>=1.26.0`, `pandas>=2.2.0`, `jinja2>=3.1.0` |

---

## 6. Memory

### 6.1 STM — Active Assessment Session Cache

```json
{
  "turn_id": "turn-arm01-20260628-001",
  "timestamp": "2026-06-28T02:00:00Z",
  "persona_id": "D1",
  "arm_id": "ARM-01",
  "assessment_id": "dmaic-pipeline-001",
  "target_id": "ci-cd-pipeline-alpha",
  "target_type": "pipeline",
  "dmaic_phase": "analyze",
  "phase_evidence": {
    "define": "completed",
    "measure": "completed",
    "analyze": "in_progress",
    "improve": "blocked",
    "control": "blocked"
  },
  "current_sigma_level": 2.3,
  "target_sigma_level": 4.0,
  "cpk": 0.85,
  "dpmo": 23000,
  "modules_analyzed": ["build", "test", "deploy", "verify"],
  "tags": ["dmaic", "pipeline", "six-sigma"],
  "embedding": [0.12, -0.45, 0.89, ...],
  "ttl_active_seconds": 86400,
  "ttl_recent_seconds": 604800
}
```

### 6.2 LTM — Six Sigma Baseline Database

```json
{
  "fact_id": "fact-20260628-001",
  "category": "six_sigma_baseline",
  "key": "pipeline-alpha_dmaic_baseline_2026q2",
  "value": {
    "target_id": "ci-cd-pipeline-alpha",
    "sigma_level": 2.3,
    "dpmo": 23000,
    "cp": 1.1,
    "cpk": 0.85,
    "pp": 1.05,
    "ppk": 0.82,
    "defect_density": 0.18,
    "coverage": 0.71,
    "test_reliability": 0.83,
    "control_plan_id": "cp-pipeline-alpha-001",
    "improvement_roadmap_id": "roadmap-pipeline-alpha-001"
  },
  "source": "ARM-01-SixSigmaAssessor",
  "timestamp": "2026-06-28T02:15:00Z",
  "confidence": 0.97,
  "expiry": "2027-06-28T00:00:00Z",
  "assessment_id": "dmaic-pipeline-001",
  "ledger_hash": "sha256:abc123..."
}
```

### 6.3 EM — DMAIC Session Record

```json
{
  "session_id": "session-20260628-001",
  "assessment_id": "dmaic-pipeline-001",
  "persona_id": "D1",
  "arm_id": "ARM-01",
  "start_time": "2026-06-28T02:00:00Z",
  "end_time": "2026-06-28T02:45:00Z",
  "target_id": "ci-cd-pipeline-alpha",
  "dmaic_phases": {
    "define": { "status": "complete", "duration_min": 5, "evidence_count": 3 },
    "measure": { "status": "complete", "duration_min": 12, "evidence_count": 8 },
    "analyze": { "status": "complete", "duration_min": 18, "evidence_count": 6 },
    "improve": { "status": "complete", "duration_min": 8, "evidence_count": 4 },
    "control": { "status": "complete", "duration_min": 2, "evidence_count": 2 }
  },
  "sigma_level_before": 2.3,
  "sigma_level_after": 4.1,
  "dpmo_before": 23000,
  "dpmo_after": 3400,
  "cpk_before": 0.85,
  "cpk_after": 1.45,
  "improvement_actions": 7,
  "control_plan_items": 5,
  "embedding": [0.12, -0.45, 0.89, ...],
  "compression_ratio": 0.10,
  "ledger_hash": "sha256:abc123..."
}
```

---

## 7. Actuators

| Output | Format | Destination | Generation Trigger |
|--------|--------|-------------|-------------------|
| **DMAIC Report** | PDF + JSON + Markdown | Quality portal / S3 / Git repo | Every assessment completion |
| **Sigma Level Scorecard** | Interactive dashboard + PDF + JSON | Executive dashboard / Quality portal | Every sigma computation |
| **Process Capability Study** | PDF + Control charts + JSON | Quality portal / Manufacturing system | Every capability study |
| **Control Plan** | Markdown + JSON + Excel | Quality portal / CI/CD config | Every DMAIC Control phase |
| **Improvement Roadmap** | Gantt-style Markdown + JSON | Project management / D3 integration | Every DMAIC Improve phase |
| **Phase Gate Evidence** | JSON + Attachments | LTM / P2 Ledger | Every phase transition |
| **Alert Dispatch** | Slack / Email / PagerDuty | D5 SRE / Team channels | Sigma level drop or Cpk < 1.0 |
| **Ledger Entry** | Hash-chained record | P2 PostgreSQL | Every session completion |

### Actuator Output Schemas

```yaml
dmaic_report_output:
  report_id: "UUID"
  assessment_id: "UUID"
  target_id: "string"
  phases:
    define: { status: "enum", evidence: "array", summary: "string" }
    measure: { status: "enum", metrics: "object", summary: "string" }
    analyze: { status: "enum", root_causes: "array", summary: "string" }
    improve: { status: "enum", actions: "array", summary: "string" }
    control: { status: "enum", control_plan: "object", summary: "string" }
  sigma_level: "float"
  dpmo: "int"
  cpk: "float"
  overall_grade: "enum [A, B, C, D, F]"
  deliverable_uris: "array[string]"
  ledger_hash: "string"
```

---

## 8. Circuit Breaker

```yaml
circuit_breaker:
  name: "arm_01_cb"
  thresholds:
    consecutive_failures: 5
    failure_rate: 0.25
    slow_call_rate: 0.50
    slow_call_duration_ms: 120000
  states:
    - CLOSED: normal operation
    - OPEN: reject all invocations, return 503
    - HALF_OPEN: allow 1 test invocation after recovery_timeout
  recovery_timeout_ms: 30000
  half_open_max_calls: 1
  health_check_interval: 5000
  fallback:
    action: "return_cached_assessment_with_staleness_warning"
    stub_template: "d1/fallback_dmaic_summary.md"
    alert: "D5 SRE Commander + P1 The Router"
```

---

## 9. Error Handler

```yaml
error_handler:
  strategies:
    retry:
      enabled: true
      policy: "exponential_backoff"
      max_attempts: 3
      base_delay_ms: 1000
      max_delay_ms: 30000
      conditions: ["timeout", "transient_error", "rate_limit"]
    fallback:
      enabled: true
      action: "degrade_to_cached_baseline"
      stub_quality: "summary_with_last_known_metrics"
      conditions: ["persistent_failure", "circuit_open"]
    escalate:
      enabled: true
      target: "human_quality_engineer"
      conditions: ["3_failed_retries", "sigma_level_critical", "cpk_below_minimum"]
      notify: ["D5", "G1", "P2"]
    circuit_break:
      enabled: true
      action: "stop_and_queue"
      queue: "d1_arm01_dlq"
      conditions: ["failure_rate > 25%", "consecutive_failures >= 5"]
    compensate:
      enabled: true
      action: "rollback_phase_transition"
      cleanup: ["revert_stm", "notify_stakeholders", "release_resources"]
      conditions: ["assessment_aborted", "evidence_rejected"]
    log:
      enabled: true
      level: "error"
      destination: ["P2_ledger", "D5_monitoring", "EM_timeseries"]
      format: "structured_json"
      include_stack_trace: true
      include_context: true
```

---

## 10. Persona Delegation

| Condition | Delegation Target | Hook Contract | Payload | Escalation Condition | Handoff Protocol | Return Policy |
|-----------|-------------------|---------------|---------|---------------------|-----------------|--------------|
| Quality issues require code fixes | D9 Forward Engineer | `d1_to_d9_quality_v1` | {quality_issues, module_ids, fix_recommendations, priority} | CVSS >= 7.0 or complexity > 15 | Async webhook + priority queue | D9 returns fixed code; D1 re-assesses sigma level |
| Test gaps require implementation | D7 Test Automator | `d1_to_d7_test_v1` | {gap_map, prioritized_tests, coverage_target, risk_levels} | Coverage gap > 20% or security-critical module | API POST + callback URL | D7 returns test suite; D1 validates coverage improvement |
| Quality certification required | G1 The Arbiter | `d1_to_g1_certification_v1` | {quality_scorecard, sigma_level, cpk, dpmo, compliance_mapping} | Sigma < 3.0 or Cpk < 1.0 | API POST + 24h SLA | G1 returns certification verdict; D1 applies conditions if conditional |
| Improvement initiatives need scheduling | D3 Delivery Captain | `d1_to_d3_planning_v1` | {improvement_roadmap, effort_estimates, priority, dependencies} | Effort > 40 person-days or cross-team | API POST + project planning sync | D3 returns scheduled plan; D1 monitors control plan |
| Continuous SPC monitoring deployment | G6 The Sentinel | `d1_to_g6_monitoring_v1` | {control_plan, metrics, thresholds, alert_rules} | Process stability critical | Async webhook + monitoring config | G6 deploys monitoring; D1 receives alert on rule violations |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team
**Classification:** Internal — Arm Specification
**Next Review:** 2026-07-28
