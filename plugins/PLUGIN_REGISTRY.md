# PLUGIN_REGISTRY.md
## Persona D1 — The Quality Engineer | Plugin Configurations

**Version:** 1.0.0
**Status:** Production-ready
**Date:** 2026-06-28
**Owner:** D1 The Quality Engineer
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`
**Master Strategy:** `C:\KimiWork Projects\GAI-OBSERVE-DESIGN\skills-hooks-plugins-strategy\STRATEGY.md`

---

## 1. Plugin Registry Overview

All plugins for D1 arms are configured in this document. Each plugin has installation, configuration, authentication, health check, quota, and security specifications. Plugins are organized by layer: Methodology Engine, Quality Integration, Reproducibility Engine, Statistical Engine, Visualization, Rule Engine, Coverage Engine, Flaky Engine, Defect Integration, Reporting, DPMO Engine, Review Integration, Test Optimization, Tracking Engine, and Planning Engine.

---

## 2. Methodology Engine Plugins

### DMAIC_Framework

```yaml
plugin:
  name: "DMAIC_Framework"
  type: "methodology_engine"
  version: "1.0.0"
  description: "DMAIC phase manager, evidence gate validator, control plan template engine, and improvement roadmap generator. Enforces phase transitions with evidence requirements and blocks transitions without sufficient data."
  installation: "pip install pydantic>=2.0 numpy>=1.26.0 scipy>=1.12.0"
  config:
    phase_definitions:
      define: { required_evidence: ["scope_document", "boundary_definition", "ctq_list"] }
      measure: { required_evidence: ["metric_baseline", "measurement_system_analysis", "data_collection_plan"] }
      analyze: { required_evidence: ["root_cause_analysis", "statistical_verification", "data_stratification"] }
      improve: { required_evidence: ["solution_selection", "pilot_results", "implementation_plan"] }
      control: { required_evidence: ["control_plan", "monitoring_system", "response_plan"] }
    evidence_gate: "strict"
    phase_timeout_hours: 168
  auth:
    method: "JWT + in-memory role validation"
    rbac: "d1-quality:assess"
  health_check:
    command: "python -c 'import pydantic; print(pydantic.__version__)'"
    expected_output: "contains '2.'"
  quotas:
    max_concurrent_assessments: 10
    max_evidence_items_per_phase: 50
  security:
    data_locality: "true"
    audit: "all phase transitions logged to P2"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "Phase management and evidence validation for all DMAIC assessments and quality audits"
```

### DevAgent

```yaml
plugin:
  name: "DevAgent"
  type: "quality_integration"
  version: "1.0.0"
  description: "DevAgent quality arm integration for code review data ingestion, defect tracking, and code quality metric extraction from Git repositories and issue trackers."
  installation: "pip install gitpython>=3.1.0 pygithub>=2.0.0 jira>=3.5.0"
  config:
    git_platforms: ["github", "gitlab", "bitbucket"]
    issue_trackers: ["jira", "github_issues", "gitlab_issues"]
    code_metrics: ["lines_of_code", "cyclomatic_complexity", "cognitive_complexity", "code_churn"]
    review_metrics: ["review_count", "review_time", "comment_count", "approval_rate"]
  auth:
    method: "OAuth 2.0 + JWT (app layer)"
    token_source: "Vault"
    scopes: ["repo", "read:org", "read:discussion"]
  health_check:
    command: "python -c 'import git; print(git.__version__)'"
    expected_output: "success"
  quotas:
    max_repos_per_scan: 50
    max_prs_per_repo: 1000
    max_issues_per_scan: 5000
  security:
    data_locality: "true"
    pii_handling: "reviewer_anonymization"
    audit: "all data access logged to P2"
  arm_integration:
    arms: ["ARM-01", "ARM-03", "ARM-04"]
    usage: "Code quality metrics, defect tracking, and review data for all D1 analyses"
