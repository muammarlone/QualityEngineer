# HOOK_CONTRACTS.md
## Persona D1 — The Quality Engineer | Hook Contracts

**Version:** 1.0.0  
**Status:** Production-ready  
**Date:** 2026-06-28  
**Owner:** D1 The Quality Engineer  
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`  
**Master Strategy:** `C:\KimiWork Projects\GAI-OBSERVE-DESIGN\skills-hooks-plugins-strategy\STRATEGY.md`

---

## 1. Hook Registry Overview

D1 exposes 5 hook contracts for cross-persona integration. Each contract defines the full trigger condition, data transformation schema, failure handling, and compliance requirements. All hooks follow the GAI-OBSERVE hook contract template from `STRATEGY.md` Section 7.3.

---

## 2. Contract: d1_to_d9_quality_v1

```yaml
hook:
  id: "d1_to_d9_quality_v1"
  name: "D1 Quality Issues → D9 Forward Engineer Code Fix"
  type: "persona_invocation"
  description: "When D1 identifies quality issues that require code fixes (e.g., race conditions causing flaky tests, complexity violations, missing error handling), trigger D9 for code generation and remediation."
  version: "1.0.0"
  status: "active"
  owner: "D1 The Quality Engineer"

  trigger:
    event: "quality_issue_code_fix_required"
    source: "D1 ARM-01 (Six Sigma Assessor — Improve phase) or ARM-03 (Test Gap Analyzer) or ARM-04 (Quality Auditor)"
    filter:
      conditions:
        - "quality_issue_type in ['race_condition', 'complexity_breach', 'missing_tests', 'security_gap', 'performance_bottleneck']"
        - "severity in ['HIGH', 'CRITICAL']"
        - "effort_estimate_days <= 14"
      match_operator: "OR (first condition) AND (second or third)"
    debounce:
      throttle: "1 per hour per target"
      deduplication_window: "3600s"

  participants:
    - id: "D1"
      role: "producer"
      type: "persona"
      required: true
    - id: "D9"
      role: "consumer"
      type: "persona"
      required: true
    - id: "D2"
      role: "validator"
      type: "persona"
      required: false
    - id: "P2"
      role: "ledger"
      type: "persona"
      required: true

  data:
    input_schema:
      name: "D1QualityIssuePayload"
      type: "object"
      required_fields:
        - issue_id
        - target_id
        - target_type
        - issue_type
        - severity
        - evidence
        - module_ids
        - fix_recommendations
        - priority
        - timestamp
      properties:
        issue_id:
          type: "string"
          format: "UUID"
        target_id:
          type: "string"
        target_type:
          type: "enum"
          values: ["repository", "process", "pipeline", "module", "system"]
        issue_type:
          type: "enum"
          values: ["race_condition", "complexity_breach", "missing_tests", "security_gap", "performance_bottleneck", "refactoring_needed"]
        severity:
          type: "enum"
          values: ["LOW", "MEDIUM", "HIGH", "CRITICAL"]
        evidence:
          type: "object"
          properties:
            dmaic_phase: "string"
            sigma_level: "float"
            cpk: "float"
            coverage_gap: "float"
            defect_density: "float"
            module_analysis: "array[object]"
            control_chart_uri: "string"
        module_ids:
          type: "array"
          items:
            type: "string"
        fix_recommendations:
          type: "array"
          items:
            type: "object"
            properties:
              recommendation: "string"
              module: "string"
              effort_days: "float"
              expected_impact: "object"
        priority:
          type: "enum"
          values: ["P0", "P1", "P2", "P3"]
        timestamp:
          type: "string"
          format: "ISO-8601"

    output_schema:
      name: "D9CodeFixResponse"
      type: "object"
      required_fields:
        - fix_id
        - fixed_modules
        - code_changes
        - test_scaffold
        - security_scan_status
        - verification_results
        - ledger_hash
      properties:
        fix_id:
          type: "string"
          format: "UUID"
        fixed_modules:
          type: "array"
          items:
            type: "string"
        code_changes:
          type: "array"
          items:
            type: "object"
            properties:
              module: "string"
              diff: "string"
              lines_changed: "int"
        test_scaffold:
          type: "array"
          items:
            type: "string"
        security_scan_status:
          type: "enum"
          values: ["PASS", "WARN", "FAIL"]
        verification_results:
          type: "object"
          properties:
            tests_pass: "boolean"
            coverage_change: "float"
            complexity_change: "float"
        ledger_hash:
          type: "string"

    transform: |
      D1 quality issue → D9 code fix pipeline
      1. D1 completes quality assessment and identifies code-level issues
      2. D1 emits quality issue payload with evidence and recommendations
      3. D9 receives payload and routes to appropriate generation arm
      4. D9 generates fixed code with tests and documentation
      5. D2 optionally validates security of generated code
      6. D9 returns fixed code with verification results
      7. D1 re-assesses quality metrics and updates sigma level
      8. P2 records full transaction to immutable ledger

  quality:
    timeout_ms: 3600000
    retry:
      policy: "exponential_backoff"
      max_attempts: 3
      base_ms: 5000
    circuit_breaker:
      threshold: 5
      recovery_timeout: 600
      fallback: "return_manual_fix_guide_and_alert_d3"

  compliance:
    audit_level: "full_payload"
    required_signatures: ["D1", "D9", "P2"]
    pii_handling: "redact"
    data_classification: "INTERNAL"

  error_handling:
    on_timeout: "return manual fix guide with D1 recommendations and alert D3"
    on_d9_unavailable: "queue to D9 backlog and alert D3 (Delivery Captain)"
    on_d2_rejection: "escalate to G1 (Arbiter) for binding decision on security fix"
    on_p2_failure: "retry async; D1 retains local audit log"
