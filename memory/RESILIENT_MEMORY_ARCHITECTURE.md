# RESILIENT_MEMORY_ARCHITECTURE.md
## Persona D1 — The Quality Engineer | 3-Layer Resilient Memory

**Version:** 1.0.0
**Status:** Production-ready
**Date:** 2026-06-28
**Owner:** D1 The Quality Engineer
**Parent:** `AGENTIC_ARMS_OVERVIEW.md`
**Master Strategy:** `C:\KimiWork Projects\GAI-OBSERVE-DESIGN\skills-hooks-plugins-strategy\STRATEGY.md`
**Backend Standards:** `C:\KimiWork Projects\GAI-OBSERVE-DESIGN\architecture.md`

---

## 1. Executive Summary

D1 requires a **resilient memory architecture** capable of storing high-velocity quality metrics, long-term Six Sigma baselines, and episodic audit trails with full evidence-backed traceability. The architecture follows the GAI-OBSERVE standard three-layer pattern: Short-Term Memory (STM) for active quality assessments and SPC sessions, Long-Term Memory (LTM) for Six Sigma baselines and quality thresholds, and Episodic Memory (EM) for time-series audit trails and DMAIC session records.

---

## 2. Memory Architecture Overview

```mermaid
flowchart TB
    subgraph LAYER1["LAYER 1: Short-Term Memory (STM)"]
        A1["Redis Cache<br/>Active Assessments<br/>SPC Sessions<br/>Alert Buffers<br/>TTL: 24h active / 7d recent"]
        A2["pgvector<br/>128-dim Embeddings<br/>Semantic Quality Retrieval<br/>Turn-level"]
    end

    subgraph LAYER2["LAYER 2: Long-Term Memory (LTM)"]
        B1["PostgreSQL JSONB<br/>Six Sigma Baselines<br/>SPC Baselines<br/>Quality Thresholds<br/>Gap Maps<br/>Control Plans"]
        B2["Filesystem<br/>Report Archives<br/>Control Charts<br/>Scorecards<br/>Ledger References"]
    end

    subgraph LAYER3["LAYER 3: Episodic Memory (EM)"]
        C1["TimescaleDB<br/>Time-Series Quality Metrics<br/>DMAIC Sessions<br/>Audit Sessions<br/>SPC Evaluations<br/>Test Gap Analyses"]
    end

    LAYER1 -->|Sync (append-only, CRDT)| LAYER2
    LAYER2 -->|Archive (compression 10:1)| LAYER3

    style LAYER1 fill:#ffcccc
    style LAYER2 fill:#ffffcc
    style LAYER3 fill:#ccffcc
```

---

## 3. Layer 1: Short-Term Memory (STM)

### 3.1 Storage Backend

| Component | Technology | Purpose | TTL |
|-----------|------------|---------|-----|
| **Active Cache** | Redis 7.2 | Active quality assessments, SPC sessions, metric buffers, alert queues | 24 hours |
| **Recent Cache** | Redis 7.2 | Recently completed analyses, pending cross-arm chains, gap maps | 7 days |
| **Vector Store** | pgvector (PostgreSQL) | 128-dimensional embeddings for semantic quality retrieval | 7 days |

### 3.2 STM Schema

```json
{
  "stm_entry": {
    "turn_id": "uuid-v4",
    "timestamp": "2026-06-28T14:00:00Z",
    "persona_id": "D1",
    "arm_id": "ARM-01",
    "target_id": "pipeline-alpha",
    "target_type": "pipeline",
    "metric_name": "build_time",
    "metric_value": 14.3,
    "sigma_level": 2.3,
    "dpmo": 23000,
    "cpk": 0.85,
    "coverage": 0.71,
    "test_reliability": 0.83,
    "alert_status": "SIGNIFICANT",
    "embedding": [0.12, -0.05, 0.33, ...],
    "ttl": 86400
  }
}
```

### 3.3 Special STM Collections

| Collection | Purpose | Schema | TTL |
|------------|---------|--------|-----|
| **Active Assessment Session** | Currently running DMAIC or quality assessments | `{session_id, target_id, arm_id, dmaic_phase, phase_status, start_time, status, last_heartbeat}` | Session lifetime + 1h |
| **SPC Session Buffer** | Active control chart data and rule evaluation state | `{chart_id, target_id, metric_name, data_points, control_limits, last_evaluation, rule_violations}` | 24h |
| **Metric Alert Buffer** | Pending quality alerts awaiting dispatch or deduplication | `{alert_id, target_id, metric, severity, created_at, dispatched}` | 24h |
| **Cross-Arm Chain State** | Pending chained arm invocations (D1→D9, D1→D7, D1→G1, D1→D3, D1→G6) | `{chain_id, from_arm, to_persona, hook_contract, payload, status, retry_count}` | 1h |
| **Gap Map Cache** | Recently computed coverage gap maps for fast retrieval | `{analysis_id, repository_id, branch, gap_map, timestamp}` | 7 days |