```

---

## 3. Reproducibility & Test Engine Plugins

### ReproSense

```yaml
plugin:
  name: "ReproSense"
  type: "reproducibility_engine"
  version: "1.0.0"
  description: "ReproSense coverage and flaky test integration with run history analysis, coverage XML parsing, and test result aggregation. Validates coverage gap maps against coverage.xml (R-ARM-REPRO-1) and flaky verdicts against run statistics (R-ARM-REPRO-2)."
  installation: "pip install coverage>=7.3.0 pytest>=7.4.0 pytest-rerunfailures>=12.0"
  config:
    coverage_formats: ["coverage.xml", "lcov", "jacoco", "cobertura", "clover"]
    test_frameworks: ["pytest", "jest", "junit", "go_test", "cargo_test", "xunit"]
    flaky_detection_runs: 10
    coverage_threshold: 0.80
  auth:
    method: "JWT (app layer)"
    rbac: "d1-quality:analyze"
  health_check:
    command: "python -c 'import coverage; print(coverage.__version__)'"
    expected_output: "contains '7.'"
  quotas:
    max_coverage_files_per_scan: 100
    max_test_runs_per_analysis: 10000
    max_repos_per_scan: 50
  security:
    data_locality: "true"
    audit: "all coverage analyses logged to P2"
  arm_integration:
    arms: ["ARM-03", "ARM-04"]
    usage: "Coverage gap analysis, flaky test detection, and test reliability assessment"
```

### SPC_Tool

```yaml
plugin:
  name: "SPC_Tool"
  type: "statistical_engine"
  version: "1.0.0"
  description: "Core Statistical Process Control computation engine with control limit computation, process capability index calculation, and normality testing. All computations deterministic and validated against NumPy reference fixtures (R-GATE-2)."
  installation: "pip install numpy>=1.26.0 scipy>=1.12.0 statsmodels>=0.14.0"
  config:
    control_limit_method: "standard (3-sigma)"
    capability_indices: ["Cp", "Cpk", "Pp", "Ppk"]
    normality_tests: ["shapiro_wilk", "anderson_darling", "kolmogorov_smirnov"]
    deterministic_validation: true
    numpy_fixture_path: "/opt/d1/fixtures/spc_reference.npz"
  auth:
    method: "none (local library)"
  health_check:
    import: "import numpy; print(numpy.__version__)"
    expected_output: "contains '1.26'"
  quotas:
    max_data_points: 1000000
    max_subgroups: 10000
  security:
    data_locality: "true"
    deterministic_core: "true — R-GATE-2"
  arm_integration:
    arms: ["ARM-01", "ARM-02"]
    usage: "SPC math, control limits, capability indices for all D1 control and assessment arms"
```

---

## 4. Visualization & Rule Engine Plugins

### Control_Chart

```yaml
plugin:
  name: "Control_Chart"
  type: "visualization"
  version: "1.0.0"
  description: "Control chart generator with Western Electric rule annotations, specification limit overlays, trend lines, and GAI-OBSERVE visual theme. Supports X-bar, R, S, X, MR, p, np, c, u chart types with accessibility alt text."
  installation: "pip install matplotlib>=3.8.0 seaborn>=0.13.0 theme-factory>=1.0.0"
  config:
    chart_types: ["xbar", "r", "s", "x", "mr", "p", "np", "c", "u"]
    annotation_rules: ["WE_1", "WE_2", "WE_3", "WE_4", "WE_5", "WE_6", "WE_7", "WE_8"]
    theme: "gai-observe-dark"
    output_formats: ["png", "svg", "pdf"]
    dpi_web: 150
    dpi_print: 300
    accessibility: true
  auth:
    method: "none (local library)"
  health_check:
    import: "import matplotlib; print(matplotlib.__version__)"
    expected_output: "contains '3.8'"
  quotas:
    max_data_points_per_chart: 10000
    max_charts_per_batch: 50
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-01", "ARM-02"]
    usage: "Control chart generation for SPC monitoring and Six Sigma assessments"