```

---

## 3. Contract: d1_to_d7_test_v1

```yaml
hook:
  id: "d1_to_d7_test_v1"
  name: "D1 Test Gaps / Instability → D7 Test Automator Implementation"
  type: "persona_invocation"
  description: "When D1 identifies test gaps, flaky tests, or process instability requiring test automation improvements, trigger D7 for test implementation, stabilization, and optimization."
  version: "1.0.0"
  status: "active"
  owner: "D1 The Quality Engineer"

  trigger:
    event: "test_gap_or_instability_detected"
    source: "D1 ARM-02 (SPC Controller) or ARM-03 (Test Gap Analyzer)"
    filter:
      conditions:
        - "coverage_gap_percent > 0"
        - "flaky_test_count > 0"
        - "spc_stability_status in ['UNSTABLE', 'TRENDING']"
        - "test_reliability < 0.95"
      match_operator: "OR"
    debounce:
      throttle: "1 per day per repository"
      deduplication_window: "86400s"

  participants:
    - id: "D1"
      role: "producer"
      type: "persona"
      required: true
    - id: "D7"
      role: "consumer"
      type: "persona"
      required: true
    - id: "D9"
      role: "collaborator"
      type: "persona"
      required: false
    - id: "P2"
      role: "ledger"
      type: "persona"
      required: true

  data:
    input_schema:
      name: "D1TestGapPayload"
      type: "object"
      required_fields:
        - gap_id
        - repository_id
        - branch
        - gap_type
        - severity
        - gap_details
        - test_plan
        - priority
        - timestamp
      properties:
        gap_id:
          type: "string"
          format: "UUID"
        repository_id:
          type: "string"
        branch:
          type: "string"
        gap_type:
          type: "enum"
          values: ["coverage_gap", "flaky_test", "missing_tests", "unstable_process", "test_optimization"]
        severity:
          type: "enum"
          values: ["LOW", "MEDIUM", "HIGH", "CRITICAL"]
        gap_details:
          type: "object"
          properties:
            current_coverage: "float"
            target_coverage: "float"
            gap_modules: "array[object]"
            flaky_tests: "array[object]"
            rule_violations: "array[object]"
            test_reliability: "float"
        test_plan:
          type: "object"
          properties:
            total_tests_recommended: "int"
            total_effort_days: "float"
            expected_coverage_after: "float"
            phases: "array[string]"
        priority:
          type: "enum"
          values: ["P0", "P1", "P2", "P3"]
        timestamp:
          type: "string"
          format: "ISO-8601"

    output_schema:
      name: "D7TestImplementationResponse"
      type: "object"
      required_fields:
        - implementation_id
        - tests_implemented
        - tests_stabilized
        - coverage_after
        - test_reliability_after
        - verification_results
        - ledger_hash
      properties:
        implementation_id:
          type: "string"
          format: "UUID"
        tests_implemented:
          type: "array"
          items:
            type: "object"
            properties:
              module: "string"
              test_name: "string"
              lines_covered: "int"
        tests_stabilized:
          type: "array"
          items:
            type: "string"
        coverage_after:
          type: "float"
        test_reliability_after:
          type: "float"
        verification_results:
          type: "object"
          properties:
            all_tests_pass: "boolean"
            no_flaky_tests: "boolean"
            mutation_score: "float"
        ledger_hash:
          type: "string"

    transform: |
      D1 test gap / instability → D7 test automation pipeline
      1. D1 completes gap analysis or SPC evaluation and identifies test needs
      2. D1 emits test gap payload with prioritized test plan
      3. D7 receives payload and routes to appropriate test generation arm
      4. D7 implements missing tests, stabilizes flaky tests, or optimizes suite
      5. D9 optionally generates test scaffolding for complex modules
      6. D7 runs full test suite and verifies coverage improvement
      7. D1 validates coverage and reliability improvement
      8. P2 records full transaction to immutable ledger

  quality:
    timeout_ms: 1800000
    retry:
      policy: "exponential_backoff"
      max_attempts: 3
      base_ms: 3000
    circuit_breaker:
      threshold: 5
      recovery_timeout: 600
      fallback: "return_manual_test_plan_and_alert_d3"

  compliance:
    audit_level: "full_payload"
    required_signatures: ["D1", "D7", "P2"]
    pii_handling: "redact"
    data_classification: "INTERNAL"

  error_handling:
    on_timeout: "return manual test plan with D1 recommendations and alert D3"
    on_d7_unavailable: "queue to D7 backlog and alert D3 (Delivery Captain)"
    on_d9_rejection: "proceed with D7-only implementation and note scaffolding gap"
    on_p2_failure: "retry async; D1 retains local audit log"
