# ARM_04_D1_QualityAuditor.md
## Persona D1 — The Quality Engineer | Secondary Arm 4: Quality Auditor

**Arm ID:** `ARM-04`  
**Name:** `quality_auditor`  
**Type:** Secondary  
**Status:** Production-ready  
**Version:** 1.0.0  
**Owner:** D1 The Quality Engineer  
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`

---

## 1. Identity

```yaml
arm_identity:
  id: ARM-04
  name: "Quality Auditor"
  persona_id: D1
  version: "1.0.0"
  purpose: >
    Comprehensive quality audit of software development practices, including code review
    assessment, defect density analysis, DPMO/FTY/RTY computation, and overall quality scorecard
    generation. Evaluates review velocity, thoroughness, and distribution; computes defect
    metrics per KLOC; translates defects to DPMO and sigma levels; computes first-time yield
    (FTY) and rolled throughput yield (RTY) for multi-stage processes. Every audit is
    evidence-backed and every metric is traceable to source data.
```

---

## 2. Sensors

| Sensor ID | Name | Trigger | Filter | Arm Action | Handler Module |
|-----------|------|---------|--------|------------|---------------|
| `SENS-D1-04A` | `monthly_audit_sensor` | Cron: `0 0 1 * *` UTC | Repositories with `audit_due=true` | Full quality audit: reviews, defects, coverage, scorecard | `sensors.monthly_audit_handler` |
| `SENS-D1-04B` | `pre_release_audit_sensor` | Release candidate tagged | `release_tag` matches `v*` pattern | Pre-release quality audit with certification recommendation | `sensors.pre_release_handler` |
| `SENS-D1-04C` | `defect_spike_sensor` | Event: defect count exceeds 2σ from baseline | `defect_rate` > `baseline_defect_rate * 2` | Emergency quality audit with root cause analysis | `sensors.defect_spike_handler` |
| `SENS-D1-04D` | `cross_persona_chained_sensor` | Hook: `g1_to_d1_audit_v1` | G1 requests quality certification evidence | Compile full audit evidence package for certification | `sensors.hook_handler` |
| `SENS-D1-04E` | `code_review_batch_sensor` | Weekly PR batch completion | `pr_count` > 0 AND `review_data_available=true` | Review practice assessment and velocity computation | `sensors.review_batch_handler` |
| `SENS-D1-04F` | `quality_scorecard_request` | On-demand scorecard request | `target_type` in [repository, team, project, system] | Generate multi-dimensional quality scorecard | `sensors.scorecard_request_handler` |

### Sensor Event Schemas

```yaml
monthly_audit_event:
  audit_id: "UUID"
  target_id: "string (repository, team, or project ID)"
  target_type: "enum [repository, team, project, system]"
  audit_period:
    start: "ISO-8601"
    end: "ISO-8601"
  data_uris:
    - defect_logs: "string (S3/MinIO)"
    - pr_data: "string (S3/MinIO)"
    - coverage_reports: "string (S3/MinIO)"
    - test_results: "string (S3/MinIO)"
    - commit_history: "string (S3/MinIO)"
  standards:
    target_defect_density: "float (default: 0.2 defects/KLOC)"
    target_review_velocity_days: "float (default: 2.0)"
    target_review_thoroughness: "float (default: 3.0 comments/PR)"
    target_dpmo: "int (default: 3400)"
    target_sigma_level: "float (default: 4.0)"
  priority: "enum [p0, p1, p2, p3]"
  callback_url: "string (optional)"

pre_release_audit_event:
  release_id: "UUID"
  repository_id: "string"
  release_tag: "string"
  commit_range: "string (base..head)"
  target_grade: "enum [A, B, C] (default: B)"
  data_uris: "object (same as monthly audit)"