```

### Western_Electric

```yaml
plugin:
  name: "Western_Electric"
  type: "rule_engine"
  version: "1.0.0"
  description: "Western Electric rule engine with all 8 rules implemented: WE1 (single point beyond 3σ), WE2 (2 of 3 beyond 2σ), WE3 (4 of 5 beyond 1σ), WE4 (8 consecutive on one side), WE5 (6 trending), WE6 (14 alternating), WE7 (15 in zone C), WE8 (8 beyond 1σ both sides). Configurable thresholds and custom rule support."
  installation: "pip install numpy>=1.26.0 pandas>=2.2.0"
  config:
    rules:
      WE_1: { enabled: true, threshold: "3-sigma", description: "Single point beyond 3σ" }
      WE_2: { enabled: true, threshold: "2-sigma", consecutive: 3, required: 2 }
      WE_3: { enabled: true, threshold: "1-sigma", consecutive: 5, required: 4 }
      WE_4: { enabled: true, consecutive: 8, description: "8 consecutive on one side of center line" }
      WE_5: { enabled: true, consecutive: 6, description: "6 consecutive trending up or down" }
      WE_6: { enabled: true, consecutive: 14, description: "14 consecutive alternating up and down" }
      WE_7: { enabled: true, consecutive: 15, zone: "C", description: "15 consecutive in zone C" }
      WE_8: { enabled: true, threshold: "1-sigma", consecutive: 8, both_sides: true, description: "8 consecutive beyond 1σ on both sides" }
    custom_rules_supported: true
  auth:
    method: "none (local library)"
  health_check:
    import: "import numpy; print(numpy.__version__)"
    expected_output: "contains '1.26'"
  quotas:
    max_data_points_per_evaluation: 100000
  security:
    data_locality: "true"
    deterministic_core: "true"
  arm_integration:
    arms: ["ARM-02"]
    usage: "Western Electric rule evaluation for all SPC monitoring and control chart analysis"
```

---

## 5. Coverage & Flaky Test Plugins

### Coverage_Analyzer

```yaml
plugin:
  name: "Coverage_Analyzer"
  type: "coverage_engine"
  version: "1.0.0"
  description: "Coverage XML parser, gap mapper, branch-level analysis, and test suite optimizer. Parses coverage.xml, lcov, JaCoCo, Cobertura, and Clover formats. Maps coverage gaps to source code AST for branch, condition, and edge case identification."
  installation: "pip install coverage>=7.3.0 lxml>=4.9.0 pandas>=2.2.0"
  config:
    supported_formats: ["coverage.xml", "lcov", "jacoco", "cobertura", "clover"]
    ast_analysis: true
    gap_map_resolution: "branch"
    risk_scoring: true
  auth:
    method: "none (local library)"
  health_check:
    import: "import coverage; print(coverage.__version__)"
    expected_output: "contains '7.'"
  quotas:
    max_coverage_files_per_scan: 100
    max_lines_per_file: 100000
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-03"]
    usage: "Coverage gap analysis and branch-level mapping for test gap analysis"
```

### Flaky_Test_Detector

```yaml
plugin:
  name: "Flaky_Test_Detector"
  type: "flaky_engine"
  version: "1.0.0"
  description: "Flaky test statistical detection and root cause classification engine. Computes pass rate variance, binomial proportion confidence intervals, and classifies root causes: race condition, timeout misconfiguration, external dependency failure, data contamination, environment variance."
  installation: "pip install scipy>=1.12.0 numpy>=1.26.0 pandas>=2.2.0"
  config:
    min_runs_for_detection: 10
    flaky_threshold: 0.05
    confidence_level: 0.95
    root_cause_classification: true
    classification_model: "statistical_heuristic"
  auth:
    method: "none (local library)"
  health_check:
    import: "import scipy; print(scipy.__version__)"
    expected_output: "contains '1.12'"
  quotas:
    max_tests_per_analysis: 10000
    max_runs_per_test: 1000
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-03"]
    usage: "Flaky test detection and root cause classification for test gap analysis"
