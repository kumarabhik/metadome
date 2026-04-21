# Multi-Region Cloud Deployment

**Phase:** 5
**Roadmap Item:** Infrastructure: Multi-region cloud deployment for low-latency OEM CAD serving

---

## Objective

Deploy the X-Ray Vision Diagnostics cloud infrastructure across multiple AWS regions to:
1. Serve OEM CAD data from geographically close regions — reducing the latency of CAD cache refresh and model updates to edge servers
2. Meet data residency requirements for Canada (ca-central-1) and UK (eu-west-2) markets entering Phase 5
3. Eliminate single-region failure as a platform risk as we scale to 500+ bays

The edge server in each bay handles the real-time render path — cloud latency does not affect the critical <40ms repair interaction path. Cloud infrastructure supports: CAD model delivery, sensor telemetry ingestion, dashboard data, recording uploads, and configuration management.

---

## Current State (Phase 4)

All infrastructure runs in a single AWS region: **us-east-1** (N. Virginia).

Services deployed:
- CAD ingestion pipeline (ECS Fargate)
- VIN-indexed CAD cache (S3 + CloudFront)
- DiagnosticCore RAG corpus (S3 + OpenSearch Serverless)
- Telemetry ingestion (Kinesis Data Streams → S3)
- Manager dashboard backend (ECS Fargate + RDS Aurora PostgreSQL)
- Video recording storage (S3)
- Configuration management (AWS Systems Manager Parameter Store)

Single-region risk: a us-east-1 outage (rare but has occurred — Nov 2020, Dec 2021) would take all cloud-dependent features offline. Edge servers continue operating in cached mode but cannot receive new CAD updates, cannot sync dashboard data, and cannot upload recordings.

---

## Target Multi-Region Architecture

### Regions

| Region | AWS Code | Purpose |
|---|---|---|
| US East (Primary) | us-east-1 | All existing services; primary for US market |
| US West (HA Failover) | us-west-2 | Hot standby for US; failover target |
| Canada (Data Residency) | ca-central-1 | Canada market data — PII, repair events, recordings |
| UK (Data Residency) | eu-west-2 | UK market data — PII, repair events, recordings |

### Data Sovereignty Boundaries
- US dealer data: stays in us-east-1 / us-west-2 only
- Canadian dealer data: ca-central-1 only — no replication to US regions
- UK dealer data: eu-west-2 only — no replication to US regions
- De-identified aggregate telemetry (no PII, no VIN): replicated globally for fleet baseline models

---

## Service-by-Service Multi-Region Plan

### 1. CAD Model Distribution (S3 + CloudFront)
Current: Single S3 bucket in us-east-1, global CloudFront distribution.

Change: Add S3 origin in ca-central-1 (Canada market CAD files) and eu-west-2 (UK market CAD files). Canadian edge servers pull from ca-central-1 S3; UK edge servers pull from eu-west-2 S3.

- CAD files are not PII — they can be replicated globally. But for simplicity and compliance cleanliness, regional files stay in regional buckets.
- CloudFront: existing global distribution covers CDN caching for all markets — no change needed. Regional S3 buckets serve as origin for their respective market's edge servers (direct S3 pull, not through CloudFront — edge servers are fixed locations, not browsers).

### 2. DiagnosticCore RAG Corpus
Current: OpenSearch Serverless in us-east-1. OEM service bulletins indexed here.

Change: Replicate RAG corpus to ca-central-1 and eu-west-2. OEM bulletins are not PII and can be replicated freely. Use OpenSearch Serverless in each region with daily snapshot restore from us-east-1 primary.

- Canadian/UK edge servers query their regional OpenSearch instance — eliminates transatlantic query latency for initial CAD layer manifest generation
- RAG corpus update pipeline: ingest to us-east-1 → async replicate to ca-central-1 and eu-west-2 within 4 hours

### 3. Telemetry Ingestion
Current: Kinesis Data Streams → S3 → Athena queries.

Change: Each market writes telemetry to a regional Kinesis stream → regional S3:
- US: us-east-1 Kinesis → us-east-1 S3
- Canada: ca-central-1 Kinesis → ca-central-1 S3
- UK: eu-west-2 Kinesis → eu-west-2 S3

De-identified aggregate data (no PII, no VIN, aggregated to fleet-level sensor baselines): copied from all regions to a global analytics bucket in us-east-1 for the predictive maintenance fleet baseline model.

### 4. Manager Dashboard Backend
Current: ECS Fargate + RDS Aurora PostgreSQL in us-east-1.

Change: Deploy regional instances in ca-central-1 and eu-west-2. Each regional dashboard backend writes to its regional Aurora instance. Cross-region replication is not used for dashboard data — no commingling of market data.

US HA: RDS Aurora Global Database between us-east-1 (writer) and us-west-2 (reader/failover). Failover time: < 1 minute (Aurora Global Database RPO ~1s, RTO <1m).