```

---

## 3. Tools

| Tool ID | Name | Description | Input Schema | Output Schema | Timeout | Retry | Permissions | Error Handling |
|---------|------|-------------|-------------|-------------|---------|-------|-------------|---------------|
| `TOOL-D1-09` | `DefectAnalyzer` | Defect density, trend, root cause classification, and Pareto analysis | `d1/tools/defect_input.json` | `d1/tools/defect_output.json` | 120s | 2× exponential | `d1-quality:analyze` | Fallback: raw defect count |
| `TOOL-D1-10` | `QualityScorecard` | Multi-dimensional quality scorecard generator with grade computation | `d1/tools/scorecard_input.json` | `d1/tools/scorecard_output.json` | 60s | 2× fixed | `d1-quality:report` | Fallback: text-only scorecard |
| `TOOL-D1-11` | `DPMOCalculator` | DPMO and sigma level translation with confidence intervals | `d1/tools/dpmo_input.json` | `d1/tools/dpmo_output.json` | 30s | 2× fixed | `d1-quality:calculate` | Fallback: manual DPMO formula reference |
| `TOOL-D1-12` | `CodeReviewAuditor` | Code review practice assessment — velocity, thoroughness, distribution | `d1/tools/review_audit_input.json` | `d1/tools/review_audit_output.json` | 120s | 2× exponential | `d1-quality:audit` | Fallback: basic review count |
| `TOOL-D1-39` | `FTYCalculator` | First Time Yield calculator per stage and overall process | `d1/tools/fty_input.json` | `d1/tools/fty_output.json` | 60s | 2× fixed | `d1-quality:calculate` | Fallback: simple pass rate |
| `TOOL-D1-40` | `RTYCalculator` | Rolled Throughput Yield calculator for multi-stage processes | `d1/tools/rty_input.json` | `d1/tools/rty_output.json` | 60s | 2× fixed | `d1-quality:calculate` | Fallback: product of FTYs |
| `TOOL-D1-41` | `OEECalculator` | Overall Equipment Effectiveness for CI/CD pipeline efficiency | `d1/tools/oee_input.json` | `d1/tools/oee_output.json` | 60s | 2× fixed | `d1-quality:calculate` | Fallback: availability × performance |
| `TOOL-D1-42` | `ParetoAnalyzer` | Pareto analysis (80/20) for defect types, modules, and root causes | `d1/tools/pareto_input.json` | `d1/tools/pareto_output.json` | 120s | 2× exponential | `d1-quality:analyze` | Fallback: simple frequency sort |
| `TOOL-D1-43` | `ReviewerDistributionAnalyzer` | Reviewer load balancing, expertise matching, and bias detection | `d1/tools/reviewer_dist_input.json` | `d1/tools/reviewer_dist_output.json` | 120s | 2× exponential | `d1-quality:audit` | Fallback: basic reviewer counts |
| `TOOL-D1-44` | `DefectEscapeRateTracker` | Track defects escaped to production vs. caught in review/testing | `d1/tools/escape_rate_input.json` | `d1/tools/escape_rate_output.json` | 120s | 2× exponential | `d1-quality:track` | Fallback: manual classification |
| `TOOL-D1-45` | `QualityTrendForecaster` | Forecast quality metric trends using ARIMA and exponential smoothing | `d1/tools/forecast_input.json` | `d1/tools/forecast_output.json` | 180s | 2× exponential | `d1-quality:forecast` | Fallback: linear trend projection |
| `TOOL-D1-46` | `BenchmarkComparator` | Compare quality metrics against industry benchmarks and historical baselines | `d1/tools/benchmark_input.json` | `d1/tools/benchmark_output.json` | 120s | 2× fixed | `d1-quality:compare` | Fallback: internal historical comparison only |
| `TOOL-D1-47` | `AuditReportGenerator` | Generate formatted quality audit reports with executive and technical sections | `d1/tools/audit_report_input.json` | `d1/tools/audit_report_output.json` | 180s | 2× exponential | `d1-quality:report` | Fallback: plain text summary |
| `TOOL-D1-48` | `CostOfQualityCalculator` | Cost of poor quality (COPQ) and cost of good quality (COGQ) analysis | `d1/tools/coq_input.json` | `d1/tools/coq_output.json` | 120s | 2× fixed | `d1-quality:calculate` | Fallback: estimated cost ranges |
| `TOOL-D1-49` | `ComplianceMapper` | Map quality metrics to ISO 9001, CMMI, and industry compliance frameworks | `d1/tools/compliance_input.json` | `d1/tools/compliance_output.json` | 120s | 2× fixed | `d1-quality:map` | Fallback: basic framework checklist |
| `TOOL-D1-50` | `QualityScorecardDashboard` | Update quality scorecard dashboard with latest audit results | `d1/tools/qs_dashboard_input.json` | `d1/tools/qs_dashboard_output.json` | 60s | 2× fixed | `d1-quality:dashboard` | Fallback: batch update on next cycle |

---

## 4. Skills

| Skill | Name | Description | Steps | Validation | Error Handling |
|-------|------|-------------|-------|------------|---------------|
| `SKILL-D1-06` | `DefectAnalysis` | Defect density, trend, root cause classification, and Pareto analysis | 1. Ingest defect logs from issue tracker. 2. Classify defects by type, severity, module. 3. Compute defect density (defects/KLOC). 4. Run Pareto analysis (80/20 rule). 5. Identify systemic vs. sporadic defects. 6. Trend analysis with forecast. | Defect density matches KLOC denominator; Pareto chart validated; trends statistically significant (p < 0.05); DPMO computation correct | Fallback to manual defect count; flag data quality issues; escalate if defect classification ambiguous |
| `SKILL-D1-07` | `QualityAuditing` | Multi-dimensional quality audit with scorecard generation | 1. Collect quality metrics across all dimensions. 2. Normalize metrics to 0-100 scale. 3. Apply weighting based on target audience. 4. Compute overall score with grade. 5. Identify priority gaps. 6. Generate actionable recommendations. | Score computation transparent and reproducible; weights configurable; all sub-metrics within valid ranges; grades match standard scale (A≥90, B≥80, C≥70, D≥60, F<60) | Fallback to unweighted average; flag missing metrics; escalate to G1 if certification grade contested |
| `SKILL-D1-09` | `CodeReviewAudit` | Assessment of code review practices and their quality impact | 1. Ingest PR data from Git platform. 2. Compute review velocity (median time to first review, time to approve). 3. Compute review thoroughness (comments per PR, review depth). 4. Analyze reviewer distribution (load balancing, expertise matching). 5. Correlate review metrics with defect escape rate. | Data matches Git API response; velocity metrics use median not mean; correlation statistically significant; reviewer anonymity preserved | Fallback to basic counts; flag API access issues; escalate to D9 for review bot improvements |
| `SKILL-D1-10` | `QualityImprovementPlanning` | Generate prioritized improvement plans with effort estimates and ROI | 1. Consolidate findings from all quality analyses. 2. Score improvements by effort, impact, and risk. 3. Generate Pareto-optimal improvement set. 4. Create phased roadmap with milestones. 5. Estimate effort using historical data. 6. Define success metrics and control plan. | Effort estimates within ±30% of historical; improvement plan covers all fail/marginal metrics; milestones achievable; ROI positive for all recommended items | Fallback to generic recommendations; flag missing historical data; escalate to D3 for resource planning |
| `SKILL-D1-11` | `QualityAuditing` | Multi-dimensional quality audit including code review and defect dimensions | 1. Collect quality metrics across all dimensions including reviews and defects. 2. Normalize metrics to 0-100 scale. 3. Apply weighting based on target audience. 4. Compute overall score with grade. 5. Identify priority gaps. 6. Generate actionable recommendations. | Score computation transparent and reproducible; weights configurable; all sub-metrics within valid ranges; grades match standard scale | Fallback to unweighted average; flag missing metrics; escalate to G1 if certification grade contested |

---

## 5. Plugins

| Plugin | Name | Type | Config | Version | Dependencies |
|--------|------|------|--------|---------|-------------|
| `PLUGIN-D1-01` | `DMAIC_Framework` | methodology_engine | DMAIC phase manager for audit-improvement-control cycles | 1.0.0 | `pydantic>=2.0`, `numpy>=1.26.0`, `scipy>=1.12.0` |
| `PLUGIN-D1-02` | `DevAgent` | quality_integration | DevAgent integration for code review and defect tracking | 1.0.0 | `gitpython>=3.1.0`, `pygithub>=2.0.0`, `jira>=3.5.0` |
| `PLUGIN-D1-09` | `Defect_Tracker` | defect_integration | Issue tracker integration for defect ingestion and Pareto analysis | 1.0.0 | `jira>=3.5.0`, `github>=2.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-10` | `Quality_Scorecard` | reporting | Multi-dimensional quality scorecard generator with branding | 1.0.0 | `jinja2>=3.1.0`, `reportlab>=4.0.0`, `openpyxl>=3.1.0` |
| `PLUGIN-D1-11` | `DPMO_Calculator` | dpmo_engine | DPMO, sigma level, FTY, RTY, and OEE computation engine | 1.0.0 | `numpy>=1.26.0`, `scipy>=1.12.0` |
| `PLUGIN-D1-12` | `Review_Auditor` | review_integration | Git platform PR data ingestion and review metrics computation | 1.0.0 | `pygithub>=2.0.0`, `gitlab-python>=4.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-35` | `Jira_Integration` | defect_tracker | Jira integration for defect ingestion, classification, and trend analysis | 1.0.0 | `jira>=3.5.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-36` | `GitHub_PR` | review_platform | GitHub PR data ingestion for review velocity and thoroughness | 1.0.0 | `pygithub>=2.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-37` | `GitLab_MR` | review_platform | GitLab MR data ingestion for review metrics | 1.0.0 | `gitlab-python>=4.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-38` | `Bitbucket_PR` | review_platform | Bitbucket PR data ingestion for review metrics | 1.0.0 | `bitbucket-api>=1.0.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-39` | `ISO_9001_Mapper` | compliance | ISO 9001 quality management system mapping and gap analysis | 1.0.0 | `jinja2>=3.1.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-40` | `CMMI_Mapper` | compliance | CMMI maturity level mapping and assessment | 1.0.0 | `jinja2>=3.1.0`, `pandas>=2.2.0` |
| `PLUGIN-D1-41` | `Pareto_Visualizer` | visualization | Pareto chart generator with 80/20 line and annotations | 1.0.0 | `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `theme-factory>=1.0.0` |
| `PLUGIN-D1-42` | `FastAPI_Audit` | api_server | FastAPI >=0.104.0 router for audit endpoints with Pydantic v2 schemas | 1.0.0 | `fastapi>=0.104.0`, `pydantic>=2.0`, `uvicorn>=0.23.0` |
| `PLUGIN-D1-43` | `PostgreSQL_Audit` | database | PostgreSQL 15 with JSONB for audit records and scorecard storage | 1.0.0 | `psycopg2>=2.9.0`, `pgvector>=0.2.0` |