```

---

## 6. Defect & Reporting Plugins

### Defect_Tracker

```yaml
plugin:
  name: "Defect_Tracker"
  type: "defect_integration"
  version: "1.0.0"
  description: "Issue tracker integration for defect ingestion, classification, trend analysis, and Pareto analysis. Supports Jira, GitHub Issues, and GitLab Issues with configurable classification schemas."
  installation: "pip install jira>=3.5.0 github>=2.0.0 pandas>=2.2.0"
  config:
    platforms: ["jira", "github", "gitlab"]
    classification_fields: ["type", "severity", "module", "root_cause", "detection_phase"]
    pareto_enabled: true
    trend_forecast: true
  auth:
    method: "OAuth 2.0 + JWT (app layer)"
    token_source: "Vault"
  health_check:
    command: "python -c 'import jira; print(jira.__version__)'"
    expected_output: "success"
  quotas:
    max_issues_per_scan: 10000
    max_platforms_per_scan: 3
  security:
    data_locality: "true"
    pii_handling: "reporter_anonymization"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "Defect ingestion, classification, and Pareto analysis for Six Sigma and quality audit"
```

### Quality_Scorecard

```yaml
plugin:
  name: "Quality_Scorecard"
  type: "reporting"
  version: "1.0.0"
  description: "Multi-dimensional quality scorecard generator with configurable dimensions, weights, and audience-appropriate formatting. Outputs PDF, Excel, JSON, and interactive dashboard formats. Supports GAI-OBSERVE branded reporting with theme-factory integration."
  installation: "pip install jinja2>=3.1.0 reportlab>=4.0.0 openpyxl>=3.1.0 theme-factory>=1.0.0"
  config:
    output_formats: ["pdf", "xlsx", "json", "html"]
    dimensions: ["coverage", "test_reliability", "defect_density", "process_capability", "review_velocity", "review_thoroughness", "dpmo", "sigma_level"]
    weight_presets:
      executive: { coverage: 0.15, test_reliability: 0.15, defect_density: 0.15, process_capability: 0.15, review_velocity: 0.10, review_thoroughness: 0.10, dpmo: 0.10, sigma_level: 0.10 }
      technical: { coverage: 0.20, test_reliability: 0.20, defect_density: 0.10, process_capability: 0.15, review_velocity: 0.10, review_thoroughness: 0.10, dpmo: 0.05, sigma_level: 0.10 }
    grade_scale: { A: 90, B: 80, C: 70, D: 60, F: 0 }
  auth:
    method: "none (local library)"
  health_check:
    import: "import jinja2; print(jinja2.__version__)"
    expected_output: "contains '3.1'"
  quotas:
    max_scorecards_per_batch: 50
    max_dimensions_per_scorecard: 20
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "Quality scorecard generation for Six Sigma assessments and quality audits"
```

---

## 7. DPMO, Review & Optimization Plugins

### DPMO_Calculator

```yaml
plugin:
  name: "DPMO_Calculator"
  type: "dpmo_engine"
  version: "1.0.0"
  description: "DPMO and sigma level translation engine with confidence intervals, 1.5σ shift adjustment, and capability level classification. Validates DPMO-to-sigma translation against standard Six Sigma tables."
  installation: "pip install numpy>=1.26.0 scipy>=1.12.0"
  config:
    include_1_5_sigma_shift: true
    confidence_level: 0.95
    standard_sigma_table: "/opt/d1/fixtures/sigma_table.json"
    validation_mode: "strict"
  auth:
    method: "none (local library)"
  health_check:
    import: "import numpy; print(numpy.__version__)"
    expected_output: "contains '1.26'"
  quotas:
    max_computations_per_batch: 100000
  security:
    data_locality: "true"
    deterministic_core: "true"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "DPMO computation and sigma level translation for all quality assessments"