### 3.4 STM Resilience

- **Redis Sentinel:** High availability with automatic failover
- **RDB + AOF:** Dual persistence for crash recovery
- **LRU Eviction:** `allkeys-lru` policy when memory limit reached
- **Replication:** Master-replica with async replication (eventual consistency acceptable for STM)

---

## 4. Layer 2: Long-Term Memory (LTM)

### 4.1 Storage Backend

| Component | Technology | Purpose | Retention |
|-----------|------------|---------|-----------|
| **Structured Baselines** | PostgreSQL 15 JSONB | Six Sigma baselines, SPC baselines, quality thresholds, gap maps, control plans | Indefinite (versioned) |
| **Report Archives** | Filesystem (S3/MinIO) | Generated reports, control charts, scorecards, audit reports, test plans | 7 years (regulatory) |
| **Ledger References** | PostgreSQL 15 + P2 | Immutable references to P2 ledger entries | Indefinite |

### 4.2 LTM Schema

```json
{
  "ltm_entry": {
    "fact_id": "uuid-v4",
    "category": "six_sigma_baseline",
    "key": "pipeline-alpha_dmaic_baseline_2026q2",
    "value": {
      "target_id": "pipeline-alpha",
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
}
```

### 4.3 LTM Categories

| Category | Description | Example Key | Update Frequency |
|----------|-------------|-------------|-----------------|
| **six_sigma_baseline** | DMAIC assessment baselines with sigma levels and capability indices | `pipeline-alpha_dmaic_baseline_2026q2` | Per assessment |
| **spc_baseline** | Control chart baselines with control limits and capability indices | `pipeline-alpha_build-time_spc_baseline` | Per control limit update |
| **quality_threshold** | Configurable quality thresholds per target and dimension | `threshold_coverage_pipeline-alpha` | Per threshold change |
| **test_gap_map** | Coverage gap maps with risk scores and test plans | `knowledgeworker_main_gap_map` | Per gap analysis |
| **control_plan** | Control plans with monitoring metrics and response plans | `cp-pipeline-alpha-001` | Per control plan creation |
| **improvement_roadmap** | Prioritized improvement plans with milestones | `roadmap-pipeline-alpha-001` | Per plan update |
| **review_practice_baseline** | Code review practice baselines with velocity and thoroughness | `gitsentinental_review_baseline_2026q2` | Per audit |
| **defect_pattern** | Recognized defect patterns and systemic defect signatures | `defect_pattern_auth_bypass_v1` | Per discovery |
| **flaky_test_signature** | Recognized flaky test patterns and root cause signatures | `flaky_pattern_race_condition_async` | Per discovery |
| **experiment_metadata** | Quality experiment references and results | `exp_coverage_optimization_2026-06` | Per experiment |

### 4.4 LTM Synchronization

- **Append-Only:** All LTM writes are append-only; updates create new versions
- **CRDT (Conflict-free Replicated Data Types):** For concurrent quality updates across distributed instances
- **Versioning:** Every fact has a version number; queries default to latest
- **Compaction:** Annual compaction of obsolete versions (retain last 3 versions per key)

### 4.5 LTM Sync Protocol

```yaml
ltm_sync:
  method: "append-only with vector clock"
  conflict_resolution: "last-writer-wins per field with CRDT merge"
  replication: "async to read replicas, sync for critical baseline updates"
  backup:
    schedule: "daily at 02:00 UTC"
    target: "S3 encrypted bucket"
    retention: "90 days incremental, 7 years full"
  integrity:
    checksum: "SHA-256 per JSONB row"
    verification: "weekly full scan"
```

---

## 5. Layer 3: Episodic Memory (EM)

### 5.1 Storage Backend