---

## 6. Memory

### 6.1 STM — Active Audit Session Cache

```json
{
  "turn_id": "turn-arm04-20260628-001",
  "timestamp": "2026-06-28T02:00:00Z",
  "persona_id": "D1",
  "arm_id": "ARM-04",
  "audit_id": "audit-gitsentinental-2026q2",
  "target_id": "GitSentinental",
  "target_type": "repository",
  "audit_period": {
    "start": "2026-04-01T00:00:00Z",
    "end": "2026-06-30T23:59:59Z"
  },
  "scorecard": {
    "coverage": { "value": 87, "target": 80, "status": "PASS" },
    "test_reliability": { "value": 94, "target": 95, "status": "MARGINAL" },
    "defect_density": { "value": 0.12, "target": 0.2, "status": "PASS" },
    "cpk": { "value": 1.2, "target": 1.33, "status": "MARGINAL" },
    "review_velocity": { "value": 2.3, "target": 2.0, "status": "PASS" },
    "review_thoroughness": { "value": 4.2, "target": 3.0, "status": "PASS" },
    "overall": { "score": 87, "grade": "B+" }
  },
  "priority_gaps": ["test_reliability", "cpk"],
  "status": "complete",
  "tags": ["audit", "scorecard", "quarterly"],
  "embedding": [0.22, -0.15, 0.55, ...],
  "ttl_active_seconds": 86400,
  "ttl_recent_seconds": 604800
}
```

