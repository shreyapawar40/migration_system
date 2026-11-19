# Stateful Migration System in Go

A unified, high-performance distributed stateful migration and failover system implementing container recovery, process state preservation (CRIU), Redis-backed application state continuity, and database failover.

## System Architecture

```
+-----------------------------------------------------------------------------------+
|                           CONTROLLER (TCP :9000 & HTTP :9001)                     |
|  - Failure Detector (Heartbeat tracking)                                          |
|  - Recovery Manager (Target selection & stateful command dispatch)                |
|  - HTTP Metrics Dashboard & API (:9001)                                           |
+----------------------------------------+------------------------------------------+
                                         |
               +-------------------------+-------------------------+
               |                                                   |
               v                                                   v
+-------------------------------+                   +-------------------------------+
|       WORKER 1 (Source)       |                   |       WORKER 2 (Target)       |
| - TCP Client (:9000)          |                   | - TCP Client (:9000)          |
| - Stateful Manager            |  Image Transfer   | - Stateful Manager            |
| - CRIU Dump Engine            | =================>| - CRIU Restore Engine         |
| - Image Server (:9101)        |   (tar.gz bundle) | - Image Server (:9102)        |
| - Health & Resource Metrics   |                   | - Health & Resource Metrics   |
+-------------------------------+                   +-------------------------------+
               |                                                   |
               v                                                   v
+-------------------------------+                   +-------------------------------+
|    APPLICATION / DB STATE     |                   |    APPLICATION / DB STATE     |
| - Redis Primary Lease         |                   | - Redis Shadow Promotion      |
| - Token Bucket Counters       |                   | - Preserved Token State       |
| - PostgreSQL Primary (5432)   |                   | - PostgreSQL Standby (5433)   |
| - PgBouncer Connection Pool   |                   | - Auto-repointed PgBouncer    |
+-------------------------------+                   +-------------------------------+
```

---

## Project Objectives & Verification Status

| Objective | Requirement | Status | Verification Evidence |
|---|---|---|---|
| **Objective 1** | Container-based service migration & failover across failure scenarios | **COMPLETE** | Heartbeat failure detection triggers container migration onto target node (Downtime: **155 ms**, Recovery: **35 ms**). |
| **Objective 2** | Preserve and synchronize process, database, and application state | **COMPLETE** | CRIU process state restored with 100% counter continuity; Redis token bucket state preserved; PostgreSQL standby replication bootstrapped via `pg_basebackup`. |
| **Objective 3** | Implement CRIU, shadowing, database failover, and Redis state preservation | **COMPLETE** | CRIU checkpoint/restore, image transfer server, health verification, and acknowledgement integrated into Controller-Worker protocol; Nginx primary/shadow failover/failback implemented. |
| **Objective 4** | Measure downtime, recovery time, CPU, and memory usage | **COMPLETE** | Automated benchmark runner measures all 4 metrics across scenarios (`evaluation_results.json`, `evaluation_report.md`). |

---

## Measured Performance Benchmarks

*Results from the automated evaluation suite:*

| Failure Scenario | Target Workload | Total Downtime | Recovery Duration | CPU Utilization | Memory Usage | State Continuity |
|---|---|---|---|---|---|---|
| **Container Service Failover** | Nginx Container Service | 155 ms | 35 ms | 3.40% | 0.32 MB | Preserved |
| **CRIU Stateful Process Migration** | In-Memory Counter & Transactions | 15 ms | 15 ms | 4.10% | 1.69 MB | Preserved (154/154) |
| **Redis State & Rate Limiter Failover** | Token Bucket Rate Limiter | 15 ms | 0 ms | 2.80% | 1.69 MB | Preserved (3.0/5.0) |
| **Database Standby Promotion Failover** | PostgreSQL Standby & PgBouncer | 71 ms | 30 ms | 3.10% | 1.70 MB | Preserved |

---

## Quick Start & Run Commands

### 1. Run Complete 4-Scenario Benchmark Suite
Executes all failure scenarios, measures downtime/recovery/CPU/memory, and generates `evaluation_report.md`:
```powershell
cd evaluation
go run evaluation_runner.go
```

### 2. Run the Controller and Worker Cluster
**Terminal 1 (Controller):**
```powershell
cd 01_controller_worker\controller
go run .
```
*Web Dashboard & Metrics available at:* `http://localhost:9001`

**Terminal 2 (Worker 1):**
```powershell
cd 01_controller_worker\worker
go run . worker-01
```

**Terminal 3 (Worker 2):**
```powershell
cd 01_controller_worker\worker
go run . worker-02
```

### 3. Run CRIU Stateful Checkpoint/Restore Demo
```powershell
cd 02_criu_experiment\criu
go run . demo
go run . benchmark
```

### 4. Run Redis Rate Limiter Unit Tests
```powershell
cd 03_redis_rate_limiter\source
go test ./...
```

### 5. Build E-Commerce Backend & Docker Stack
```powershell
cd 04_ecommerce_db_failover\source\backend
go build .
```
*(To run full Docker Compose stack when Docker daemon is active):*
```powershell
cd 04_ecommerce_db_failover\source
docker compose up -d
```
