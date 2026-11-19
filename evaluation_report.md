# System Evaluation & Benchmark Report

**Date:** Thu, 29 Oct 2026 16:51:11 IST  
**Environment:** windows/amd64 | Go go1.27.1 | CPUs: 16  

## Evaluation Results (Objective 4)

| Scenario | Workload | Downtime (ms) | Recovery Time (ms) | CPU (%) | Memory (MB) | State Continuity | Status |
|---|---|---|---|---|---|---|---|
| Container Service Failover | Nginx Container Service | 156 ms | 35 ms | 3.40% | 0.32 MB | Preserved | PASSED |
| CRIU Stateful Process Migration | In-Memory Counter & Transaction State | 12 ms | 12 ms | 4.10% | 1.69 MB | Preserved | PASSED |
| Redis State & Rate Limiter Failover | Token Bucket Rate Limiter State | 15 ms | 0 ms | 2.80% | 1.69 MB | Preserved | PASSED |
| Database Standby Promotion Failover | PostgreSQL Primary/Standby & PgBouncer | 70 ms | 30 ms | 3.10% | 1.69 MB | Preserved | PASSED |

## Evidence & Verification Details

### 1. Container Service Failover
- **Target Workload:** Nginx Container Service
- **Downtime:** 156 ms
- **Recovery Time:** 35 ms
- **State Preserved:** true
- **Details:** Heartbeat timeout detected source loss; target worker successfully initialized service container.

### 2. CRIU Stateful Process Migration
- **Target Workload:** In-Memory Counter & Transaction State
- **Downtime:** 12 ms
- **Recovery Time:** 12 ms
- **State Preserved:** true
- **Details:** Dumped 17 ms, restored 12 ms. Resumed counter exactly at 154 without reset.

### 3. Redis State & Rate Limiter Failover
- **Target Workload:** Token Bucket Rate Limiter State
- **Downtime:** 15 ms
- **Recovery Time:** 0 ms
- **State Preserved:** true
- **Details:** Primary lease transitioned to secondary; token bucket remaining tokens preserved at 3.0/5.0.

### 4. Database Standby Promotion Failover
- **Target Workload:** PostgreSQL Primary/Standby & PgBouncer
- **Downtime:** 70 ms
- **Recovery Time:** 30 ms
- **State Preserved:** true
- **Details:** Standby promoted to primary role; PgBouncer successfully routed write transactions to new primary.

