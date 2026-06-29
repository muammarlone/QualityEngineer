# ARM_02_D1_SPC_Controller.md
## Persona D1 — The Quality Engineer | Primary Arm 2: SPC Controller

**Arm ID:** `ARM-02`  
**Name:** `spc_controller`  
**Type:** Primary  
**Status:** Production-ready  
**Version:** 1.0.0  
**Owner:** D1 The Quality Engineer  
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`

---

## 1. Identity

```yaml
arm_identity:
  id: ARM-02
  name: "SPC Controller"
  persona_id: D1
  version: "1.0.0"
  purpose: >
    Real-time Statistical Process Control (SPC) monitoring with control chart generation,
    Western Electric rule violation detection, and process stability classification. Supports
    all standard control chart types (X-bar, R, S, X, MR, p, np, c, u) and all 8 Western
    Electric rules. Every computation is deterministic and validated against NumPy reference
    fixtures. Process stability verdicts trigger automated quality alerts and cross-persona
    chains.
```

---

## 2. Sensors

| Sensor ID | Name | Trigger | Filter | Arm Action | Handler Module |
|-----------|------|---------|--------|------------|---------------|
| `SENS-D1-02A` | `metric_stream_sensor` | Real-time metric stream ingestion | `metric_type` in [build_time, test_duration, defect_count, coverage, error_rate] | Append to control chart, evaluate Western Electric rules | `sensors.metric_stream_handler` |
| `SENS-D1-02B` | `scheduled_spc_scan` | Cron: `*/15 * * * *` UTC | Active control charts with `monitoring_status=active` | Recompute control limits, evaluate all rules | `sensors.scheduled_spc_handler` |
| `SENS-D1-02C` | `stability_check_sensor` | On-demand stability check request | `chart_id` exists and `data_points>=2` | Generate control chart, classify stability, return verdict | `sensors.stability_check_handler` |
| `SENS-D1-02D` | `threshold_breach_sensor` | Event: metric exceeds threshold | `metric_value` outside `specification_limits` | Immediate rule evaluation, alert dispatch, D5 notification | `sensors.threshold_breach_handler` |
| `SENS-D1-02E` | `cross_persona_chained_sensor` | Hook: `d5_to_d1_spc_v1` | D5 reports production metric anomaly | SPC analysis of production metric, classify process stability | `sensors.hook_handler` |
| `SENS-D1-02F` | `control_limit_update` | Event: process baseline recalculated | `new_baseline` differs from `current_baseline` by >5% | Recompute control limits, update all active charts | `sensors.baseline_update_handler` |

### Sensor Event Schemas

```yaml
metric_stream_event:
  metric_id: "UUID"
  timestamp: "ISO-8601"
  target_id: "string (process, pipeline, or module ID)"
  metric_name: "enum [build_time, test_duration, defect_count, coverage, error_rate, latency, throughput]"
  metric_value: "float"
  unit: "string"
  sample_size: "int (default: 1)"
  subgroup_id: "string (optional, for X-bar/R/S charts)"

stability_check_event:
  chart_id: "UUID"
  target_id: "string"
  chart_type: "enum [xbar, r, s, x, mr, p, np, c, u]"
  data_window: "enum [24h, 7d, 30d, custom]"
  custom_start: "ISO-8601 (optional)"
  custom_end: "ISO-8601 (optional)"
  include_rules: "array[enum] (default: all 8)"
  callback_url: "string (optional)"