### 6.2 LTM — Quality Audit Baseline Database

```json
{
  "fact_id": "fact-20260628-004",
  "category": "quality_audit_baseline",
  "key": "gitsentinental_2026q2_audit_baseline",
  "value": {
    "target_id": "GitSentinental",
    "audit_period": "2026-Q2",
    "overall_score": 87,
    "overall_grade": "B+",
    "dimensions": {
      "coverage": { "score": 87, "target": 80, "weight": 0.20 },
      "test_reliability": { "score": 94, "target": 95, "weight": 0.20 },
      "defect_density": { "score": 92, "target": 0.2, "weight": 0.15 },
      "process_capability": { "score": 78, "target": 1.33, "weight": 0.15 },
      "review_velocity": { "score": 88, "target": 2.0, "weight": 0.15 },
      "review_thoroughness": { "score": 95, "target": 3.0, "weight": 0.15 }
    },
    "dpmo": 4500,
    "sigma_level": 4.1,
    "fty": 0.96,
    "rty": 0.89,
    "defect_escape_rate": 0.04
  },
  "source": "ARM-04-QualityAuditor",
  "timestamp": "2026-06-28T02:15:00Z",
  "confidence": 0.95,
  "expiry": "2027-06-28T00:00:00Z",
  "audit_id": "audit-gitsentinental-2026q2",
  "ledger_hash": "sha256:jkl012..."
}
```