| Component | Technology | Purpose | Retention |
|-----------|------------|---------|-----------|
| **Time-Series Quality Metrics** | TimescaleDB 2.14 | Quality metrics, sigma levels, DPMO, capability indices, coverage trends, defect densities | 7 years |
| **DMAIC Sessions** | TimescaleDB 2.14 | Complete DMAIC assessment sessions with phase evidence and improvement outcomes | 7 years |
| **SPC Evaluations** | TimescaleDB 2.14 | Control chart evaluation sessions with rule violations and stability classifications | 7 years |
| **Audit Sessions** | TimescaleDB 2.14 | Quality audit sessions with scorecards and certification recommendations | 7 years |
| **Test Gap Analyses** | TimescaleDB 2.14 | Coverage gap analysis sessions with test plans and flaky test findings | 7 years |

### 5.2 EM Schema

```json
{
  "em_entry": {
    "session_id": "uuid-v4",
    "persona_id": "D1",
    "arm_id": "ARM-01",
    "target_id": "pipeline-alpha",
    "target_type": "pipeline",
    "start_time": "2026-06-28T13:00:00Z",
    "end_time": "2026-06-28T14:00:00Z",
    "metrics": {
      "sigma_level": 2.3,
      "dpmo": 23000,
      "cpk": 0.85,
      "coverage": 0.71,
      "test_reliability": 0.83,
      "defect_density": 0.18
    },
    "dmaic_phases": {
      "define": { "status": "complete", "evidence_count": 3 },
      "measure": { "status": "complete", "evidence_count": 8 },
      "analyze": { "status": "complete", "evidence_count": 6 },
      "improve": { "status": "complete", "evidence_count": 4 },
      "control": { "status": "complete", "evidence_count": 2 }
    },
    "spc_evaluations": [
      {
        "chart_id": "chart-build-time-alpha",
        "rule_violations": [{"rule": "WE_1", "point_index": 42}],
        "stability_status": "UNSTABLE"
      }
    ],
    "gap_findings": [
      {
        "module": "auth.py",
        "coverage": 0.34,
        "risk_score": 9.2
      }
    ],
    "audit_results": {
      "overall_score": 87,
      "overall_grade": "B+",
      "priority_gaps": ["test_reliability", "cpk"]
    },
    "improvement_outcomes": {
      "actions_completed": 5,
      "sigma_level_after": 4.1,
      "dpmo_after": 3400
    },
    "embedding": [0.15, -0.08, 0.22, ...],
    "compression_ratio": 0.10
  }
}
```

### 5.3 EM Hypertable Design

```sql
-- TimescaleDB hypertable for D1 episodic memory
CREATE TABLE d1_episodic_memory (
    session_id UUID PRIMARY KEY,
    persona_id TEXT NOT NULL,
    arm_id TEXT NOT NULL,
    target_id TEXT NOT NULL,
    target_type TEXT NOT NULL,
    start_time TIMESTAMPTZ NOT NULL,
    end_time TIMESTAMPTZ,
    metrics JSONB NOT NULL,
    dmaic_phases JSONB,
    spc_evaluations JSONB,
    gap_findings JSONB,
    audit_results JSONB,
    improvement_outcomes JSONB,
    embedding VECTOR(128),
    compression_ratio FLOAT
);

-- Convert to hypertable
SELECT create_hypertable('d1_episodic_memory', 'start_time', chunk_time_interval => INTERVAL '7 days');

-- Indexes for retrieval patterns
CREATE INDEX idx_em_target_time ON d1_episodic_memory (target_id, start_time DESC);
CREATE INDEX idx_em_arm ON d1_episodic_memory (arm_id, start_time DESC);
CREATE INDEX idx_em_metrics ON d1_episodic_memory USING GIN (metrics);
CREATE INDEX idx_em_embedding ON d1_episodic_memory USING ivfflat (embedding vector_cosine_ops);
```

### 5.4 EM Retrieval Patterns

| Retrieval Query | Filter | Time Range | Index Used |
|-----------------|--------|------------|------------|
| **Target History** | `target_id = ?` | Any | `idx_em_target_time` |
| **Arm Sessions** | `arm_id = ?` | Any | `idx_em_arm` |
| **Quality Alert Investigation** | `metrics->alert_status = 'CRITICAL'` | Last 30 days | `idx_em_metrics` |
| **Semantic Search** | `embedding <-> query_vector` | Any | `idx_em_embedding` |
| **DMAIC History** | `dmaic_phases IS NOT NULL` | Last 2 years | `idx_em_arm` + `start_time` |
| **SPC Violation History** | `spc_evaluations->violations != '[]'` | Last 90 days | `idx_em_metrics` |
| **Compliance Audit** | `audit_results IS NOT NULL` | Custom range | `idx_em_arm` + `start_time` |
| **Gap Analysis History** | `gap_findings IS NOT NULL` | Any | `idx_em_target_time` |

