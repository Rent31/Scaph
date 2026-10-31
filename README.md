# Scaph - scalable distributed object storage

<img src="doc/scaph-logo.png" alt="Scaph logo" width="120" height="120" align="right">

Scaph is an S3-compatible object storage platform built on Ceph. It stores
objects as erasure-coded chunks spread across failure domains, heals itself
after disk or node failures, rebalances automatically when capacity changes,
and exposes a full S3 API with versioning, bucket policies, ACLs, Object Lock,
quotas, usage metering, and rate limiting.

Scaph runs on commodity hardware. Mix any server model, CPU, and disk type
(SATA, SAS, NVMe). There is no vendor lock-in.

## Features

Storage infrastructure
- Erasure coding with configurable profiles (4+2, 8+3, ...) per pool or storage class
- Auto-healing: automatic rebuild after disk or server failure
- Auto-rebalancing: redistribution when nodes or disks are added or removed
- Throttled recovery: rebuild speed limits so customer latency is not affected
- Failure-domain awareness: chunks are placed across hosts or racks, never in one domain
- Mixed disk types: separate pools per disk type, exposed as storage classes
- Hardware-independent cluster: any server model, CPU, and disk type
- NVMe metadata tier: bucket index and metadata on NVMe for small-object performance
- Data scrubbing: periodic integrity checks against silent bit-rot
- Rolling upgrades: upgrade nodes one at a time with zero downtime

S3 API and data
- Core S3 API: multipart upload, presigned URLs, SigV4, CORS
- Path-style and virtual-hosted-style endpoints
- Unlimited objects per bucket; index resharding scales automatically
- Versioning and Object Lock (governance, compliance, legal hold)
- Bucket policies in JSON with conditions (for example source IP)
- Bucket and object ACLs
- Server access logs per bucket
- Bucket notifications (webhooks on create/delete)
- Static website hosting from a bucket
- Lifecycle policies (tier transitions by age)
- Object legal hold (per object and per bucket)

Multi-tenancy and billing
- Tenant isolation: separate namespace, users, and keys per customer
- Sub-users and access keys with rotation
- Quotas: capacity and object-count limits per tenant and bucket
- Usage metering: storage GB-hours, requests, and egress exported to billing
- Rate limiting / QoS per tenant
- Admin provisioning API for tenant signup from the customer portal

Security and compliance
- TLS in transit for all API and replication traffic
- SSE-S3 and SSE-C encryption at rest
- SSE-KMS with external key management (KMS integration)
- Audit logging of admin and API actions
- Object legal hold (WORM) per object and per bucket

Operations
- Monitoring and alerting: disk health, latency, capacity, rebuilds
- Metadata and config backup with scheduling and retention
- Ransomware protection via versioning and Object Lock
- Cross-zone replication with lag monitoring
- Disaster recovery failover/failback (RPO 15 min, RTO 1 hr)
- Global load balancer with endpoint failover across zones
- Ransomware recovery drills with verification
- Per-tenant rate limiting enforcement (token bucket)
- Per-tenant quota enforcement on object writes
- Bucket quarantine for ransomware isolation
- Emergency lock-down for security incidents
- Cost optimization advisor (tier transition recommendations)
- Data egress optimizer for cost reduction
- Auto-tiering by access frequency
- Security scanner for issues and remediation
- Compliance reporting (SOC2, GDPR, HIPAA, PCI-DSS)

Storage tiers (AWS-style)
- Standard: hot data on SAS or NVMe
- Infrequent Access: warm data on SATA with erasure coding
- Archive / Cold: wide erasure coding (for example 8+3) on high-density SATA

## Quick start (single node)

Scaph provides a one-command deploy tool that installs dependencies, creates
the configuration, boots the monitor, manager, storage daemons, and the S3
gateway, and provisions the first tenant:

```
./tools/scaph-deploy
```

The script prints the admin credentials and the URL of the web console when it
finishes. See `tools/scaph-deploy --help` for options.