### 6.3 EM — Quality Audit Session Record

```json
{
  "session_id": "session-20260628-004",
  "audit_id": "audit-gitsentinental-2026q2",
  "persona_id": "D1",
  "arm_id": "ARM-04",
  "start_time": "2026-06-28T02:00:00Z",
  "end_time": "2026-06-28T02:30:00Z",
  "target_id": "GitSentinental",
  "target_type": "repository",
  "audit_period": {
    "start": "2026-04-01T00:00:00Z",
    "end": "2026-06-30T23:59:59Z"
  },
  "metrics_computed": {
    "coverage": 87.0,
    "test_reliability": 94.0,
    "defect_density": 0.12,
    "defects_total": 14,
    "kloc": 117,
    "dpmo": 4500,
    "sigma_level": 4.1,
    "cp": 1.4,
    "cpk": 1.2,
    "fty": 0.96,
    "rty": 0.89,
    "review_velocity_median": 2.3,
    "review_thoroughness_mean": 4.2
  },
  "review_findings": {
    "total_prs": 142,
    "reviewed_prs": 138,
    "median_time_to_first_review_hours": 18,
    "median_time_to_approve_hours": 55,
    "mean_comments_per_pr": 4.2,
    "reviewer_count": 8,
    "reviewer_distribution_gini": 0.34
  },
  "defect_findings": {
    "total_defects": 14,
    "critical": 0,
    "high": 2,
    "medium": 6,
    "low": 6,
    "by_module": { "auth.py": 4, "api.py": 3, "models.py": 2 },
    "pareto_top_modules": ["auth.py", "api.py", "models.py"]
  },
  "scorecard": {
    "overall_score": 87,
    "overall_grade": "B+",
    "priority": "Improve test reliability (flaky tests in test_security.py, test_api.py)"
  },
  "embedding": [0.22, -0.15, 0.55, ...],
  "compression_ratio": 0.10,
  "ledger_hash": "sha256:jkl012..."
}
```

---

## 7. Actuators

| Output | Format | Destination | Generation Trigger |
|--------|--------|-------------|-------------------|
| **Quality Scorecard** | Interactive dashboard + PDF + JSON + Excel | Executive dashboard / Quality portal / Team channels | Every audit completion |
| **Audit Report** | PDF + Markdown + JSON | Quality portal / S3 / Git repo | Every audit completion |
| **Defect Density Report** | PNG + JSON + Markdown | Quality portal / Team channels | Every defect analysis |
| **DPMO/Sigma Report** | JSON + Markdown | Quality portal / Executive dashboard | Every DPMO computation |
| **Review Practice Report** | JSON + Markdown | Team leads / HR / Quality portal | Every review audit |
| **Pareto Chart** | PNG/SVG + Data table | Quality portal / S3 | Every defect Pareto analysis |
| **Certification Package** | ZIP (PDF + JSON + Evidence) | G1 Arbiter / Compliance | Pre-release or certification request |
| **Ledger Entry** | Hash-chained record | P2 PostgreSQL | Every session completion |
| **Improvement Plan** | Markdown + Gantt JSON | Project management / D3 integration | Every audit with grade < A |
| **Compliance Mapping** | Excel + JSON | Compliance team / ISO audit | Quarterly or on-demand |