```

### Review_Auditor

```yaml
plugin:
  name: "Review_Auditor"
  type: "review_integration"
  version: "1.0.0"
  description: "Git platform PR/MR data ingestion and review metrics computation. Supports GitHub, GitLab, and Bitbucket with review velocity, thoroughness, reviewer distribution, and defect escape correlation analysis."
  installation: "pip install pygithub>=2.0.0 gitlab-python>=4.0.0 bitbucket-api>=1.0.0 pandas>=2.2.0"
  config:
    platforms: ["github", "gitlab", "bitbucket"]
    metrics: ["velocity", "thoroughness", "distribution", "correlation"]
    reviewer_anonymization: true
    gini_coefficient: true
  auth:
    method: "OAuth 2.0 + JWT (app layer)"
    token_source: "Vault"
  health_check:
    command: "python -c 'import github; print(github.__version__)'"
    expected_output: "success"
  quotas:
    max_repos_per_scan: 50
    max_prs_per_repo: 1000
  security:
    data_locality: "true"
    pii_handling: "reviewer_anonymization"
  arm_integration:
    arms: ["ARM-04"]
    usage: "Code review practice assessment for quality audits"
```

### Test_Optimizer

```yaml
plugin:
  name: "Test_Optimizer"
  type: "optimization"
  version: "1.0.0"
  description: "Test suite optimizer using genetic algorithm (DEAP) and greedy selection to maximize coverage while minimizing execution time. Identifies redundant tests, reorders for fast failure, and validates mutation score preservation."
  installation: "pip install deap>=1.4.0 numpy>=1.26.0 pandas>=2.2.0"
  config:
    algorithm: "NSGA2"
    objectives: ["maximize_coverage", "minimize_execution_time", "preserve_mutation_score"]
    population_size: 100
    generations: 50
    crossover_prob: 0.7
    mutation_prob: 0.2
  auth:
    method: "none (local library)"
  health_check:
    import: "import deap; print(deap.__version__)"
    expected_output: "contains '1.4'"
  quotas:
    max_tests_per_optimization: 5000
    max_optimization_time_seconds: 600
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-03"]
    usage: "Test suite coverage optimization for test gap analysis"
```

---

## 8. Tracking & Planning Plugins

### Sigma_Tracker

```yaml
plugin:
  name: "Sigma_Tracker"
  type: "tracking_engine"
  version: "1.0.0"
  description: "Historical sigma level tracking with trend projection using ARIMA, exponential smoothing, and linear regression. Forecasts sigma level attainment dates and detects regression warnings."
  installation: "pip install numpy>=1.26.0 scipy>=1.12.0 statsmodels>=0.14.0"
  config:
    projection_methods: ["arima", "exponential_smoothing", "linear_regression"]
    default_method: "arima"
    forecast_horizon_default: 90
    alert_on_regression: true
    regression_threshold: 0.5
  auth:
    method: "none (local library)"
  health_check:
    import: "import statsmodels; print(statsmodels.__version__)"
    expected_output: "contains '0.14'"
  quotas:
    max_historical_points: 10000
    max_forecast_horizon: 365
  security:
    data_locality: "true"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "Sigma level tracking and trend projection for quality assessments and audits"
```

### Improvement_Planner

```yaml
plugin:
  name: "Improvement_Planner"
  type: "planning_engine"
  version: "1.0.0"
  description: "Improvement plan generator with effort estimation, ROI computation, and phased roadmap generation. Uses Pareto optimization to select highest-impact, lowest-effort improvements. Integrates with D3 Delivery Captain for scheduling."
  installation: "pip install numpy>=1.26.0 pandas>=2.2.0 jinja2>=3.1.0"
  config:
    estimation_method: "historical_median_with_buffer"
    buffer_multiplier: 1.3
    min_roi: 1.5
    max_phases: 5
    gantt_enabled: true
    control_plan_template: "/opt/d1/templates/control_plan.md.j2"
  auth:
    method: "none (local library) + LLM (narrative generation)"
    llm: "ollama/llama3:8b"
  health_check:
    import: "import jinja2; print(jinja2.__version__)"
    expected_output: "contains '3.1'"
  quotas:
    max_improvements_per_plan: 50
    max_effort_days_total: 365
  security:
    data_locality: "true"
    llm_output_filter: "enabled"
  arm_integration:
    arms: ["ARM-01", "ARM-04"]
    usage: "Improvement plan generation for Six Sigma and quality audit findings"