```

---

## 3. Tools

| Tool ID | Name | Description | Input Schema | Output Schema | Timeout | Retry | Permissions | Error Handling |
|---------|------|-------------|-------------|-------------|---------|-------|-------------|---------------|
| `TOOL-D1-02` | `SPC_Controller` | Core SPC engine — control limit computation, rule evaluation, stability classification | `d1/tools/spc_input.json` | `d1/tools/spc_output.json` | 300s | 3× exponential | `d1-quality:control` | Fallback: last known control limits with warn status |
| `TOOL-D1-03` | `ControlChartGenerator` | Generate control chart images with Western Electric rule annotations | `d1/tools/chart_gen_input.json` | `d1/tools/chart_gen_output.json` | 120s | 2× exponential | `d1-quality:visualize` | Fallback: ASCII chart with data table |
| `TOOL-D1-04` | `WesternElectricAnalyzer` | Apply all 8 Western Electric rules to control chart data | `d1/tools/we_input.json` | `d1/tools/we_output.json` | 60s | 2× fixed | `d1-quality:analyze` | Fallback: manual rule check checklist |
| `TOOL-D1-05` | `ProcessCapabilityCalculator` | Compute Cp, Cpk, Pp, Ppk with normality tests and confidence intervals | `d1/tools/capability_input.json` | `d1/tools/capability_output.json` | 120s | 3× exponential | `d1-quality:calculate` | Fallback: point estimates without CI |
| `TOOL-D1-16` | `AnomalyDetector` | Statistical anomaly detection using control chart deviations and run patterns | `d1/tools/anomaly_input.json` | `d1/tools/anomaly_output.json` | 180s | 2× exponential | `d1-quality:detect` | Fallback: threshold-based anomaly flag |
| `TOOL-D1-17` | `ControlLimitUpdater` | Dynamic control limit recomputation with rolling windows and seasonal adjustment | `d1/tools/cl_update_input.json` | `d1/tools/cl_update_output.json` | 120s | 2× exponential | `d1-quality:control` | Fallback: fixed limits with drift warning |
| `TOOL-D1-18` | `ProcessStabilityClassifier` | Classify process stability (stable, unstable, trending, cyclical) from rule violations | `d1/tools/stability_input.json` | `d1/tools/stability_output.json` | 60s | 2× fixed | `d1-quality:classify` | Fallback: manual classification prompt |
| `TOOL-D1-19` | `SubgroupAnalyzer` | Subgroup analysis for X-bar, R, S charts — homogeneity tests, stratification | `d1/tools/subgroup_input.json` | `d1/tools/subgroup_output.json` | 120s | 2× exponential | `d1-quality:analyze` | Fallback: pooled standard deviation |
| `TOOL-D1-20` | `NormalityTester` | Shapiro-Wilk, Anderson-Darling, and Kolmogorov-Smirnov normality tests | `d1/tools/normality_input.json` | `d1/tools/normality_output.json` | 60s | 2× fixed | `d1-quality:calculate` | Fallback: visual inspection flag |
| `TOOL-D1-21` | `TrendAnalyzer` | Trend detection using CUSUM, EWMA, and run tests for process drift | `d1/tools/trend_input.json` | `d1/tools/trend_output.json` | 120s | 2× exponential | `d1-quality:analyze` | Fallback: simple linear regression |
| `TOOL-D1-22` | `SpecificationLimitValidator` | Validate specification limits against process capability and customer requirements | `d1/tools/spec_limit_input.json` | `d1/tools/spec_limit_output.json` | 60s | 2× fixed | `d1-quality:validate` | Fallback: warning about unverified limits |
| `TOOL-D1-23` | `RuleViolationReporter` | Generate formatted rule violation reports with root cause suggestions | `d1/tools/violation_report_input.json` | `d1/tools/violation_report_output.json` | 60s | 2× fixed | `d1-quality:report` | Fallback: plain text violation list |
| `TOOL-D1-24` | `ChartTypeRecommender` | Recommend optimal control chart type based on data characteristics | `d1/tools/chart_recommend_input.json` | `d1/tools/chart_recommend_output.json` | 30s | 2× fixed | `d1-quality:recommend` | Fallback: X chart for continuous, p chart for proportion |
| `TOOL-D1-25` | `SPCDashboardUpdater` | Update real-time SPC dashboard with new data points and rule status | `d1/tools/dashboard_input.json` | `d1/tools/dashboard_output.json` | 30s | 2× fixed | `d1-quality:dashboard` | Fallback: batch update on next cycle |
| `TOOL-D1-26` | `SeasonalAdjuster` | Apply seasonal decomposition and adjustment to control chart data | `d1/tools/seasonal_input.json` | `d1/tools/seasonal_output.json` | 120s | 2× exponential | `d1-quality:adjust` | Fallback: no seasonal adjustment |

---

## 4. Skills

| Skill | Name | Description | Steps | Validation | Error Handling |
|-------|------|-------------|-------|------------|---------------|
| `SKILL-D1-02` | `StatisticalProcessControl` | Complete SPC workflow — chart selection, limit computation, rule evaluation, stability classification | 1. Select control chart type based on data characteristics. 2. Compute center line and control limits (±3σ). 3. Plot data points and identify patterns. 4. Apply all 8 Western Electric rules. 5. Classify process stability (stable, unstable, trending, cyclical). 6. Generate control plan with monitoring schedule. | Chart type matches data; limits within ±3σ; all 8 rules implemented; rule violations correctly identified; stability classification matches expert assessment >90% | Retry with simpler chart; alert D5 for data quality issues; fallback to manual SPC review |
| `SKILL-D1-04` | `CoverageOptimization` | Optimize test suite for maximum coverage with minimum execution cost | 1. Analyze test execution times and coverage contributions. 2. Identify redundant tests. 3. Reorder tests for fast failure. 4. Select minimal subset for target coverage. 5. Validate optimized suite preserves fault detection. | Optimized suite achieves target coverage; execution time reduced ≥20%; no fault detection regression; mutation score maintained | Fallback to original suite; flag redundant tests; escalate if coverage unattainable |
| `SKILL-D1-05` | `FlakyTestDetection` | Statistical detection of flaky tests from run history with root cause classification | 1. Collect test run history (≥10 runs). 2. Compute pass rate variance and confidence intervals. 3. Identify tests with non-deterministic outcomes. 4. Classify root cause (race, timeout, dependency, data). 5. Generate stabilization recommendations. | Flaky verdict backed by ≥10 runs; variance threshold matches R-ARM-REPRO-2; root cause classification accuracy >85% | Require more runs if <10 available; flag manual inspection if classification uncertain |
| `SKILL-D1-08` | `ProcessCapabilityStudy` | Statistical process capability study (Cp, Cpk, Pp, Ppk) with normality tests | 1. Collect process data (n≥30). 2. Test normality (Shapiro-Wilk, Anderson-Darling). 3. Compute Cp, Cpk, Pp, Ppk. 4. Generate confidence intervals (bootstrap). 5. Compare to specification limits. 6. Classify capability level. | Sample size ≥30 for reliable Cpk; normality tests correct; Cpk computation matches AIAG standards; bootstrap CI valid | Require more samples if n<30; transform data if non-normal; fallback to Pp/Ppk only |
| `SKILL-D1-11` | `QualityAuditing` | Multi-dimensional quality audit with scorecard generation | 1. Collect quality metrics across all dimensions. 2. Normalize metrics to 0-100 scale. 3. Apply weighting based on target audience. 4. Compute overall score with grade. 5. Identify priority gaps. 6. Generate actionable recommendations. | Score computation transparent and reproducible; weights configurable; all sub-metrics within valid ranges; grades match standard scale | Fallback to unweighted average; flag missing metrics; escalate to G1 if certification grade contested |

---

## 5. Plugins

| Plugin | Name | Type | Config | Version | Dependencies |
|--------|------|------|--------|---------|-------------|
| `PLUGIN-D1-04` | `SPC_Tool` | statistical_engine | Core SPC computation engine with control limits and rule evaluation | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0`, `statsmodels>=0.14.0` |
| `PLUGIN-D1-05` | `Control_Chart` | visualization | Control chart generator with Western Electric rule annotations and GAI-OBSERVE theme | 1.0.0 | `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `theme-factory>=1.0.0` |
| `PLUGIN-D1-06` | `Western_Electric` | rule_engine | Western Electric rule engine with configurable thresholds and custom rule support | 1.0.0 | `numpy>=1.26.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-07` | `Coverage_Analyzer` | coverage_engine | Coverage XML parser, gap mapper, and test suite optimizer | 1.0.0 | `coverage>=7.3.0`, `lxml>=4.9.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-08` | `Flaky_Test_Detector` | flaky_engine | Flaky test statistical detection and root cause classification | 1.0.0 | `scipy>=1.12.0`, `numpy>=1.26.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-09` | `Defect_Tracker` | defect_integration | Issue tracker integration for defect ingestion and trend analysis | 1.0.0 | `jira>=3.5.0`, `github>=2.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-11` | `DPMO_Calculator` | dpmo_engine | DPMO and sigma level translation engine with confidence intervals | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0` |
| `PLUGIN-D1-16` | `Anomaly_Detector` | anomaly_engine | Statistical anomaly detection for SPC deviations and run patterns | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0`, `scikit-learn>=1.5.0` |
| `PLUGIN-D1-17` | `TimescaleDB_Metrics` | timeseries_store | Time-series metric storage for SPC data with hypertable support | 1.0.0 | `timescale-client>=2.14.0`, `psycopg2>=2.9.0` |
| `PLUGIN-D1-18` | `Grafana_SPC` | dashboard | Pre-built Grafana dashboards for SPC monitoring with rule violation alerts | 1.0.0 | `grafana-api>=1.0.0`, `docker>=24.0.0` |
| `PLUGIN-D1-19` | `Prometheus_Metrics` | metrics_exporter | Prometheus metrics exporter for SPC control limits, rule violations, and stability scores | 1.0.0 | `prometheus-client>=0.20.0` |
| `PLUGIN-D1-20` | `Redis_Stream` | stream_cache | Redis stream for real-time metric ingestion and rule evaluation buffer | 1.0.0 | `redis>=5.0.0`, `redis-py>=5.0.0` |
| `PLUGIN-D1-21` | `NumPy_Fixture` | validation_engine | NumPy reference fixture validator for deterministic SPC math (R-GATE-2) | 1.0.0 | `numpy>=1.26.0`, `pytest>=7.4.0` |
| `PLUGIN-D1-22` | `PostgreSQL_15` | database | PostgreSQL 15 with pgvector for SPC metadata and JSONB rule configurations | 1.0.0 | `psycopg2>=2.9.0`, `pgvector>=0.2.0` |
| `PLUGIN-D1-23` | `FastAPI_SPC` | api_server | FastAPI >=0.104.0 router for SPC endpoints with Pydantic v2 schemas | 1.0.0 | `fastapi>=0.104.0`, `pydantic>=2.0`, `uvicorn>=0.23.0` |

---

## 6. Memory

### 6.1 STM — Active SPC Session Cache

```json
{
  "turn_id": "turn-arm02-20260628-001",
  "timestamp": "2026-06-28T14:00:00Z",
  "persona_id": "D1",
  "arm_id": "ARM-02",
  "chart_id": "chart-build-time-alpha",
  "target_id": "ci-cd-pipeline-alpha",
  "metric_name": "build_time",
  "chart_type": "x",
  "center_line": 14.3,
  "ucl": 23.9,
  "lcl": 4.7,
  "data_points": 47,
  "rule_violations": [
    {
      "rule": "WE_1",
      "description": "Single point beyond 3σ",
      "point_index": 42,
      "point_value": 28.5,
      "severity": "CRITICAL"
    }
  ],
  "stability_status": "UNSTABLE",
  "last_evaluation": "2026-06-28T14:00:00Z",
  "tags": ["spc", "build-time", "unstable"],
  "embedding": [0.15, -0.08, 0.22, ...],
  "ttl_active_seconds": 86400,
  "ttl_recent_seconds": 604800
}
```

### 6.2 LTM — SPC Baseline Database

```json
{
  "fact_id": "fact-20260628-002",
  "category": "spc_baseline",
  "key": "pipeline-alpha_build-time_spc_baseline",
  "value": {
    "target_id": "ci-cd-pipeline-alpha",
    "metric_name": "build_time",
    "chart_type": "x",
    "center_line": 14.3,
    "ucl": 23.9,
    "lcl": 4.7,
    "sigma": 3.2,
    "specification_limits": {
      "usl": 20.0,
      "lsl": 5.0
    },
    "capability_indices": {
      "cp": 0.78,
      "cpk": 0.52
    },
    "sample_size": 47,
    "computation_method": "standard",
    "normality_test": {
      "test": "shapiro_wilk",
      "p_value": 0.12,
      "is_normal": true
    }
  },
  "source": "ARM-02-SPCController",
  "timestamp": "2026-06-28T14:00:00Z",
  "confidence": 0.98,
  "expiry": "2027-06-28T00:00:00Z",
  "chart_id": "chart-build-time-alpha",
  "ledger_hash": "sha256:def456..."
}
```

### 6.3 EM — SPC Evaluation Session Record

```json
{
  "session_id": "session-20260628-002",
  "chart_id": "chart-build-time-alpha",
  "persona_id": "D1",
  "arm_id": "ARM-02",
  "start_time": "2026-06-28T13:00:00Z",
  "end_time": "2026-06-28T14:00:00Z",
  "target_id": "ci-cd-pipeline-alpha",
  "metric_name": "build_time",
  "data_points_evaluated": 47,
  "rule_evaluations": {
    "WE_1": { "violations": 1, "points": [42] },
    "WE_2": { "violations": 0, "points": [] },
    "WE_3": { "violations": 1, "points": [38, 39, 40] },
    "WE_4": { "violations": 0, "points": [] },
    "WE_5": { "violations": 1, "points": [35, 36, 37, 38, 39, 40] },
    "WE_6": { "violations": 0, "points": [] },
    "WE_7": { "violations": 0, "points": [] },
    "WE_8": { "violations": 0, "points": [] }
  },
  "stability_classification": "UNSTABLE",
  "root_cause": "Recent dependency upgrade increased build time by 30% on average",
  "alerts_dispatched": ["D5", "D3"],
  "control_limit_recomputed": false,
  "embedding": [0.15, -0.08, 0.22, ...],
  "compression_ratio": 0.10,
  "ledger_hash": "sha256:def456..."
}
```

---

## 7. Actuators

| Output | Format | Destination | Generation Trigger |
|--------|--------|-------------|-------------------|
| **Control Chart** | PNG/SVG with rule annotations | Quality dashboard / S3 / Grafana | Every evaluation cycle |
| **Rule Violation Report** | JSON + Markdown | Alert system / P2 Ledger / Team channels | Any rule violation detected |
| **Process Stability Verdict** | JSON + Status badge | CI/CD gates / Quality portal | Every stability evaluation |
| **Capability Study** | PDF + JSON | Quality portal / Manufacturing system | Capability index computed |
| **Alert Dispatch** | Slack / Email / PagerDuty | D5 SRE / Team channels | Critical rule violation |
| **Dashboard Update** | Grafana JSON / Prometheus metrics | Monitoring stack | Real-time metric ingestion |
| **Ledger Entry** | Hash-chained record | P2 PostgreSQL | Every session completion |
| **Control Plan Update** | Markdown + JSON | Quality portal / CI config | Control limit recomputation |

### Actuator Output Schemas

```yaml
spc_evaluation_output:
  evaluation_id: "UUID"
  chart_id: "UUID"
  target_id: "string"
  timestamp: "ISO-8601"
  chart_type: "enum"
  center_line: "float"
  ucl: "float"
  lcl: "float"
  data_points: "int"
  rule_violations: "array[RuleViolation]"
  stability_status: "enum [STABLE, UNSTABLE, TRENDING, CYCLICAL, UNKNOWN]"
  capability_indices: "object (optional)"
  alert_required: "boolean"
  alert_severity: "enum [INFO, LOW, MEDIUM, HIGH, CRITICAL]"
  deliverable_uris: "array[string]"
  ledger_hash: "string"