```

---

## 4. Contract: d1_to_g1_certification_v1

```yaml
hook:
  id: "d1_to_g1_certification_v1"
  name: "D1 Quality Audit → G1 Arbiter Quality Certification"
  type: "persona_invocation"
  description: "When D1 completes a quality audit or Six Sigma assessment, trigger G1 The Arbiter for quality certification, risk classification, and binding sign-off. Required for release readiness and compliance."
  version: "1.0.0"
  status: "active"
  owner: "D1 The Quality Engineer"

  trigger:
    event: "quality_audit_completed"
    source: "D1 ARM-01 (Six Sigma Assessor) or ARM-04 (Quality Auditor)"
    filter:
      conditions:
        - "certification_requested == true"
        - "overall_grade in ['A', 'B', 'C']"
        - "critical_gaps == 0"
      match_operator: "AND (first and second) OR (first and third)"
    debounce:
      throttle: "1 per audit per target"
      deduplication_window: "86400s"

  participants:
    - id: "D1"
      role: "producer"
      type: "persona"
      required: true
    - id: "G1"
      role: "consumer"
      type: "persona"
      required: true
    - id: "P3"
      role: "verifier"
      type: "persona"
      required: false
    - id: "P2"
      role: "ledger"
      type: "persona"
      required: true

  data:
    input_schema:
      name: "D1QualityCertificationRequest"
      type: "object"
      required_fields:
        - audit_id
        - target_id
        - target_type
        - overall_score
        - overall_grade
        - dimension_scores
        - sigma_level
        - cpk
        - dpmo
        - compliance_mapping
        - priority_gaps
        - recommendations
        - timestamp
      properties:
        audit_id:
          type: "string"
          format: "UUID"
        target_id:
          type: "string"
        target_type:
          type: "enum"
          values: ["repository", "team", "project", "system", "process"]
        overall_score:
          type: "float"
          range: [0, 100]
        overall_grade:
          type: "enum"
          values: ["A", "B", "C", "D", "F"]
        dimension_scores:
          type: "object"
          properties:
            coverage: "float"
            test_reliability: "float"
            defect_density: "float"
            process_capability: "float"
            review_velocity: "float"
            review_thoroughness: "float"
            dpmo: "int"
            sigma_level: "float"
        sigma_level:
          type: "float"
        cpk:
          type: "float"
        dpmo:
          type: "int"
        compliance_mapping:
          type: "object"
          additionalProperties:
            type: "object"
            properties:
              framework: "string"
              requirement: "string"
              status: "enum [COMPLIANT, MARGINAL, NON_COMPLIANT, UNKNOWN]"
              gap: "string"
        priority_gaps:
          type: "array"
          items:
            type: "string"
        recommendations:
          type: "array"
          items:
            type: "string"
        timestamp:
          type: "string"
          format: "ISO-8601"

    output_schema:
      name: "G1QualityCertification"
      type: "object"
      required_fields:
        - certification_id
        - verdict
        - risk_classification
        - binding_signatures
        - conditions
        - expiry
        - ledger_hash
      properties:
        certification_id:
          type: "string"
          format: "UUID"
        verdict:
          type: "enum"
          values: ["CERTIFIED", "CONDITIONAL", "REJECTED", "PENDING_REVIEW"]
        risk_classification:
          type: "enum"
          values: ["LOW", "MEDIUM", "HIGH", "CRITICAL"]
        binding_signatures:
          type: "array"
          items:
            type: "object"
            properties:
              persona_id: "string"
              signature: "string"
              timestamp: "string"
        conditions:
          type: "array"
          items:
            type: "string"
        expiry:
          type: "string"
          format: "ISO-8601 date"
        ledger_hash:
          type: "string"

    transform: |
      D1 quality audit → G1 certification pipeline
      1. D1 completes quality audit with full evidence package
      2. G1 receives audit with dimension scores and compliance mapping
      3. G1 applies legal-risk-assessment skill
      4. G1 classifies risk by severity × likelihood
      5. G1 determines verdict (CERTIFIED / CONDITIONAL / REJECTED)
      6. P3 optionally verifies statistical claims
      7. G1 generates binding signatures
      8. P2 records certification to immutable ledger
      9. If REJECTED, trigger D3 for remediation scheduling
      10. If CONDITIONAL, D1 monitors compliance with conditions

  quality:
    timeout_ms: 600000
    retry:
      policy: "exponential_backoff"
      max_attempts: 3
      base_ms: 5000
    circuit_breaker:
      threshold: 3
      recovery_timeout: 600
      fallback: "return 'PENDING_REVIEW' status and queue for manual G1 review"

  compliance:
    audit_level: "full_payload"
    required_signatures: ["D1", "G1", "P2"]
    pii_handling: "aggregate_only"
    data_classification: "RESTRICTED"

  error_handling:
    on_timeout: "return 'PENDING_REVIEW' with 7-day manual review deadline"
    on_g1_unavailable: "queue to G1 backlog and alert D3"
    on_p3_rejection: "return audit to D1 for correction with specific claim issues"
    on_p2_failure: "retry async; G1 retains local certification draft"