### Actuator Output Schemas

```yaml
quality_audit_output:
  audit_id: "UUID"
  target_id: "string"
  target_type: "enum"
  audit_period:
    start: "ISO-8601"
    end: "ISO-8601"
  timestamp: "ISO-8601"
  scorecard:
    overall_score: "float (0-100)"
    overall_grade: "enum [A, B, C, D, F]"
    dimensions: "object (per-dimension score, target, status)"
  metrics:
    coverage: "float"
    test_reliability: "float"
    defect_density: "float"
    dpmo: "int"
    sigma_level: "float"
    cp: "float"
    cpk: "float"
    fty: "float"
    rty: "float"
  review_metrics:
    velocity_median_days: "float"
    thoroughness_mean: "float"
    reviewer_distribution_gini: "float"
  defect_findings: "object"
  priority_gaps: "array[string]"
  recommendations: "array[string]"
  certification_recommendation: "enum [CERTIFIED, CONDITIONAL, NOT_CERTIFIED]"
  deliverable_uris: "array[string]"
  ledger_hash: "string"
```

---

## 8. Circuit Breaker

```yaml
circuit_breaker:
  name: "arm_04_cb"
  thresholds:
    consecutive_failures: 5
    failure_rate: 0.25
    slow_call_rate: 0.50
    slow_call_duration_ms: 900000
  states:
    - CLOSED: normal operation
    - OPEN: reject all invocations, return 503
    - HALF_OPEN: allow 1 test invocation after recovery_timeout
  recovery_timeout_ms: 30000
  half_open_max_calls: 1
  health_check_interval: 5000
  fallback:
    action: "return_cached_scorecard_with_staleness_warning"
    stub_template: "d1/fallback_scorecard.md"
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
      action: "degrade_to_cached_scorecard"
      stub_quality: "last_known_scorecard_with_staleness_warning"
      conditions: ["persistent_failure", "circuit_open"]
    escalate:
      enabled: true
      target: "human_quality_engineer"
      conditions: ["3_failed_retries", "data_source_unavailable", "audit_evidence_missing"]
      notify: ["D5", "G1", "P2"]
    circuit_break:
      enabled: true
      action: "stop_and_queue"
      queue: "d1_arm04_dlq"
      conditions: ["failure_rate > 25%", "consecutive_failures >= 5"]
    compensate:
      enabled: true
      action: "rollback_audit_session"
      cleanup: ["revert_stm", "notify_stakeholders", "release_resources"]
      conditions: ["audit_aborted", "data_integrity_failure"]
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
| Quality certification required | G1 The Arbiter | `d1_to_g1_certification_v1` | {quality_scorecard, sigma_level, cpk, dpmo, fty, rty, compliance_mapping} | Sigma < 3.0 or Cpk < 1.0 or grade < C | API POST + 24h SLA | G1 returns certification verdict; D1 applies conditions if conditional |
| Code review practice improvement | D9 Forward Engineer | `d1_to_d9_quality_v1` | {review_audit_findings, fix_recommendations, bot_improvements, priority} | Reviewer distribution Gini > 0.5 or velocity > 5 days | Async webhook + priority queue | D9 returns improved review automation; D1 re-audits in next cycle |
| Defect reduction initiatives | D3 Delivery Captain | `d1_to_d3_planning_v1` | {defect_analysis, pareto_findings, improvement_actions, effort_estimates} | Defect density > 1.0 or systemic defects > 5 | API POST + planning sync | D3 returns scheduled plan; D1 monitors defect trends |
| Quality monitoring deployment | G6 The Sentinel | `d1_to_g6_monitoring_v1` | {quality_scorecard, metrics, thresholds, alert_rules, dashboard_config} | Critical grade or certification failed | Async webhook + monitoring config | G6 deploys monitoring; D1 receives alert on metric degradation |
| Test reliability improvements | D7 Test Automator | `d1_to_d7_test_v1` | {test_reliability_findings, flaky_tests, stabilization_plan, priority} | Test reliability < 90% or flaky rate > 5% | API POST + callback URL | D7 returns stabilized test suite; D1 validates reliability improvement |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team  
**Classification:** Internal — Arm Specification  
**Next Review:** 2026-07-28