### 5.5 EM Compression

- **Automatic Compression:** Enabled after 30 days
- **Compression Ratio:** Target 10:1 (from ~10 KB to ~1 KB per session)
- **Decompression:** On-demand for audit queries; < 100ms latency
- **Cold Storage:** After 1 year, compress and archive to S3 Glacier

---

## 6. Memory Hooks & Resilience

### 6.1 Memory Hooks

```yaml
memory_hooks:
  - name: "six_sigma_baseline_tracking"
    trigger: "dmaic_assessment_completed"
    action: "Store sigma level, DPMO, Cpk in LTM with expiry = assessment_date + 1 year"
    target: "LTM (PostgreSQL)"

  - name: "spc_baseline_update"
    trigger: "control_limit_recomputed"
    action: "Update SPC baseline in LTM with new center line and control limits"
    target: "LTM (PostgreSQL)"

  - name: "gap_map_persistence"
    trigger: "test_gap_analysis_completed"
    action: "Store coverage gap map in LTM for cross-reference with future analyses"
    target: "LTM (PostgreSQL) + STM (Redis)"

  - name: "quality_trend_analysis"
    trigger: "audit_completed"
    action: "Append quality scorecard to time-series in EM; compare to previous audits; flag worsening trends"
    target: "EM (TimescaleDB)"

  - name: "defect_pattern_recognition"
    trigger: "defect_spike_detected"
    action: "Compare to known defect patterns in LTM; if match, annotate with pattern_id; if novel, create new pattern entry"
    target: "LTM (PostgreSQL) + EM (TimescaleDB)"

  - name: "flaky_test_signature_tracking"
    trigger: "flaky_test_detected"
    action: "Store flaky test signature in LTM; if root cause matches known pattern, link to signature"
    target: "LTM (PostgreSQL) + EM (TimescaleDB)"

  - name: "alert_buffer_flush"
    trigger: "alert_dispatch or alert_timeout"
    action: "Move from STM alert buffer to EM audit trail"
    target: "STM (Redis) → EM (TimescaleDB)"

  - name: "cross_arm_chain_completion"
    trigger: "hook_response_received"
    action: "Update chain state in STM; if all chains complete, archive to EM"
    target: "STM (Redis) → EM (TimescaleDB)"
```

### 6.2 Resilience Patterns

| Pattern | Implementation | RTO | RPO |
|---------|---------------|-----|-----|
| **STM Failover** | Redis Sentinel (3-node) | < 30s | < 1s |
| **LTM Replication** | PostgreSQL streaming replication | < 5 min | < 1 min |
| **EM Replication** | TimescaleDB multi-node | < 5 min | < 1 min |
| **Cross-Region Backup** | S3 cross-region replication | < 1 hour | < 1 hour |
| **Disaster Recovery** | Daily snapshot + WAL archiving | < 4 hours | < 1 hour |

### 6.3 Privacy & Compliance

```yaml
memory_privacy:
  reviewer_anonymization:
    method: "hash-based pseudonymization"
    fields: ["reviewer_name", "reviewer_email", "reviewer_id"]
    applied_at: "ingestion_time"

  defect_reporter_redaction:
    method: "tokenization with reversible mapping"
    fields: ["reporter_name", "reporter_email"]
    storage: "Vault-encrypted lookup table"
    access_control: "role_d1-audit_only"

  aggregated_quality_metrics:
    rule: "Never store individual-level code review or defect data with PII in EM"
    enforcement: "Schema validation + runtime check"

  gdpr_right_to_erasure:
    support: "Target-level deletion (cascade to all memory layers)"
    procedure: "1. Identify target_id → 2. Delete from STM → 3. Soft-delete LTM (retain hash) → 4. Archive EM with retention note → 5. Log to P2"
```

---

## 7. Memory Performance Budgets

| Layer | Write Latency | Read Latency | Throughput | Capacity |
|-------|--------------|--------------|------------|----------|
| **STM** | < 1 ms | < 5 ms | 100K ops/sec | 4 GB |
| **LTM** | < 50 ms | < 100 ms | 10K rows/sec | 500 GB |
| **EM** | < 100 ms | < 500 ms | 1K rows/sec | 10 TB |

---

**Document Owner:** GAI-OBSERVE Advisory Architecture Team
**Next Review:** 2026-07-28
**Classification:** Internal — Architecture