```

---

## 8. Circuit Breaker

```yaml
circuit_breaker:
  name: "arm_02_cb"
  thresholds:
    consecutive_failures: 5
    failure_rate: 0.25
    slow_call_rate: 0.50
    slow_call_duration_ms: 60000
  states:
    - CLOSED: normal operation
    - OPEN: reject all invocations, return 503
    - HALF_OPEN: allow 1 test invocation after recovery_timeout
  recovery_timeout_ms: 30000
  half_open_max_calls: 1
  health_check_interval: 5000
  fallback:
    action: "return_last_known_control_limits_with_warn_status"
    stub_template: "d1/fallback_spc_summary.md"
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
      action: "degrade_to_cached_control_limits"
      stub_quality: "last_known_limits_with_staleness_warning"
      conditions: ["persistent_failure", "circuit_open"]
    escalate:
      enabled: true
      target: "human_quality_engineer"
      conditions: ["3_failed_retries", "control_limit_computation_failure", "data_corruption_detected"]
      notify: ["D5", "G1", "P2"]
    circuit_break:
      enabled: true
      action: "stop_and_queue"
      queue: "d1_arm02_dlq"
      conditions: ["failure_rate > 25%", "consecutive_failures >= 5"]
    compensate:
      enabled: true
      action: "revert_to_previous_control_limits"
      cleanup: ["revert_stm", "notify_stakeholders", "release_resources"]
      conditions: ["evaluation_aborted", "invalid_data_detected"]
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
| Process instability detected in tests | D7 Test Automator | `d1_to_d7_test_v1` | {instability_type, affected_tests, rule_violations, control_chart_data} | >5 rule violations or flaky test rate >10% | Async webhook + priority queue | D7 returns stabilized test suite; D1 re-evaluates SPC |
| Production metric anomaly detected | D5 SRE Commander | `d1_to_d5_alert_v1` | {metric_name, anomaly_type, severity, control_chart_snapshot} | Critical rule violation (WE_1) or Cpk < 0.67 | Real-time alert + incident channel | D5 acknowledges alert; D1 receives incident resolution status |
| Continuous monitoring deployment needed | G6 The Sentinel | `d1_to_g6_monitoring_v1` | {control_plan, metrics, thresholds, alert_rules, dashboard_config} | Process stability critical | Async webhook + monitoring config | G6 deploys monitoring; D1 receives alert on rule violations |
| Capability index below minimum | D3 Delivery Captain | `d1_to_d3_planning_v1` | {capability_study, improvement_actions, effort_estimates, priority} | Cpk < 1.0 or improvement effort > 20 days | API POST + planning sync | D3 returns scheduled improvement plan; D1 monitors control plan |
| Quality certification of process stability | G1 The Arbiter | `d1_to_g1_certification_v1` | {spc_report, stability_verdict, capability_indices, compliance_mapping} | Process unstable for >30 days or Cpk < 1.33 | API POST + 24h SLA | G1 returns certification verdict; D1 applies conditions |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team  
**Classification:** Internal — Arm Specification  
**Next Review:** 2026-07-28