```

---

## 5. Contract: d1_to_d3_planning_v1

```yaml
hook:
  id: "d1_to_d3_planning_v1"
  name: "D1 Improvement Roadmap → D3 Delivery Captain Scheduling"
  type: "persona_invocation"
  description: "When D1 generates an improvement plan or roadmap, trigger D3 Delivery Captain for resource allocation, scheduling, and dependency management. Ensures quality initiatives are tracked and delivered."
  version: "1.0.0"
  status: "active"
  owner: "D1 The Quality Engineer"

  trigger:
    event: "improvement_plan_generated"
    source: "D1 ARM-01 (Six Sigma Assessor — Improve phase) or ARM-04 (Quality Auditor)"
    filter:
      conditions:
        - "total_effort_days > 0"
        - "improvement_count > 0"
        - "priority in ['P0', 'P1']"
      match_operator: "AND"
    debounce:
      throttle: "1 per plan per target"
      deduplication_window: "86400s"

  participants:
    - id: "D1"
      role: "producer"
      type: "persona"
      required: true
    - id: "D3"
      role: "consumer"
      type: "persona"
      required: true
    - id: "G1"
      role: "approver"
      type: "persona"
      required: false
    - id: "P2"
      role: "ledger"
      type: "persona"
      required: true

  data:
    input_schema:
      name: "D1ImprovementPlanPayload"
      type: "object"
      required_fields:
        - plan_id
        - target_id
        - target_type
        - improvements
        - total_effort_days
        - expected_impact
        - roi
        - phased_roadmap
        - dependencies
        - priority
        - timestamp
      properties:
        plan_id:
          type: "string"
          format: "UUID"
        target_id:
          type: "string"
        target_type:
          type: "enum"
          values: ["repository", "team", "project", "system", "process"]
        improvements:
          type: "array"
          items:
            type: "object"
            properties:
              item_id: "string"
              description: "string"
              effort_days: "float"
              expected_impact: "object"
              priority: "enum [P0, P1, P2, P3]"
              dependencies: "array[string]"
        total_effort_days:
          type: "float"
        expected_impact:
          type: "object"
          properties:
            sigma_level_improvement: "float"
            coverage_improvement: "float"
            reliability_improvement: "float"
            defect_reduction_percent: "float"
        roi:
          type: "float"
        phased_roadmap:
          type: "object"
          properties:
            phases: "array[object]"
            milestones: "array[object]"
            start_date: "string"
            end_date: "string"
        dependencies:
          type: "array"
          items:
            type: "string"
        priority:
          type: "enum"
          values: ["P0", "P1", "P2", "P3"]
        timestamp:
          type: "string"
          format: "ISO-8601"

    output_schema:
      name: "D3DeliveryPlanResponse"
      type: "object"
      required_fields:
        - delivery_plan_id
        - scheduled_improvements
        - resource_allocation
        - timeline
        - status
        - ledger_hash
      properties:
        delivery_plan_id:
          type: "string"
          format: "UUID"
        scheduled_improvements:
          type: "array"
          items:
            type: "object"
            properties:
              item_id: "string"
              assigned_team: "string"
              start_date: "string"
              end_date: "string"
              status: "enum [SCHEDULED, IN_PROGRESS, COMPLETE, BLOCKED]"
        resource_allocation:
          type: "object"
          properties:
            total_person_days: "float"
            teams: "array[string]"
            capacity_utilization: "float"
        timeline:
          type: "object"
          properties:
            start_date: "string"
            end_date: "string"
            critical_path: "array[string]"
        status:
          type: "enum"
          values: ["APPROVED", "PENDING", "REJECTED", "CONDITIONAL"]
        ledger_hash:
          type: "string"

    transform: |
      D1 improvement plan → D3 delivery scheduling pipeline
      1. D1 completes quality analysis and generates improvement plan
      2. D1 emits improvement plan with effort estimates and ROI
      3. D3 receives plan and analyzes resource availability
      4. D3 schedules improvements with team assignments and milestones
      5. G1 optionally approves plan if compliance-related
      6. D3 returns scheduled delivery plan with critical path
      7. D1 monitors quality metrics during implementation
      8. P2 records plan and schedule to immutable ledger
      9. On completion, D1 re-assesses and validates improvement

  quality:
    timeout_ms: 600000
    retry:
      policy: "exponential_backoff"
      max_attempts: 3
      base_ms: 3000
    circuit_breaker:
      threshold: 5
      recovery_timeout: 300
      fallback: "return unscheduled plan with dependency graph and alert_d3_manual"

  compliance:
    audit_level: "full_payload"
    required_signatures: ["D1", "D3", "P2"]
    pii_handling: "redact"
    data_classification: "INTERNAL"

  error_handling:
    on_timeout: "return unscheduled plan with manual scheduling instructions"
    on_d3_unavailable: "queue to D3 backlog and alert D3 via alternate channel"
    on_g1_rejection: "return plan to D1 for revision with G1 conditions"
    on_p2_failure: "retry async; D3 retains local schedule draft"