### 5. Configuration Management
Current: AWS Systems Manager (SSM) Parameter Store in us-east-1. Edge servers pull config on boot and on 1-hour refresh cycle.

Change: Replicate non-sensitive config parameters to all regions via SSM replication. Sensitive parameters (API keys, OEM credentials): regional SSM only — US secrets stay in US regions, Canada secrets in ca-central-1, UK secrets in eu-west-2.

### 6. Video Recording Storage
Current: S3 in us-east-1.

Change: Canada recordings → ca-central-1 S3. UK recordings → eu-west-2 S3. US recordings remain in us-east-1. No cross-region replication — recordings are PII-adjacent (contain VIN and repair context).

---

## US High-Availability Failover

### Scenario: us-east-1 Partial or Full Outage

1. Health checks: Route 53 health checks on us-east-1 ALB endpoint (30-second interval)
2. Failover: Route 53 DNS failover to us-west-2 ALB within 60 seconds of unhealthy check
3. us-west-2 readiness: Aurora Global Database writer promoted automatically; ECS services in us-west-2 are pre-deployed (scale-to-zero — scale up on failover event via CloudWatch alarm)
4. CAD cache: us-west-2 S3 bucket is sync'd from us-east-1 daily; edge servers have 72-hour local CAD cache — a us-east-1 outage does not affect active repair sessions
5. Recovery: when us-east-1 recovers, Route 53 fails back; Aurora Global Database demotes us-west-2 back to reader

### RPO / RTO Targets

| System | RPO | RTO |
|---|---|---|
| Dashboard / API | < 30 seconds | < 5 minutes |
| CAD updates to edge servers | 24 hours (daily sync cadence) | N/A (edge cache covers gap) |
| Telemetry ingestion | < 1 minute | < 10 minutes |
| Recording uploads | Best-effort (edge buffers 72h) | N/A (edge buffers cover gap) |

---

## Fleet Management at Scale (500 Edge Servers)

At 500 bays, managing edge server configuration manually is impractical.

### Zero-Touch Provisioning (ZTP)
- New edge server ships pre-loaded with a bootstrap agent
- On first network connection, agent calls home to regional SSM endpoint using hardware ID
- SSM returns: software version, CAD cache bucket ARN, regional endpoints, TLS certs
- Entire config applied automatically — no technician intervention

### OTA Software Updates
- Update packages built and tested in us-east-1
- Staged rollout: 5% of servers → 24-hour soak → 20% → 48-hour soak → 100%
- Rollback: if error rate rises > 2% during soak, automatic rollback via SSM Run Command
- Maintenance window: updates deployed between 11pm–4am local time per edge server timezone (configured at provisioning)

### Fleet Monitoring
- CloudWatch agent on each edge server: CPU, memory, disk, GPU utilization, network latency to regional endpoints
- Custom metrics: CAD cache freshness (hours since last successful sync), active session count, HV-STOP event count
- Alert thresholds: disk > 80% used → alert to ops; CAD cache > 48 hours stale → alert + forced sync

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 22 | Architecture review; select AWS services per region; cost model approved |
| 23 | Provision ca-central-1 and eu-west-2 infrastructure (VPC, S3, Aurora, ECS) |
| 23 | Build ZTP bootstrap agent and SSM provisioning workflow |
| 24 | Deploy RAG corpus replication to ca-central-1 and eu-west-2 |
| 24 | Deploy regional Kinesis + S3 telemetry pipelines |
| 25 | Deploy us-west-2 HA standby (Aurora Global DB, ECS pre-deploy, Route 53 failover) |
| 25 | Failover drill: simulate us-east-1 outage; verify RTO < 5 minutes |
| 26 | ZTP tested on 10 new edge servers |
| 27 | OTA update pipeline built and tested on 50-server fleet |
| 28 | All international market infrastructure live (Canada + UK pilot sites) |
| 30 | 500-server fleet management in production |

---

## Cost Estimate

| Item | Monthly Cost (500 bays) |
|---|---|
| S3 storage (CAD + recordings, all regions) | ~$1,200 |
| CloudFront (CAD delivery) | ~$400 |
| ECS Fargate (dashboard backends, all regions) | ~$2,800 |
| Aurora Global Database (us-east-1 writer + us-west-2 reader) | ~$1,600 |
| Aurora regional (ca-central-1, eu-west-2) | ~$900 |
| Kinesis + telemetry pipeline (all regions) | ~$600 |
| Data transfer (inter-region replication) | ~$300 |
| CloudWatch + monitoring | ~$400 |
| **Total** | **~$8,200/month (~$98K/year)** |

At 500 bays billing at $3,500/year average: infrastructure cost = $98K / $1.75M revenue = 5.6% of revenue. Target: keep below 8%.
