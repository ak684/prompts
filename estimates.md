 OpenHands Self-Hosting Infrastructure Requirements

  Architecture Overview

  OpenHands has two resource categories:
  - Core Applications - stable, predictable (server, database, Redis, auth)
  - Runtime Pods - dynamic, usage-dependent (where AI agents execute)

  Each active runtime pod consumes 1 vCPU, 3 GiB RAM, 35 GiB storage.

  ---
  Scenario 1: 5 Engineer Pilot

  | Component            | Specification                                           |
  |----------------------|---------------------------------------------------------|
  | Core + Runtime Nodes | 6-8 × (4 vCPU, 16 GiB RAM)                              |
  | Instance types       | AWS c5.xlarge, GCP c2-standard-4, Azure Standard_F4s_v2 |
  | Database             | PostgreSQL: 2 vCPU, 8 GiB RAM (managed service fine)    |
  | Storage              | ~300 GiB total                                          |
  | Total                | ~24-32 vCPU, 96-128 GiB RAM                             |

  Recommended settings:
  - max_concurrent_conversations: 2
  - warm_runtime_count: 2

  ---
  Scenario 2: 200 Engineer Rollout

  | Component       | Specification                                                 |
  |-----------------|---------------------------------------------------------------|
  | Core Cluster    | 6-9 × (8 vCPU, 16 GiB RAM) across 3 AZs                       |
  | Runtime Cluster | 25-35 × (8 vCPU, 16 GiB RAM) with 300+ GiB disk each          |
  | Instance types  | AWS c5.2xlarge, GCP c2-standard-8, Azure Standard_F8s_v2      |
  | Database        | PostgreSQL: 2-4 vCPU, 8-16 GiB RAM + read replica recommended |
  | Storage         | ~7 TB for workspaces                                          |
  | Total           | 30-45 nodes (~250-350 vCPU, 500-700 GiB RAM)                  |

  Recommended settings:
  - max_concurrent_conversations: 3
  - warm_runtime_count: 15

  ---
  Scaling Considerations

  1. Runtime sizing is usage-dependent - scale runtime cluster based on actual concurrent usage patterns.
  2. Warm runtimes - more = better UX (instant start), but higher baseline cost. Start at 5-10% of expected concurrent users.
  3. Auto-scaling essential - configure node pool auto-scaling on runtime cluster to handle demand spikes.
  4. Multi-cluster recommended for 200+ - separate core apps from runtimes for independent scaling.

  ---
  A Note on These Estimates

  The pilot numbers are conservative to ensure a smooth experience. It's possible to run with fewer resources, but underallocation can cause conversation failures when the
  cluster can't provision runtime pods. For a pilot where first impressions matter, we recommend starting with these numbers and scaling down later if monitoring shows consistent
   headroom.

  For the 200-engineer rollout, actual requirements depend heavily on concurrent usage (we've assumed 25-40% of engineers active at peak). Monitor closely in the first weeks and
  adjust runtime cluster size based on observed patterns.