```

---

## 9. Infrastructure Plugins (Standard)

### PostgreSQL_15

```yaml
plugin:
  name: "PostgreSQL"
  type: "database"
  version: "15.6"
  description: "Primary structured data store for D1 reports, metadata, quality scorecards, audit records, and long-term memory."
  installation: "docker run postgres:15.6"
  config:
    port: 5433
    database: "gai_observe_d1"
    extensions: ["pgvector", "uuid-ossp", "jsonb"]
    max_connections: 200
    shared_buffers: "2GB"
  auth:
    method: "SCRAM-SHA-256 + JWT (app layer)"
    connection_pooler: "PgBouncer"
  health_check:
    endpoint: "SELECT 1"
    expected_response: "1"
  quotas:
    max_query_time: "30s"
    max_connections_per_user: 50
  security:
    tls: true
    encryption_at_rest: "AES-256"
    backup: "daily to S3"
  arm_integration:
    arms: ["All"]
    usage: "Structured storage for all arm reports, metadata, and memory layers"
```

### Redis

```yaml
plugin:
  name: "Redis"
  type: "cache"
  version: "7.2"
  description: "Short-term memory cache, active session store, metric alert buffer, and SPC real-time data stream for D1."
  installation: "docker run redis:7.2"
  config:
    port: 6380
    maxmemory: "4GB"
    maxmemory_policy: "allkeys-lru"
    persistence: "RDB + AOF"
  auth:
    method: "ACL + Redis AUTH password"
    password_source: "Vault"
  health_check:
    command: "PING"
    expected_response: "PONG"
  quotas:
    max_key_size: "512 MB"
    max_value_size: "512 MB"
    max_clients: 10000
  security:
    tls: true
    encryption: "TLS 1.3"
  arm_integration:
    arms: ["All"]
    usage: "Active monitoring session cache, metric alert buffer, SPC data stream"
```

---

## 10. Plugin-to-Arm Summary Matrix

| Plugin | Type | ARM-01 | ARM-02 | ARM-03 | ARM-04 | Priority |
|--------|------|--------|--------|--------|--------|----------|
| DMAIC_Framework | methodology_engine | ✅ | ❌ | ❌ | ✅ | P0 |
| DevAgent | quality_integration | ✅ | ❌ | ✅ | ✅ | P0 |
| ReproSense | reproducibility_engine | ❌ | ❌ | ✅ | ✅ | P0 |
| SPC_Tool | statistical_engine | ✅ | ✅ | ❌ | ❌ | P0 |
| Control_Chart | visualization | ✅ | ✅ | ❌ | ❌ | P0 |
| Western_Electric | rule_engine | ❌ | ✅ | ❌ | ❌ | P0 |
| Coverage_Analyzer | coverage_engine | ❌ | ❌ | ✅ | ❌ | P0 |
| Flaky_Test_Detector | flaky_engine | ❌ | ❌ | ✅ | ❌ | P0 |
| Defect_Tracker | defect_integration | ✅ | ❌ | ❌ | ✅ | P0 |
| Quality_Scorecard | reporting | ✅ | ❌ | ❌ | ✅ | P0 |
| DPMO_Calculator | dpmo_engine | ✅ | ❌ | ❌ | ✅ | P0 |
| Review_Auditor | review_integration | ❌ | ❌ | ❌ | ✅ | P0 |
| Test_Optimizer | optimization | ❌ | ❌ | ✅ | ❌ | P1 |
| Sigma_Tracker | tracking_engine | ✅ | ❌ | ❌ | ✅ | P1 |
| Improvement_Planner | planning_engine | ✅ | ❌ | ❌ | ✅ | P1 |
| PostgreSQL | database | ✅ | ✅ | ✅ | ✅ | P0 |
| Redis | cache | ✅ | ✅ | ✅ | ✅ | P0 |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team
**Next Review:** 2026-07-28
**Classification:** Internal — Plugin Registry