```

---

## 6. Contract: d1_to_g6_monitoring_v1

```yaml
hook:
  id: "d1_to_g6_monitoring_v1"
  name: "D1 Control Plan → G6 Sentinel Continuous Monitoring"
  type: "persona_invocation"
  description: "When D1 completes a control plan or identifies metrics requiring continuous monitoring, trigger G6 The Sentinel for monitoring deployment, alert configuration, and dashboard setup. Ensures quality metrics are tracked in real-time."
  version: "1.0.0"
  status: "active"
  owner: "D1 The Quality Engineer"

  trigger:
    event: "control_plan_deployed OR monitoring_request"
    source: "D1 ARM-01 (Six Sigma Assessor — Control phase) or ARM-02 (SPC Controller)"
    filter:
      conditions:
        - "control_plan_status == 'complete'"
        - "monitoring_required == true"
        - "metrics_count > 0"
      match_operator: "AND"
    debounce:
      throttle: "1 per control plan per target"
      deduplication_window: "86400s"

  participants:
    - id: "D1"
      role: "producer"
      type: "persona"
      required: true
    - id: "G6"
      role: "consumer"
      type: "persona"
      required: true
    - id: "D5"
      role: "observer"
      type: "persona"
      required: false
    - id: "P2"
      role: "ledger"
      type: "persona"
      required: true

  data:
    input_schema:
      name: "D1MonitoringConfigPayload"
      type: "object"
      required_fields:
        - config_id
        - target_id
        - target_type
        - control_plan
        - metrics
        - thresholds
        - alert_rules
        - dashboard_config
        - timestamp
      properties:
        config_id:
          type: "string"
          format: "UUID"
        target_id:
          type: "string"
        target_type:
          type: "enum"
          values: ["repository", "team", "project", "system", "process"]
        control_plan:
          type: "object"
          properties:
            control_plan_id: "string"
            control_items: "array[object]"
            response_plan: "string"
            owner: "string"
        metrics:
          type: "array"
          items:
            type: "object"
            properties:
              metric_name: "enum [coverage, test_reliability, defect_density, build_time, sigma_level, cpk, dpmo, review_velocity]"
              metric_type: "enum [gauge, counter, histogram]"
              collection_interval: "string"
              data_source: "string"
        thresholds:
          type: "array"
          items:
            type: "object"
            properties:
              metric_name: "string"
              warning_value: "float"
              critical_value: "float"
              direction: "enum [above, below]"
        alert_rules:
          type: "array"
          items:
            type: "object"
            properties:
              rule_name: "string"
              condition: "string"
              severity: "enum [INFO, LOW, MEDIUM, HIGH, CRITICAL]"
              recipients: "array[string]"
              channels: "array[enum]"
        dashboard_config:
          type: "object"
          properties:
            dashboard_name: "string"
            panels: "array[object]"
            refresh_interval: "string"
            theme: "enum [gai-observe-dark, gai-observe-light]"
        timestamp:
          type: "string"
          format: "ISO-8601"

    output_schema:
      name: "G6MonitoringDeploymentResponse"
      type: "object"
      required_fields:
        - deployment_id
        - status
        - deployed_metrics
        - alert_endpoints
        - dashboard_uri
        - ledger_hash
      properties:
        deployment_id:
          type: "string"
          format: "UUID"
        status:
          type: "enum"
          values: ["DEPLOYED", "PARTIAL", "FAILED", "PENDING"]
        deployed_metrics:
          type: "array"
          items:
            type: "object"
            properties:
              metric_name: "string"
              endpoint: "string"
              status: "enum [ACTIVE, INACTIVE, ERROR]"
        alert_endpoints:
          type: "array"
          items:
            type: "string"
        dashboard_uri:
          type: "string"
        ledger_hash:
          type: "string"

    transform: |
      D1 control plan → G6 monitoring deployment pipeline
      1. D1 completes control plan with monitoring requirements
      2. D1 emits monitoring configuration with metrics, thresholds, and alert rules
      3. G6 receives configuration and validates metric endpoints
      4. G6 deploys Prometheus exporters, Grafana dashboards, and alertmanager rules
      5. D5 optionally subscribes to HIGH/CRITICAL alerts for SRE escalation
      6. G6 returns deployment status with dashboard URI and alert endpoints
      7. D1 receives confirmation and begins monitoring control plan compliance
      8. P2 records deployment to immutable ledger
      9. On threshold breach, G6 alerts D1 and D5 per configured rules

  quality:
    timeout_ms: 300000
    retry:
      policy: "exponential_backoff"
      max_attempts: 3
      base_ms: 2000
    circuit_breaker:
      threshold: 5
      recovery_timeout: 300
      fallback: "return manual_monitoring_guide_and_alert_d5"

  compliance:
    audit_level: "metadata"
    required_signatures: ["D1", "G6", "P2"]
    pii_handling: "aggregate_only"
    data_classification: "INTERNAL"

  error_handling:
    on_timeout: "return manual monitoring setup guide and alert D5"
    on_g6_unavailable: "queue to G6 backlog and alert D5 for manual monitoring"
    on_d5_rejection: "proceed with G6-only monitoring and note SRE gap"
    on_p2_failure: "retry async; G6 retains local deployment config"
```

---

## 7. Hook Summary Matrix

| Hook ID | Trigger | Producer | Consumer | Validator | Ledger | Priority | Data Classification |
|---------|---------|----------|----------|-----------|--------|----------|---------------------|
| `d1_to_d9_quality_v1` | Quality issue code fix | D1 | D9 | D2 (opt) | P2 | P0 | INTERNAL |
| `d1_to_d7_test_v1` | Test gap / instability | D1 | D7 | D9 (opt) | P2 | P0 | INTERNAL |
| `d1_to_g1_certification_v1` | Quality audit complete | D1 | G1 | P3 (opt) | P2 | P0 | RESTRICTED |
| `d1_to_d3_planning_v1` | Improvement plan generated | D1 | D3 | G1 (opt) | P2 | P1 | INTERNAL |
| `d1_to_g6_monitoring_v1` | Control plan / monitoring | D1 | G6 | D5 (opt) | P2 | P1 | INTERNAL |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team  
**Next Review:** 2026-07-28  
**Classification:** Internal — Hook Contracts