## Web console and billing dashboard

Scaph ships with a modern IaaS web console (`web/console`) and a user billing
dashboard (`web/billing`). Serve them with any HTTP server, or run the bundled
API server:

```
cd web
./scaph-api
```

The console is for cluster administration: nodes, disks, pools, erasure
profiles, buckets, tenants, keys, and billing. The billing dashboard is for
customers: usage (storage GB-hours, requests, egress), invoices, access keys,
and buckets.

Design rules for both interfaces: square corners only, no gradients, no
emojis, no decorative flourishes.

## Improvements over the base codebase

The following functional changes were made in this overhaul:

1. New IaaS web console for cluster administration (`web/console`).
2. New user billing dashboard with metering and invoices (`web/billing`).
3. New `scaph-api` API server backing both interfaces (`web/scaph-api`).
4. New one-command deploy tool `tools/scaph-deploy` for single-node setup.
5. New `tools/scaph-status` cluster health check tool.
6. New `tools/scaph-meter` usage metering collector for billing export.
7. Dockerfile for the console and API server (`web/Dockerfile`).
8. systemd service unit for the API server (`systemd/scaph-api.service`).
9. OpenAPI spec for the API (`web/openapi.json`).
10. Usage metering exporter hook: RGW usage logs feed the billing API.
11. Billing data model: tenants, invoices, line items, and payment status.
12. Rate-limit and quota UI in the admin console.
13. Audit log viewer in the admin console.
14. Erasure profile manager in the admin console (4+2, 8+3, custom).
15. Recovery throttle controls in the admin console.
16. Scrubbing schedule and status view in the admin console.
17. Storage class UI mapping tiers to disk types (Standard / IA / Archive).
18. Object Lock and versioning controls per bucket.
19. Presigned URL generator in the console.
20. Tenant provisioning API endpoint (POST /admin/tenants).
21. Access key creation and rotation UI for customers.
22. Lifecycle policy engine (background transition of objects between tiers)
23. Server access logs viewer (per-bucket request logs)
24. Bucket notifications / webhooks with retry delivery
25. Static website hosting configuration per bucket
26. KMS / SSE-KMS key management (create, enable, disable, rotate, encrypt/decrypt)
27. Cross-zone replication configuration and lag monitoring
28. Per-tenant rate limiting enforcement (token bucket)
29. Per-tenant quota enforcement on object writes
30. Object legal hold (WORM) per object and per bucket
31. Disaster recovery failover/failback with RPO/RTO targets
32. Ransomware recovery drill runner with verification
33. Cost optimization advisor (tier transition recommendations)
34. Global load balancer with active-passive endpoint failover and health checks
35. Configuration backup with scheduling, retention, and manual creation
36. CDN origin integration per bucket with edge caching configuration
37. Auto-tiering engine: background transitions based on access frequency
38. Compliance reporting: SOC2, GDPR, HIPAA, PCI-DSS report generation
39. Security scanner: finds public buckets, unrotated keys, missing versioning
40. Bucket quarantine: isolate buckets suspected of ransomware activity
41. Network policies: IP allow/block lists per tenant
42. Emergency lock-down: instant cluster-wide or per-bucket freeze
43. Data egress optimizer: access-pattern analysis for egress cost reduction
44. Web console pages for all Phase 2 and Phase 3 features
45. Background subsystems: LB health checker, config backup scheduler, auto-tiering engine, security scanner

## Building

Scaph is C++. Build with CMake and Ninja:

```
./do_cmake.sh
cd build
ninja -j$(nproc)
```

Build dependencies are installed by `./install-deps.sh`. Submodules must be
checked out with `git submodule update --init --recursive`.

Packaging produces `.deb` and `.rpm` packages named `scaph`.

## License

Most of Scaph is dual-licensed under the LGPL version 2.1 or 3.0. See the
file COPYING for a full inventory of licenses by file.
