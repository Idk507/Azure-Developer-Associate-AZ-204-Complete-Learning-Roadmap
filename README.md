# ☁️ Azure Developer Associate (AZ-204) — Complete Learning Roadmap

> **90-Day structured journey** from Azure basics to passing the AZ-204 exam.
> Track daily progress in this repo: [github.com/Idk507/Azure_Learning](https://github.com/Idk507/Azure_Learning)

---

## 📋 Table of Contents

- [Exam Overview](#-exam-overview)
- [Repo Structure](#-repo-structure)
- [Legend](#-legend)
- [Phase 1 — Azure Foundations (Day 01–14)](#-phase-1--azure-foundations--cloud-basics-day-0114)
- [Phase 2 — Azure Compute Solutions (Day 15–35)](#-phase-2--azure-compute-solutions-day-1535)
- [Phase 3 — Azure Storage Solutions (Day 36–52)](#-phase-3--azure-storage-solutions-day-3652)
- [Phase 4 — Security & Identity (Day 53–65)](#-phase-4--security--identity-day-5365)
- [Phase 5 — Integration, Messaging & Monitoring (Day 66–80)](#-phase-5--integration-messaging--monitoring-day-6680)
- [Phase 6 — Exam Sprint (Day 81–90)](#-phase-6--exam-sprint-day-8190)
- [AZ-204 Exam Domain Weights](#-az-204-exam-domain-weights)
- [Resources](#-resources)
- [Progress Tracker](#-progress-tracker)

---

## 🎯 Exam Overview

| Item | Detail |
|------|--------|
| **Exam** | AZ-204: Developing Solutions for Microsoft Azure |
| **Pass Score** | 700 / 1000 |
| **Duration** | 120 minutes |
| **Question Count** | ~40–60 questions |
| **Format** | Multiple choice, case studies, drag-and-drop, code completion |
| **Cost** | $165 USD (discounts available via Microsoft offers) |
| **Provider** | Pearson VUE — online proctored or test center |
| **Renewal** | Free annual online assessment on Microsoft Learn |

---

## 📁 Repo Structure

```
Azure_Learning/
├── README.md                   ← This roadmap
├── roadmap.md                  ← Daily checkpoint tracker
├── Phase1_Foundations/
├── Phase2_Compute/
├── Phase3_Storage/
├── Phase4_Security/
├── Phase5_Integration/
├── Phase6_ExamPrep/
├── Day01/ → Day90/             ← Daily notes, code, labs
└── resources/
    ├── cheat_sheets/
    ├── mock_exams/
    └── cli_reference.md
```

**Daily commit convention:**
```
git add Day{NN}/
git commit -m "Day {NN}: {Topic} — {brief note}"
git push origin main
```

---

## 🔖 Legend

| Tag | Meaning |
|-----|---------|
| 📖 Theory | Concept study, reading Microsoft Docs |
| 🧪 Lab | Hands-on in Azure portal / CLI |
| 💻 Code | Writing SDK code or scripts |
| 🎯 Mock | Practice questions or checkpoint exam |
| 🏗️ Project | End-to-end mini project |

---

## 🟦 Phase 1 — Azure Foundations & Cloud Basics (Day 01–14)

> **Goal:** Build the foundation before AZ-204-specific content. Covers cloud concepts, Azure services, CLI, networking, IAM, and IaC.

---

### Day 01 — What is Cloud Computing & Azure 📖

**Topics:**
- IaaS, PaaS, SaaS models and when to use each
- Azure global infrastructure: regions, availability zones, paired regions, data centers
- Azure vs AWS vs GCP — high-level comparison
- Create a free Azure account and explore the portal

**Commit:** `Day01/cloud_basics.md`

---

### Day 02 — Azure Portal, CLI & PowerShell 🧪

**Topics:**
- Navigate the Azure portal: subscriptions, resource groups, search bar, notifications
- Install Azure CLI: `az login`, `az account list`, `az account set`
- Azure PowerShell: `Connect-AzAccount`, `Get-AzSubscription`
- Azure Cloud Shell (browser-based) setup and usage
- Key CLI commands: `az group create`, `az resource list`, `az vm list`

**Commit:** `Day02/portal_cli_setup.md`

---

### Day 03 — Resource Groups & Subscriptions 📖🧪

**Topics:**
- Azure hierarchy: Management Groups → Subscriptions → Resource Groups → Resources
- Resource Groups as logical containers — lifecycle management
- Tagging strategy: environment, owner, cost-center, project
- Naming conventions (Azure CAF recommendations)
- Resource locks: `ReadOnly` and `Delete` — applying via CLI and portal

**Commit:** `Day03/resource_groups.md`

---

### Day 04 — Azure Networking Fundamentals 📖🧪

**Topics:**
- Virtual Networks (VNet) and subnet design
- Network Security Groups (NSG): inbound and outbound rules, priority, default rules
- VNet peering: local and global peering
- Public vs private IP addresses
- Service endpoints vs private endpoints (conceptual overview)
- Azure DNS: private zones and resolution

**Commit:** `Day04/networking.md`

---

### Day 05 — Microsoft Entra ID (Azure AD) Basics 📖🧪

**Topics:**
- Entra ID tenants, users, groups, and guest accounts
- RBAC: built-in roles (Owner, Contributor, Reader) vs custom roles
- Role assignments: scope levels (management group, subscription, resource group, resource)
- Service Principals and App Registrations: when and why
- Azure AD B2B vs B2C overview

**Commit:** `Day05/entra_id_basics.md`

---

### Day 06 — ARM Templates & Bicep 💻🧪

**Topics:**
- ARM template structure: `$schema`, `contentVersion`, `parameters`, `variables`, `resources`, `outputs`
- Deploy with `az deployment group create --template-file`
- Bicep syntax: `resource`, `param`, `var`, `output`, `module`
- Bicep loops: `[for item in items: { ... }]`
- Convert ARM to Bicep: `az bicep decompile`
- What-if deployment: `az deployment group what-if`

**Commit:** `Day06/arm_bicep/`

---

### Day 07 — Azure DevOps & GitHub Actions Intro 🧪💻

**Topics:**
- Azure DevOps: Repos, Pipelines, Boards, Artifacts, Test Plans overview
- GitHub Actions: workflow YAML structure — `on`, `jobs`, `steps`
- Service connection setup for Azure in GitHub Actions (OIDC — no secrets needed)
- Basic CI pipeline: checkout → build → test → deploy to Azure
- Environment secrets and `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`

**Commit:** `Day07/devops_intro/`

---

### Day 08 — Azure Monitor, Cost Management & Governance 📖🧪

**Topics:**
- Azure Monitor: metrics explorer, log analytics, activity log
- Log Analytics workspace: create, connect resources, run KQL queries
- Azure Cost Management: budgets, alerts, cost analysis
- Azure Policy: `Deny`, `Audit`, `Append`, `DeployIfNotExists` effects
- Policy initiatives (policy sets) and compliance dashboard

**Commit:** `Day08/monitor_governance.md`

---

### Day 09 — Azure SDKs & Developer Tools 💻🧪

**Topics:**
- Azure SDK for Python (`azure-*` packages), .NET (`Azure.*` NuGet), JavaScript (`@azure/*` npm)
- `azure-identity` library: `DefaultAzureCredential` authentication chain
- VS Code Azure extensions: Azure Account, Azure Functions, Azure Storage Explorer
- SDK authentication patterns: environment variables, managed identity, CLI credential
- Install and verify: `pip install azure-identity azure-storage-blob`

**Commit:** `Day09/sdk_setup/`

---

### Day 10 — Azure Compute Options Overview 📖

**Topics:**
- Decision matrix: VMs vs App Service vs Azure Functions vs Containers vs AKS
- When to use ACI vs ACA vs AKS
- Serverless vs PaaS vs IaaS tradeoffs
- Azure pricing calculator for compute
- Shared responsibility model across compute types

**Commit:** `Day10/compute_overview.md`

---

### Day 11 — Azure Virtual Machines Deep Dive 🧪💻

**Topics:**
- VM sizes and families: General Purpose (D-series), Compute Optimized (F-series), Memory (E-series)
- OS disks, data disks, managed disks: Standard HDD, Standard SSD, Premium SSD, Ultra Disk
- VM scale sets (VMSS): autoscaling, rolling upgrades, health probes
- Connecting to VMs: SSH (Linux), RDP (Windows), Azure Bastion (secure browser-based)
- Custom Script Extension and cloud-init for VM configuration

**Commit:** `Day11/virtual_machines/`

---

### Day 12 — Azure Load Balancer & Application Gateway 📖🧪

**Topics:**
- Load Balancer (L4): public vs internal, backend pools, health probes, load balancing rules
- Application Gateway (L7): URL-based routing, multi-site hosting, SSL termination, WAF
- Traffic Manager: DNS-based routing — performance, geographic, weighted, priority methods
- Azure Front Door: global CDN + WAF + load balancing

**Commit:** `Day12/load_balancing.md`

---

### Day 13 — Project: Deploy a VNet + VM + NSG 🏗️🧪💻

**Project:**
- Create a VNet with public and private subnets using Bicep
- Deploy Ubuntu VM in the private subnet
- Deploy a jumpbox VM in the public subnet
- Configure NSG: allow SSH to jumpbox only; allow jumpbox → private VM
- Test connectivity and review Azure Network Watcher

**Commit:** `Day13/project_vnet_vm/`

---

### Day 14 — Phase 1 Review & Checkpoint 🎯

**Topics:**
- Self-quiz: networking, Entra ID, ARM/Bicep, CLI commands
- Review any Day 01–13 topics scoring below confidence level 8/10
- Update `roadmap.md` with Phase 1 checkpoint
- Plan Phase 2 schedule

**Commit:** `Day14/phase1_review.md`

---

## 🟩 Phase 2 — Azure Compute Solutions (Day 15–35)

> **AZ-204 Domain 1 — 25–30% of exam weight**
> Covers App Service, Azure Functions, Containers (ACR, ACI, ACA), and CI/CD for compute.

---

### Day 15 — Azure App Service Overview 📖🧪

**Topics:**
- App Service Plans: Free, Shared, Basic (B1/B2/B3), Standard (S1-S3), Premium (P1v3-P3v3), Isolated (I-series)
- Create Web App via portal and Azure CLI: `az webapp create`
- Runtime stacks: .NET, Python, Node.js, Java, PHP, Ruby
- App Service on Linux vs Windows: differences and limitations
- Built-in features: custom domains, TLS, autoscale, deployment slots

**Commit:** `Day15/app_service_intro/`

---

### Day 16 — App Service: Configuration & Diagnostics 💻🧪

**Topics:**
- Application settings and connection strings (override `appsettings.json`)
- `az webapp config appsettings set` and slot-specific settings
- Diagnostic logging: HTTP access logs, application logs, detailed error pages, failed request tracing
- Log streaming: `az webapp log tail`
- Kudu (SCM) site: `https://<app>.scm.azurewebsites.net` — file system, console, extensions

**Commit:** `Day16/app_service_config/`

---

### Day 17 — App Service: TLS, Custom Domains & Auth 🧪💻

**Topics:**
- Custom domain binding: CNAME and A record configuration
- App Service managed certificates (free TLS for custom domains)
- Uploading and binding PFX certificates
- Easy Auth (built-in authentication): Entra ID, Google, Facebook, GitHub providers
- Client certificate authentication (mTLS) — `X-ARR-ClientCert` header

**Commit:** `Day17/app_service_tls_auth/`

---

### Day 18 — App Service: Autoscaling & Deployment Slots 🧪💻

**Topics:**
- Scale up (vertical): change App Service Plan tier
- Scale out (horizontal): manual, custom autoscale rules (CPU %, memory, HTTP queue)
- Autoscale: minimum instances, maximum instances, cool-down period
- Deployment slots: creating staging, QA slots
- Slot settings (sticky settings) vs swappable settings
- Slot swap: warm-up, swap-with-preview, auto-swap
- Traffic splitting: route percentage of traffic to a slot (A/B testing)

**Commit:** `Day18/autoscale_slots/`

---

### Day 19 — Azure Functions: Core Concepts 📖💻

**Topics:**
- Serverless architecture and cold start behaviour
- Hosting plans: Consumption (pay-per-execution), Flex Consumption, Premium (pre-warmed), Dedicated (App Service Plan)
- Trigger types: HTTP, Timer (CRON), Blob, Queue, Service Bus, Event Hub, Event Grid, Cosmos DB, Durable
- `host.json`: `functionTimeout`, `extensions`, `logging` configuration
- `local.settings.json` for local development (never commit to Git)

**Commit:** `Day19/functions_intro/`

---

### Day 20 — Azure Functions: Bindings & Development 💻🧪

**Topics:**
- Input bindings: read data before function executes (Blob, Cosmos DB, Table, Queue peek)
- Output bindings: write data after function executes (Blob, Cosmos DB, Queue, Event Hub)
- Binding expressions: `{name}`, `{queueTrigger}`, `{blobTrigger}`, datetime patterns
- Local development: Azure Functions Core Tools (`func host start`, `func new`, `func azure functionapp publish`)
- Testing HTTP triggers locally with curl or REST client

**Commit:** `Day20/functions_bindings/`

---

### Day 21 — Azure Functions: Durable Functions 💻🧪

**Topics:**
- Durable Functions patterns:
  - **Function chaining** — sequential execution
  - **Fan-out / Fan-in** — parallel execution with aggregation
  - **Async HTTP API** — long-running operations with polling
  - **Monitor** — recurring check with flexible intervals
  - **Human interaction** — wait for external event with timeout
- Orchestrator function, activity function, entity function roles
- `DurableTaskClient`: `ScheduleNewOrchestrationInstanceAsync`, `GetInstanceAsync`, `TerminateInstanceAsync`
- Durable Functions storage provider: Azure Storage (default) or Cosmos DB

**Commit:** `Day21/durable_functions/`

---

### Day 22 — Docker & Containerization Basics 📖💻🧪

**Topics:**
- Docker fundamentals: images, containers, volumes, networks
- Dockerfile instructions: `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `ENV`, `CMD`, `ENTRYPOINT`
- Multi-stage builds: builder stage → runtime stage (smaller final image)
- Key commands: `docker build -t`, `docker run -p`, `docker ps`, `docker logs`, `docker push`
- `docker-compose.yml` for local multi-container setups
- `.dockerignore` file

**Commit:** `Day22/docker_basics/`

---

### Day 23 — Azure Container Registry (ACR) 🧪💻

**Topics:**
- ACR SKUs: Basic, Standard, Premium (geo-replication, private link, zone redundancy)
- Push image to ACR: `az acr login`, `docker tag`, `docker push`
- ACR Tasks: `az acr build` (cloud build), automated build tasks on git push
- Geo-replication: replicate registry to multiple regions
- ACR webhooks: trigger deployments on image push
- Vulnerability scanning with Microsoft Defender for Containers

**Commit:** `Day23/acr_setup/`

---

### Day 24 — Azure Container Instances (ACI) 🧪💻

**Topics:**
- ACI use cases: batch jobs, CI agents, event-driven short tasks
- Deploy container: `az container create --image --cpu --memory`
- Container groups: multiple containers sharing network namespace and storage
- Persistent storage: mount Azure File Share as volume (`--azure-file-volume-*`)
- Environment variables and secrets (secure environment variables)
- Restart policies: `Always`, `OnFailure`, `Never`

**Commit:** `Day24/aci_deployment/`

---

### Day 25 — Azure Container Apps (ACA) 🧪💻

**Topics:**
- ACA vs AKS: managed Kubernetes without cluster management
- Container Apps Environments: shared network and Log Analytics workspace
- Revisions and replicas: traffic splitting between revisions
- KEDA-based autoscaling: HTTP requests, Azure Queue depth, custom metrics
- Dapr integration: service invocation, pub/sub, state management overview
- `az containerapp create`, `az containerapp update`, `az containerapp revision list`

**Commit:** `Day25/container_apps/`

---

### Day 26 — Azure Kubernetes Service (AKS) Introduction 📖🧪

**Topics:**
- Kubernetes core concepts: pods, deployments, services (ClusterIP, NodePort, LoadBalancer), ingress
- AKS cluster creation: `az aks create`, node pools, system vs user node pools
- `kubectl` essentials: `get`, `describe`, `apply`, `delete`, `logs`, `exec`
- AKS + ACR integration: attach with `az aks update --attach-acr`
- Horizontal Pod Autoscaler (HPA) based on CPU/memory
- AKS networking: Azure CNI vs kubenet

**Commit:** `Day26/aks_intro/`

---

### Day 27 — CI/CD for Containers: GitHub Actions + ACR 💻🧪

**Topics:**
- GitHub Actions workflow: build Docker image, push to ACR
- OIDC authentication to Azure (no stored secrets): `azure/login` action with federated credentials
- Deploy to Container Apps from pipeline: `azure/container-apps-deploy-action`
- Deploy to AKS from pipeline: update `kubectl` manifest or Helm chart
- Environment secrets and variables in GitHub Actions (`${{ secrets.* }}`)

**Commit:** `Day27/cicd_containers/`

---

### Day 28 — App Service: WebJobs & Background Tasks 💻🧪

**Topics:**
- Continuous WebJobs vs triggered WebJobs
- WebJob SDK: `IJobHost`, `QueueTrigger`, `BlobTrigger` attributes
- WebJobs vs Azure Functions: when WebJobs still makes sense
- Running background tasks within App Service using `IHostedService` (.NET)
- WebJob logs in Kudu dashboard

**Commit:** `Day28/webjobs/`

---

### Day 29 — Project: Full Containerized App Deployment 🏗️💻🧪

**Project:**
- Containerize a Python FastAPI or Node.js Express application
- Write multi-stage Dockerfile
- Push to ACR using GitHub Actions with OIDC
- Deploy to Azure Container Apps
- Configure environment variables and Key Vault reference (preview)
- Set up HTTP-based autoscaling (min 1, max 10 replicas)
- Verify with `az containerapp logs show`

**Commit:** `Day29/project_fullapp/`

---

### Day 30 — Compute Review + Practice Questions 🎯

**Topics:**
- AZ-204 Domain 1 targeted practice: App Service, Functions, Containers
- Review: deployment slot swap behaviour, sticky settings
- Review: Function trigger vs binding syntax
- Review: ACI container group networking
- Flash-card the essential CLI commands for Domain 1
- Score weak areas and note for Phase 6 revisit

**Commit:** `Day30/compute_review.md`

---

### Day 31 — App Service: Advanced Networking 💻🧪

**Topics:**
- CORS configuration in App Service: allowed origins, allowed methods
- VNet integration for App Service (outbound to private resources)
- Access restrictions: IP rules, service tag rules, HTTP header rules
- Private endpoints for App Service (inbound traffic restriction)
- App Service Environment (ASE) v3: fully isolated, dedicated VNet

**Commit:** `Day31/app_service_advanced/`

---

### Day 32 — Azure Static Web Apps 🧪💻

**Topics:**
- Deploy React / Angular / Vue / Svelte to Static Web Apps
- Built-in GitHub Actions CI/CD: automatic workflow generation
- Static Web Apps API: linked Azure Functions backend
- Routes configuration: `staticwebapp.config.json`
- Authentication: built-in providers (AAD, GitHub, Twitter)
- Custom domains and free managed TLS

**Commit:** `Day32/static_web_apps/`

---

### Day 33 — Azure Functions: Advanced Patterns 💻🧪

**Topics:**
- Dependency injection in Azure Functions (.NET isolated model)
- Function keys: host keys, function keys, system keys — when to use each
- Premium plan benefits: pre-warmed instances, VNet integration, unlimited execution duration
- Function proxies (legacy) vs APIM in front of Functions
- Cold start mitigation strategies

**Commit:** `Day33/functions_advanced/`

---

### Day 34 — Functions + Service Bus: End-to-End Scenario 💻🧪

**Topics:**
- Wire Service Bus queue as Azure Functions trigger (ServiceBusTrigger binding)
- Dead-letter queue: what triggers DLQ, how to process DLQ messages
- Retry policies: `MaxRetryCount`, exponential backoff in host.json
- Max delivery count on Service Bus queue
- Build an end-to-end order processing pipeline: HTTP trigger → Service Bus → Function → Cosmos DB

**Commit:** `Day34/functions_servicebus/`

---

### Day 35 — Compute Phase Checkpoint & Mini-Exam 🎯

**Topics:**
- 20-question mock exam on AZ-204 Domain 1
- Review every incorrect answer with explanation
- Update `roadmap.md` Phase 2 checkpoint
- Set Phase 3 (Storage) learning goals

**Commit:** `Day35/phase2_exam.md`

---

## 🟧 Phase 3 — Azure Storage Solutions (Day 36–52)

> **AZ-204 Domain 2 — 15–20% of exam weight**
> Covers Blob Storage, Cosmos DB, Queue Storage, Azure SQL, and Redis Cache.

---

### Day 36 — Azure Storage Accounts Fundamentals 📖🧪

**Topics:**
- Storage account types: General-purpose v2 (recommended), BlockBlobStorage, FileStorage, BlobStorage
- Redundancy options: LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS
- Storage services within an account: Blobs, Files, Queues, Tables
- Access tiers: Hot, Cool, Cold, Archive — and when to use each
- Storage account endpoints, firewall, and virtual network rules

**Commit:** `Day36/storage_accounts/`

---

### Day 37 — Azure Blob Storage: Core SDK Operations 💻🧪

**Topics:**
- Blob types: block blobs (general), append blobs (logging), page blobs (disks)
- SDK setup: `BlobServiceClient`, `BlobContainerClient`, `BlobClient`
- Operations: `UploadAsync`, `DownloadAsync`, `DeleteAsync`, `ExistsAsync`
- Listing blobs: `GetBlobsAsync()` with prefix and delimiter
- Setting and retrieving blob properties and metadata
- Blob versioning and soft delete configuration

**Commit:** `Day37/blob_storage_sdk/`

---

### Day 38 — Azure Blob Storage: Access & Security 💻🧪

**Topics:**
- Shared Access Signatures (SAS):
  - Account SAS: access to service-level operations
  - Service SAS: access to specific container or blob
  - User delegation SAS: uses Entra ID credentials (recommended)
- SAS parameters: `permissions`, `startsOn`, `expiresOn`, `ipRange`, `protocol`
- Stored Access Policies: create, update, and revoke without regenerating SAS
- Blob access control: public access levels (Container, Blob, Private)

**Commit:** `Day38/blob_security_sas/`

---

### Day 39 — Azure Blob Storage: Lifecycle & Advanced Features 💻🧪

**Topics:**
- Lifecycle management rules: filter by prefix, blob type, age
  - Transition: Hot → Cool → Archive
  - Delete after N days
- Object replication: async replication between storage accounts (different regions)
- Static website hosting: `$web` container, custom 404 page
- Blob Storage events via Event Grid: `BlobCreated`, `BlobDeleted`
- Immutability policies: WORM (Write Once Read Many) for compliance

**Commit:** `Day39/blob_lifecycle/`

---

### Day 40 — Azure Queue Storage 💻🧪

**Topics:**
- Queue Storage vs Service Bus: when to use which
- SDK operations: `QueueClient`, `SendMessageAsync`, `ReceiveMessagesAsync`, `DeleteMessageAsync`
- Message visibility timeout: prevents duplicate processing
- Message TTL (time-to-live) and maximum message size (64 KB)
- Poison message handling: count visibility timeout exceeded → move to DLQ manually
- Base64 encoding for queue messages

**Commit:** `Day40/queue_storage/`

---

### Day 41 — Azure Table Storage 💻🧪

**Topics:**
- NoSQL key-value store: `PartitionKey` (distribution) + `RowKey` (uniqueness within partition)
- SDK: `TableServiceClient`, `TableClient`, `TableEntity`
- CRUD: `AddEntityAsync`, `GetEntityAsync`, `UpdateEntityAsync`, `DeleteEntityAsync`
- OData query filters: `$filter=PartitionKey eq 'users' and RowKey eq 'user1'`
- Batch operations (transactional): all same PartitionKey required
- Table Storage vs Cosmos DB Table API: feature comparison

**Commit:** `Day41/table_storage/`

---

### Day 42 — Azure Files & Azure File Sync 🧪

**Topics:**
- Azure Files: SMB file shares and NFS shares in the cloud
- Create and mount file share: `az storage share create`, mount on Windows/Linux
- Azure File Sync: cloud tiering, sync groups, server endpoints, cloud endpoints
- Access Azure Files with storage account key or identity-based authentication (Kerberos)
- Azure Files snapshots: share-level point-in-time recovery

**Commit:** `Day42/azure_files/`

---

### Day 43 — Azure Cosmos DB: Core Concepts 📖🧪

**Topics:**
- Multi-model APIs: Core (SQL), MongoDB, Cassandra, Gremlin, Table
- Partitioning: partition key selection — high cardinality, even distribution, aligned with queries
- Request Units (RU/s): what they measure, provisioned throughput vs autoscale vs serverless
- Consistency levels (weakest → strongest):
  1. Eventual
  2. Consistent Prefix
  3. Session (default — recommended for most apps)
  4. Bounded Staleness
  5. Strong
- Global distribution and multi-region write

**Commit:** `Day43/cosmosdb_intro/`

---

### Day 44 — Azure Cosmos DB: SDK Operations 💻🧪

**Topics:**
- SDK setup: `CosmosClient`, `Database`, `Container` objects
- CRUD operations:
  - `CreateItemAsync<T>()` — create
  - `ReadItemAsync<T>()` — read by id + partition key
  - `UpsertItemAsync<T>()` — create or replace
  - `PatchItemAsync()` — partial update
  - `DeleteItemAsync()` — delete
- Queries: LINQ with `GetItemLinqQueryable`, parameterized SQL with `QueryDefinition`
- `RequestOptions`: consistency level override per operation
- Handling `CosmosException`: status codes 404, 409, 429 (too many requests)

**Commit:** `Day44/cosmosdb_sdk/`

---

### Day 45 — Azure Cosmos DB: Change Feed & Advanced 💻🧪

**Topics:**
- Change feed: ordered stream of all inserts and updates (not deletes)
- Change feed processor pattern: `ChangeFeedProcessor`, lease container, `HandleChangesAsync`
- Change feed iterator: manual pull-based consumption
- Azure Functions Cosmos DB trigger (built on change feed)
- Multi-region write: conflict resolution policies — Last Writer Wins, Custom (merge procedure)
- Analytical store and Azure Synapse Link (overview)

**Commit:** `Day45/cosmosdb_advanced/`

---

### Day 46 — Azure SQL Database 🧪💻

**Topics:**
- Azure SQL options: SQL Database (PaaS), SQL Managed Instance, SQL Server on VM
- Deployment models: single database vs elastic pool
- Serverless compute tier: auto-pause and auto-resume
- Connection strings and firewall rules: `az sql server firewall-rule create`
- Azure AD / Entra ID authentication for SQL (token-based, no password)
- Point-in-time restore and long-term retention

**Commit:** `Day46/azure_sql/`

---

### Day 47 — Azure Cache for Redis 💻🧪

**Topics:**
- Redis data structures: strings, hashes, lists, sets, sorted sets, streams
- Caching patterns:
  - **Cache-aside (lazy loading):** check cache → miss → load from DB → populate cache
  - **Write-through:** write to cache and DB simultaneously
  - **Read-through:** cache reads from DB automatically on miss
- SDK: `StackExchange.Redis` (C#) or `ioredis` (Node.js)
- `ConnectionMultiplexer`, `IDatabase`, `StringGetAsync`, `StringSetAsync`, `KeyExpireAsync`
- Azure Cache for Redis SKUs: Basic, Standard, Premium, Enterprise

**Commit:** `Day47/redis_cache/`

---

### Day 48 — Storage Security: Encryption & Private Endpoints 📖🧪

**Topics:**
- Azure Storage Service Encryption (SSE): Microsoft-managed keys vs customer-managed keys (CMK) in Key Vault
- Infrastructure encryption (double encryption)
- HTTPS enforcement: `az storage account update --https-only true`
- Private endpoints for storage accounts: secure access over VNet
- Microsoft Defender for Storage: malware scanning, anomaly detection
- Shared key vs Azure AD authorization for storage

**Commit:** `Day48/storage_security/`

---

### Day 49 — Project: Document Store API 🏗️💻🧪

**Project:**
- Build a REST API (Azure Functions HTTP trigger) that:
  - Accepts file upload → stores in Blob Storage
  - Stores metadata (filename, size, tags, uploadedAt) in Cosmos DB
  - Returns a time-limited SAS token for download
- Implements Blob lifecycle management: archive after 30 days
- Cosmos DB query endpoint: search by metadata tag
- Deployed via GitHub Actions

**Commit:** `Day49/project_docstore/`

---

### Day 50 — Storage Practice: AZ-204 Domain 2 Questions 🎯

**Topics:**
- Cosmos DB consistency level selection by scenario
- SAS token permission scoping questions (`r`, `w`, `d`, `l`, `c`)
- Blob lifecycle management rule syntax (JSON policy)
- SDK method selection: when `UpsertItemAsync` vs `CreateItemAsync`
- Queue Storage vs Service Bus scenario questions

**Commit:** `Day50/storage_practice.md`

---

### Day 51 — Azure Data Factory & Synapse Overview 📖

**Topics:**
- Azure Data Factory: pipelines, datasets, linked services, integration runtimes, triggers
- Synapse Analytics: dedicated SQL pool, serverless SQL pool, Spark pool
- Data integration patterns: ETL vs ELT
- ADF vs Azure Functions for data movement: when each fits
- Logic Apps vs ADF for workflow orchestration

**Commit:** `Day51/data_overview.md`

---

### Day 52 — Storage Phase Checkpoint 🎯

**Topics:**
- 30-question mock on AZ-204 Domain 2 (Storage)
- Full SDK code review: Blob, Cosmos DB, Queue — correct class names and namespaces
- Consistency level drill: match scenario to consistency level
- Update `roadmap.md` checkpoint
- Plan Phase 4 (Security) schedule

**Commit:** `Day52/phase3_checkpoint.md`

---

## 🟪 Phase 4 — Security & Identity (Day 53–65)

> **AZ-204 Domain 3 — 15–20% of exam weight**
> Covers Entra ID authentication, MSAL, Managed Identities, Key Vault, App Configuration, and Microsoft Graph.

---

### Day 53 — Microsoft Entra ID Deep Dive 📖🧪

**Topics:**
- Identity objects: users, groups, service principals, managed identities — differences
- App registrations: client ID, tenant ID, object ID, redirect URIs, application scopes
- Enterprise applications vs app registrations (the two sides of the same coin)
- Microsoft Graph API permissions: delegated (user context) vs application (daemon/service)
- Admin consent vs user consent for permissions
- Token types: access token, ID token, refresh token

**Commit:** `Day53/entra_id_deep/`

---

### Day 54 — OAuth 2.0 & OpenID Connect on Azure 📖💻

**Topics:**
- OAuth 2.0 grant types:
  - **Authorization Code + PKCE** — web apps and SPAs (recommended)
  - **Client Credentials** — service-to-service, no user
  - **Device Code** — CLIs, IoT devices
  - **On-Behalf-Of (OBO)** — middle-tier API calling downstream API on user's behalf
- OpenID Connect: ID tokens, `/.well-known/openid-configuration` endpoint
- Token validation: signature (JWKS), `aud`, `iss`, `exp` claims
- JWT structure: header, payload, signature

**Commit:** `Day54/oauth_oidc/`

---

### Day 55 — MSAL Authentication Library 💻🧪

**Topics:**
- MSAL.NET, MSAL.js, MSAL Python — library setup and initialization
- `ConfidentialClientApplication` (server-side apps with secrets/certs)
- `PublicClientApplication` (user-interactive apps)
- Token acquisition:
  - `AcquireTokenForClient()` — client credentials flow
  - `AcquireTokenOnBehalfOf()` — OBO flow
  - `AcquireTokenInteractive()` — user login (desktop/CLI)
  - `AcquireTokenSilent()` — from cache
- Token cache: in-memory (default), distributed (Redis, SQL) for scale-out
- `.WithAuthority()`, `.WithScopes()`, `.WithTenantId()` builder methods

**Commit:** `Day55/msal_auth/`

---

### Day 56 — Managed Identities 📖💻🧪

**Topics:**
- System-assigned managed identity: tied to resource lifecycle, auto-deleted
- User-assigned managed identity: standalone resource, reusable across multiple services
- Assigning RBAC role to managed identity: `az role assignment create --assignee <principalId>`
- `DefaultAzureCredential` authentication chain (preferred in production):
  1. Environment → Workload Identity → Managed Identity → Azure CLI → VS Code → ...
- `ManagedIdentityCredential` for explicit managed identity use
- Testing locally: `az login` provides Azure CLI credential in the chain

**Commit:** `Day56/managed_identities/`

---

### Day 57 — Azure Key Vault: Secrets & Keys 🧪💻

**Topics:**
- Key Vault object types: secrets (passwords, connection strings), keys (cryptographic), certificates
- Access model: Vault access policy (legacy) vs Azure RBAC (recommended)
- SDK clients: `SecretClient`, `KeyClient`, `CertificateClient`
- Operations: `SetSecretAsync`, `GetSecretAsync`, `DeleteSecretAsync`, `PurgeDeletedSecretAsync`
- Key Vault references in App Service: `@Microsoft.KeyVault(SecretUri=...)`
- Soft delete (mandatory since 2022) and purge protection
- Key Vault firewall: trusted services and VNet integration

**Commit:** `Day57/keyvault_secrets/`

---

### Day 58 — Azure Key Vault: Certificates & Secret Rotation 💻🧪

**Topics:**
- Certificate lifecycle: create self-signed, import PFX, renew, export
- Key Vault certificate issuers: DigiCert, GlobalSign — automated issuance
- `CertificateClient`: `StartCreateCertificateAsync`, `GetCertificateAsync`, `ImportCertificateAsync`
- Secret rotation pattern: Event Grid alert on `SecretNearExpiry` → Logic App / Function → rotate → update
- Key Vault logging to Log Analytics: `AuditEvent` category
- Key Vault private endpoints for network isolation

**Commit:** `Day58/keyvault_certs/`

---

### Day 59 — Azure App Configuration 💻🧪

**Topics:**
- App Configuration vs environment variables vs Key Vault: when to use each
- Feature flags: on/off toggles with targeting filters (user, group, percentage rollout)
- SDK: `ConfigurationClient`, `FeatureManager`
- Dynamic configuration refresh: sentinel key pattern + `RefreshAsync()`
- App Configuration → Key Vault references: store KV URI as config value
- Geo-replication for App Configuration (high availability)
- Labels: environment-specific configuration (`dev`, `staging`, `prod`)

**Commit:** `Day59/app_configuration/`

---

### Day 60 — Shared Access Signatures (SAS) Mastery 💻🎯

**Topics:**
- SAS types comparison:
  | Type | Uses | Can Revoke? |
  |------|------|-------------|
  | Account SAS | Service-level ops | Rotate account key |
  | Service SAS | Specific resource | Via Stored Access Policy |
  | User Delegation SAS | Blob/Queue | Via Entra ID |
- SAS permissions: `r`ead, `w`rite, `d`elete, `l`ist, `c`reate, `a`dd, `u`pdate, `p`rocess
- IP range restriction and HTTPS-only protocol enforcement
- Stored Access Policy: attach to service SAS for revocation without key rotation
- AZ-204 exam: SAS scenario selection practice questions

**Commit:** `Day60/sas_tokens/`

---

### Day 61 — Microsoft Graph API Integration 💻🧪

**Topics:**
- Graph API v1.0 vs beta endpoints
- Common endpoints: `GET /me`, `GET /users`, `GET /groups`, `POST /me/sendMail`
- Graph SDK: `GraphServiceClient` setup with `TokenCredentialAuthProvider`
- Calling Graph from Azure Function with managed identity (application permissions)
- Graph API pagination: `@odata.nextLink` handling
- Throttling: Graph API rate limits and retry-after header

**Commit:** `Day61/microsoft_graph/`

---

### Day 62 — Azure AD B2C for App Developers 📖🧪

**Topics:**
- B2C vs B2B vs workforce identity (Entra ID External ID)
- User flows: sign-up/sign-in, password reset, profile edit
- Custom policies (Identity Experience Framework): XML-based, advanced customization
- Token customization: custom claims mapping from user attributes
- Integrating B2C with MSAL: authority URL format `https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}`

**Commit:** `Day62/adb2c/`

---

### Day 63 — Security in CI/CD Pipelines 💻🧪

**Topics:**
- Federated identity credentials (OIDC) in GitHub Actions: zero stored secrets
  - `azure/login` action with `client-id`, `tenant-id`, `subscription-id`
- Azure DevOps service connections with workload identity federation
- Secret scanning: GitHub secret scanning, `git-secrets`, Gitleaks
- Dependabot: automated dependency update PRs
- Key Vault integration in pipelines: `azure/get-keyvault-secrets` action

**Commit:** `Day63/cicd_security/`

---

### Day 64 — Project: Zero-Trust API with Managed Identity 🏗️💻🧪

**Project:**
- Azure Function with Easy Auth requiring Entra ID token (no anonymous access)
- Function reads database connection string from Key Vault using managed identity
- Configuration loaded from Azure App Configuration (feature flags + settings)
- RBAC: function's managed identity has `Key Vault Secrets User` role only
- No secrets in code, no secrets in environment variables, no secrets in Git
- End-to-end test with `az rest` and `curl` using bearer token

**Commit:** `Day64/project_zerotrust/`

---

### Day 65 — Security Phase Checkpoint + Domain 3 Mock 🎯

**Topics:**
- 25-question AZ-204 Domain 3 practice exam
- Key Vault RBAC vs Access Policy scenario questions
- Managed identity vs service principal selection scenarios
- MSAL flow selection by scenario (client credentials vs OBO vs device code)
- SAS token type selection questions
- Update `roadmap.md` checkpoint

**Commit:** `Day65/phase4_checkpoint.md`

---

## 🟦 Phase 5 — Integration, Messaging & Monitoring (Day 66–80)

> **AZ-204 Domains 4 & 5 — 40–50% combined exam weight**
> Covers API Management, Event Grid, Event Hubs, Service Bus, Azure Monitor, and Application Insights.

---

### Day 66 — Azure API Management (APIM): Setup 🧪💻

**Topics:**
- APIM SKUs: Consumption (serverless), Developer (no SLA), Basic, Standard, Premium (multi-region), Isolated
- Create APIM instance: `az apim create` (takes ~30 minutes)
- Import API: OpenAPI (Swagger) spec, WSDL, WADL, or Logic App
- Products: group APIs, require subscription, apply policies at product level
- Subscriptions: primary key and secondary key, trace options
- Developer portal: API documentation, test console, customization

**Commit:** `Day66/apim_setup/`

---

### Day 67 — APIM: Policies 💻🧪

**Topics:**
- Policy scopes (outer → inner): Global → Product → API → Operation
- **Inbound policies** (request processing):
  - `rate-limit` / `rate-limit-by-key`: limit calls per time window
  - `quota` / `quota-by-key`: call count and/or bandwidth cap
  - `validate-jwt`: validate Entra ID tokens
  - `set-header` / `remove-header`: add/remove HTTP headers
  - `rewrite-uri`: transform request URL
  - `ip-filter`: allow/deny by IP
  - `mock-response`: return static response for testing
- **Backend policies:** `set-backend-service`, `retry`, `forward-request`
- **Outbound policies:** `set-body`, `find-and-replace`, `xml-to-json`
- `<choose>` / `<when>` / `<otherwise>` for conditional policy logic
- Policy expressions: C# expressions in `@(...)` syntax

**Commit:** `Day67/apim_policies/`

---

### Day 68 — APIM: Security & Versioning 💻🧪

**Topics:**
- OAuth 2.0 authorization server configuration in APIM
- `validate-jwt` policy: `openid-config` URL, required claims, `<audiences>`
- Certificate-based (mTLS) authentication for backends: `<authentication-certificate>`
- API versioning strategies:
  - URL segment: `/v1/`, `/v2/`
  - Query string: `?api-version=1.0`
  - Header: `Api-Version: 1.0`
- API revisions: safe changes without version change, revision description, current revision
- APIM + Key Vault: store backend secrets as Named Values referencing Key Vault

**Commit:** `Day68/apim_security/`

---

### Day 69 — Azure Event Grid: Core Concepts 📖💻

**Topics:**
- Event Grid vs Event Hub vs Service Bus — the classic comparison:
  | | Event Grid | Event Hub | Service Bus |
  |-|-----------|-----------|-------------|
  | Pattern | Reactive events | Streaming data | Message queuing |
  | Protocol | HTTPS push | AMQP/Kafka | AMQP |
  | Ordering | No | Per partition | Sessions |
  | Retention | 24 hours | 1–90 days | Up to 14 days |
  | Use case | Blob created, alerts | IoT telemetry, logs | Order processing |
- Topics: system topics (Azure service events) vs custom topics
- Event subscriptions and delivery endpoints: webhook, Function, Queue, Event Hub, Hybrid Connection
- Event schemas: Event Grid schema vs CloudEvents 1.0 (recommended)
- Push delivery vs pull delivery (Event Grid Namespaces — newer model)

**Commit:** `Day69/event_grid_intro/`

---

### Day 70 — Azure Event Grid: Implementation 💻🧪

**Topics:**
- Create custom topic: `az eventgrid topic create`
- Publish events: `POST /api/events` with `aeg-sas-key` header
- Subscribe via webhook: validate ownership with `validationCode` handshake
- Azure Functions Event Grid trigger: CloudEvent or EventGridEvent schema
- Event filtering: subject begins/ends with, advanced filters on event data
- Retry policy: max delivery attempts, event time-to-live
- Dead-letter configuration: storage account for undelivered events
- System topic: Blob Storage `BlobCreated` → Function → resize image

**Commit:** `Day70/event_grid_impl/`

---

### Day 71 — Azure Event Hubs 📖💻🧪

**Topics:**
- Event Hubs architecture: namespace → event hub → partitions → consumer groups
- Partitions: data parallelism, immutable ordered sequence per partition
- Throughput units (Standard) / Processing units (Premium) / Capacity units (Dedicated)
- Producer SDK: `EventHubProducerClient`, `EventDataBatch`, `SendAsync(batch)`
- Consumer SDK: `EventHubConsumerClient`, `EventProcessorClient` (recommended for production)
- Checkpoint store: Azure Blob Storage for `EventProcessorClient` offset management
- Event Hubs Capture: automatically archive events to Azure Data Lake Gen2 or Blob Storage
- Event Hubs for Kafka: drop-in Kafka endpoint (no code change for Kafka producers)

**Commit:** `Day71/event_hubs/`

---

### Day 72 — Azure Service Bus: Queues 💻🧪

**Topics:**
- Service Bus tiers: Basic (queues only), Standard, Premium (isolation, geo-disaster recovery, JMS 2.0)
- Queue concepts: FIFO (with sessions), at-least-once delivery, duplicate detection
- Message lock: peek-lock vs receive-and-delete mode
- SDK: `ServiceBusClient`, `ServiceBusSender`, `ServiceBusReceiver`
- Operations: `SendMessageAsync`, `ReceiveMessageAsync`, `CompleteMessageAsync`, `AbandonMessageAsync`, `DeadLetterMessageAsync`
- Max delivery count: exceeded → automatic dead-lettering
- Message TTL and queue TTL
- Deferred messages: `DeferMessageAsync` + `ReceiveDeferredMessageAsync`

**Commit:** `Day72/service_bus_queues/`

---

### Day 73 — Azure Service Bus: Topics & Subscriptions 💻🧪

**Topics:**
- Topics: one message → multiple subscribers (publish-subscribe)
- Subscription filters:
  - SQL filters: `color = 'red' AND priority > 2`
  - Correlation filters: match on `CorrelationId`, `To`, `ReplyTo`, `Label` (faster, recommended)
  - Boolean filters: `TrueFilter` (all) and `FalseFilter` (none)
- Message sessions: FIFO ordering per session ID, stateful session handling
- `ServiceBusSessionReceiver` for session-aware processing
- Transactions: atomic operations across send + complete (same namespace)
- Service Bus + Azure Functions trigger: `ServiceBusTrigger` binding

**Commit:** `Day73/service_bus_topics/`

---

### Day 74 — Messaging Selection Guide 📖🎯

**Topics:**
- Decision framework: Event Grid vs Event Hubs vs Service Bus vs Queue Storage
- Delivery guarantees: at-most-once vs at-least-once vs exactly-once
- When to combine: Event Grid (trigger) → Service Bus (reliable delivery) → Function (process)
- High-throughput streaming (1M+ events/sec): Event Hubs
- Reliable business messaging with ordering: Service Bus with sessions
- Simple task queuing < 80 GB: Queue Storage
- AZ-204 exam: messaging scenario selection practice

**Commit:** `Day74/messaging_guide.md`

---

### Day 75 — Azure Monitor: Metrics & Alerts 🧪💻

**Topics:**
- Azure Monitor metrics: built-in platform metrics vs custom metrics
- Metric dimensions: filter and split by dimension values
- Metric alerts: static threshold vs dynamic threshold (ML-based)
- Log query alerts: KQL query in Log Analytics → alert if results meet condition
- Activity log alerts: alert on Azure management operations
- Action groups: Email/SMS, Azure Function, Logic App, webhook, ITSM, Automation runbook
- Alert severity: Sev 0 (Critical) → Sev 4 (Verbose)
- Smart groups: automatically group related alerts

**Commit:** `Day75/azure_monitor/`

---

### Day 76 — Application Insights: Setup & Instrumentation 💻🧪

**Topics:**
- Workspace-based Application Insights (recommended) vs classic
- Auto-instrumentation vs manual SDK instrumentation
- SDK setup: connection string (preferred over instrumentation key)
- `TelemetryClient` operations:
  - `TrackEvent(name, properties, metrics)`
  - `TrackMetric(name, value)`
  - `TrackException(exception, properties)`
  - `TrackDependency(type, name, data, duration, success)`
  - `TrackPageView(name)`
- `TelemetryInitializer`: enrich all telemetry with custom properties
- Sampling: adaptive (default), fixed-rate, ingestion sampling
- Live Metrics Stream: real-time telemetry with < 1 second latency

**Commit:** `Day76/app_insights_setup/`

---

### Day 77 — Application Insights: Availability & KQL 💻🧪

**Topics:**
- Availability tests:
  - URL ping test (classic): simple HTTP check
  - Standard test: multi-step with SSL validation, custom headers
  - Custom `TrackAvailability()`: run from any location
- Availability alerts: alert on failure rate or response time
- KQL query essentials for Application Insights:
  ```kusto
  requests | where success == false | summarize count() by bin(timestamp, 1h)
  exceptions | project timestamp, type, outerMessage | order by timestamp desc
  dependencies | where duration > 500 | summarize avg(duration) by target
  ```
- Application Map: visualize dependencies, failure rates, response times
- Failure Analysis, Performance blade, User Flows

**Commit:** `Day77/app_insights_advanced/`

---

### Day 78 — Distributed Tracing & End-to-End Monitoring 💻🧪

**Topics:**
- W3C Trace Context: `traceparent` header propagation across services
- Correlation IDs: `operation_Id` linking requests across App Insights instances
- Distributed tracing in a chain: API → Function → Service Bus → downstream Function
- OpenTelemetry SDK for Azure (OTel + Azure Exporter)
- Application Insights dependency tracking: HTTP calls, SQL, Redis, Service Bus
- End-to-end transaction view in Application Insights portal

**Commit:** `Day78/distributed_tracing/`

---

### Day 79 — Project: Event-Driven Order Pipeline 🏗️💻🧪

**Project:**
- Order API (APIM + Function): `POST /orders` with JWT validation policy
- Event Grid custom topic: publish `OrderPlaced` event on each order
- Subscriber Function 1: consumes event, enriches with product data from Cosmos DB, sends to Service Bus
- Subscriber Function 2: reads from Service Bus, stores order in Cosmos DB, writes invoice PDF to Blob
- Application Insights: end-to-end distributed trace across all Functions
- Azure Monitor alert: alert if Service Bus DLQ depth > 0

**Commit:** `Day79/project_order_pipeline/`

---

### Day 80 — Integration & Monitoring Phase Checkpoint 🎯

**Topics:**
- 35-question mock: APIM, Event Grid, Event Hubs, Service Bus, App Insights
- Policy XML syntax review: `rate-limit`, `validate-jwt`, `set-backend-service`
- Messaging selection scenario drill
- App Insights KQL query practice: write queries for common scenarios
- Update `roadmap.md` checkpoint

**Commit:** `Day80/phase5_checkpoint.md`

---

## 🟨 Phase 6 — Exam Sprint (Day 81–90)

> **Final preparation, mock exams, and exam day**
> No new concepts — consolidate, practice, and pass.

---

### Day 81 — Domain Review: Compute (AZ-204 Domain 1) 🎯

**Focus:**
- Speed-review: App Service deployment slots, TLS, autoscale rules
- Azure Functions: trigger + binding syntax for all trigger types
- Container registry → Container Apps: full deployment flow
- Durable Functions: orchestrator patterns by name
- Re-attempt any Domain 1 questions where confidence < 8/10

**Commit:** `Day81/review_compute.md`

---

### Day 82 — Domain Review: Storage (AZ-204 Domain 2) 🎯

**Focus:**
- Blob SDK: correct class hierarchy and method names
- Cosmos DB SDK: constructor pattern, `ReadItemAsync` parameters (id + partitionKey)
- Consistency level selection by scenario
- SAS token type selection by constraint
- Lifecycle management policy JSON syntax
- Queue Storage SDK: `ReceiveMessagesAsync` return type, visibility timeout

**Commit:** `Day82/review_storage.md`

---

### Day 83 — Domain Review: Security (AZ-204 Domain 3) 🎯

**Focus:**
- Key Vault: RBAC roles (`Key Vault Secrets User`, `Key Vault Secrets Officer`)
- Managed identity: system vs user-assigned selection criteria
- MSAL flow selection: which grant type for which scenario
- `DefaultAzureCredential` chain order
- Graph API: delegated vs application permission selection
- App Configuration: sentinel key refresh pattern

**Commit:** `Day83/review_security.md`

---

### Day 84 — Domain Review: Monitoring (AZ-204 Domain 4) 🎯

**Focus:**
- App Insights connection string vs instrumentation key (exam prefers connection string)
- Availability test type selection: URL ping vs standard vs custom
- `TelemetryClient` method selection by telemetry type
- Alert rule condition types: metric alert vs log query alert vs activity log alert
- KQL: `summarize`, `project`, `where`, `extend`, `bin()` quick reference

**Commit:** `Day84/review_monitoring.md`

---

### Day 85 — Domain Review: Integration (AZ-204 Domain 5) 🎯

**Focus:**
- APIM policy XML: scope order (global → product → API → operation), inbound/backend/outbound/on-error sections
- `validate-jwt` policy parameters: `openid-config`, `audiences`, `required-claims`
- Event Grid vs Event Hub vs Service Bus: final decision matrix
- Service Bus: dead-letter reason codes, max delivery count behaviour
- Service Bus sessions: FIFO ordering and `ServiceBusSessionReceiver`

**Commit:** `Day85/review_integration.md`

---

### Day 86 — Full Mock Exam #1 (60 Questions · 120 Minutes) 🎯

**Instructions:**
- Simulate real exam conditions: timed, no aids, no notes
- Use Microsoft Learn free practice assessment + SkillCertPro / ExamTopics
- Score and review every incorrect answer with root-cause analysis
- Group errors by domain → update weak area priority list
- Target: ≥ 75% before proceeding

**Commit:** `Day86/mock_exam_1/results.md`

---

### Day 87 — Weak Area Deep Dive 🎯💻

**Focus:**
- Identify lowest-scoring domain from Mock #1
- Re-read Microsoft Learn modules for those specific topics
- Write minimal code samples for any SDK patterns that felt unclear
- Practice 20 targeted questions on the weak domain
- Watch relevant Exam Readiness Zone episode (Microsoft Learn)

**Commit:** `Day87/weak_areas/`

---

### Day 88 — Full Mock Exam #2 (60 Questions · 120 Minutes) 🎯

**Instructions:**
- Second full mock with a fresh question set (different question bank)
- Target: ≥ 80% before scheduling real exam
- Focus especially on case study / scenario questions
- Note any remaining knowledge gaps
- If < 80%: schedule one more day of targeted review before booking

**Commit:** `Day88/mock_exam_2/results.md`

---

### Day 89 — Final Revision: Cheat Sheet & CLI Reference 🎯

**Produce:**
- One-page domain summary covering all 5 AZ-204 domains
- CLI command quick-reference:
  ```bash
  # App Service
  az webapp create / az webapp deployment slot create / az webapp deployment slot swap
  # Functions
  az functionapp create / az functionapp publish
  # Containers
  az acr build / az container create / az containerapp create
  # Storage
  az storage blob upload / az storage container generate-sas
  # Key Vault
  az keyvault secret set / az keyvault secret show
  # APIM
  az apim create / az apim api import
  # Service Bus
  az servicebus namespace create / az servicebus queue create
  ```
- SDK namespace reference card (which NuGet/pip package → which class → which method)
- Exam logistics: confirm ID, test center or online proctored, check system requirements

**Commit:** `Day89/cheat_sheet.md`

---

### Day 90 — AZ-204 Exam Day 🎯

**Checklist:**
- [ ] Rest well the night before — do NOT cram
- [ ] Arrive 15 minutes early (test center) or log in 30 minutes early (online proctored)
- [ ] Valid government-issued photo ID required
- [ ] Score ≥ 700 / 1000 to pass
- [ ] Results available immediately after the exam

**After Passing:**
- [ ] Claim your digital badge from Credly
- [ ] Update LinkedIn with AZ-204 certification
- [ ] Add badge to GitHub README
- [ ] Star this repo and share your achievement!

**What's Next?**

| Certification | Focus |
|---------------|-------|
| AZ-400 | Azure DevOps Engineer Expert |
| AZ-305 | Azure Solutions Architect Expert |
| AZ-500 | Azure Security Engineer Associate |
| AZ-204 Renewal | Free annual assessment on Microsoft Learn |

**Commit:** `Day90/exam_day.md`

---

## 📊 AZ-204 Exam Domain Weights

| Domain | Weight | Phase |
|--------|--------|-------|
| Develop Azure compute solutions | 25–30% | Phase 2 |
| Develop for Azure storage | 15–20% | Phase 3 |
| Implement Azure security | 15–20% | Phase 4 |
| Monitor and troubleshoot Azure solutions | 5–10% | Phase 5 |
| Connect to and consume Azure services | 20–25% | Phase 5 |

> **Key insight:** Domains 1 and 5 together cover ~50% of the exam. Prioritize App Service, Functions, Containers, APIM, Event Grid/Hubs, and Service Bus.

---

## 📚 Resources

### Official Microsoft Resources

| Resource | URL | Cost |
|----------|-----|------|
| AZ-204 Study Guide | [learn.microsoft.com/credentials/certifications/resources/study-guides/az-204](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-204) | Free |
| AZ-204 Learning Path | [learn.microsoft.com → AZ-204](https://learn.microsoft.com/en-us/credentials/certifications/azure-developer/) | Free |
| Free Practice Assessment | [AZ-204 Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-204/practice/assessment?assessment-type=practice&assessmentId=35) | Free |
| Exam Readiness Zone (Video) | [Microsoft Learn Shows → AZ-204](https://learn.microsoft.com/en-us/shows/exam-readiness-zone/?terms=az-204) | Free |
| Azure Sandbox Labs | Microsoft Learn modules with sandbox | Free |
| Azure Documentation | [docs.microsoft.com/azure](https://docs.microsoft.com/azure) | Free |

### Practice Exam Platforms

| Platform | Notes |
|----------|-------|
| Microsoft Learn Practice Assessment | Official — most accurate to real exam style |
| SkillCertPro | Community questions — use for volume practice |
| ExamTopics | Community questions — understand rationale, don't memorize |
| MeasureUp | Official Microsoft partner — paid but high quality |
| Whizlabs | Good scenario-based questions |

### Key SDK Documentation

| SDK | Language | Package |
|-----|----------|---------|
| Azure Blob Storage | Python | `azure-storage-blob` |
| Azure Cosmos DB | Python | `azure-cosmos` |
| Azure Key Vault Secrets | Python | `azure-keyvault-secrets` |
| Azure Identity | Python | `azure-identity` |
| Azure Service Bus | Python | `azure-servicebus` |
| Azure Event Grid | Python | `azure-eventgrid` |
| Azure App Configuration | Python | `azure-appconfiguration` |
| MSAL | Python | `msal` |

### Tools

| Tool | Purpose |
|------|---------|
| Azure CLI | `az` commands for all services |
| Azure Functions Core Tools | Local function development |
| Azure Storage Explorer | GUI for blob/queue/table management |
| VS Code + Azure Extensions | IDE with Azure integration |
| Postman / REST Client (VS Code) | API testing |
| Docker Desktop | Container development |

---

## ✅ Progress Tracker

Copy this into `roadmap.md` and check off each day:

```
Phase 1 — Foundations
[ ] Day 01 — Cloud Computing & Azure Basics
[ ] Day 02 — Azure Portal, CLI & PowerShell
[ ] Day 03 — Resource Groups & Subscriptions
[ ] Day 04 — Azure Networking Fundamentals
[ ] Day 05 — Microsoft Entra ID Basics
[ ] Day 06 — ARM Templates & Bicep
[ ] Day 07 — Azure DevOps & GitHub Actions Intro
[ ] Day 08 — Azure Monitor, Cost Management & Governance
[ ] Day 09 — Azure SDKs & Developer Tools
[ ] Day 10 — Azure Compute Options Overview
[ ] Day 11 — Azure Virtual Machines Deep Dive
[ ] Day 12 — Azure Load Balancer & Application Gateway
[ ] Day 13 — Project: VNet + VM + NSG
[ ] Day 14 — Phase 1 Review & Checkpoint

Phase 2 — Compute Solutions
[ ] Day 15 — App Service Overview
[ ] Day 16 — App Service Config & Diagnostics
[ ] Day 17 — App Service TLS, Custom Domains & Auth
[ ] Day 18 — App Service Autoscaling & Deployment Slots
[ ] Day 19 — Azure Functions: Core Concepts
[ ] Day 20 — Azure Functions: Bindings & Development
[ ] Day 21 — Azure Functions: Durable Functions
[ ] Day 22 — Docker & Containerization Basics
[ ] Day 23 — Azure Container Registry (ACR)
[ ] Day 24 — Azure Container Instances (ACI)
[ ] Day 25 — Azure Container Apps (ACA)
[ ] Day 26 — Azure Kubernetes Service (AKS) Introduction
[ ] Day 27 — CI/CD for Containers: GitHub Actions + ACR
[ ] Day 28 — App Service WebJobs & Background Tasks
[ ] Day 29 — Project: Full Containerized App Deployment
[ ] Day 30 — Compute Review + Practice Questions
[ ] Day 31 — App Service Advanced Networking
[ ] Day 32 — Azure Static Web Apps
[ ] Day 33 — Azure Functions: Advanced Patterns
[ ] Day 34 — Functions + Service Bus End-to-End
[ ] Day 35 — Compute Phase Checkpoint & Mini-Exam

Phase 3 — Storage Solutions
[ ] Day 36 — Storage Accounts Fundamentals
[ ] Day 37 — Blob Storage: Core SDK Operations
[ ] Day 38 — Blob Storage: Access & Security (SAS)
[ ] Day 39 — Blob Storage: Lifecycle & Advanced Features
[ ] Day 40 — Azure Queue Storage
[ ] Day 41 — Azure Table Storage
[ ] Day 42 — Azure Files & Azure File Sync
[ ] Day 43 — Cosmos DB: Core Concepts
[ ] Day 44 — Cosmos DB: SDK Operations
[ ] Day 45 — Cosmos DB: Change Feed & Advanced
[ ] Day 46 — Azure SQL Database
[ ] Day 47 — Azure Cache for Redis
[ ] Day 48 — Storage Security: Encryption & Private Endpoints
[ ] Day 49 — Project: Document Store API
[ ] Day 50 — Storage Practice: Domain 2 Questions
[ ] Day 51 — Azure Data Factory & Synapse Overview
[ ] Day 52 — Storage Phase Checkpoint

Phase 4 — Security & Identity
[ ] Day 53 — Microsoft Entra ID Deep Dive
[ ] Day 54 — OAuth 2.0 & OpenID Connect on Azure
[ ] Day 55 — MSAL Authentication Library
[ ] Day 56 — Managed Identities
[ ] Day 57 — Azure Key Vault: Secrets & Keys
[ ] Day 58 — Azure Key Vault: Certificates & Rotation
[ ] Day 59 — Azure App Configuration
[ ] Day 60 — Shared Access Signatures (SAS) Mastery
[ ] Day 61 — Microsoft Graph API Integration
[ ] Day 62 — Azure AD B2C for App Developers
[ ] Day 63 — Security in CI/CD Pipelines
[ ] Day 64 — Project: Zero-Trust API
[ ] Day 65 — Security Phase Checkpoint + Domain 3 Mock

Phase 5 — Integration, Messaging & Monitoring
[ ] Day 66 — APIM Setup
[ ] Day 67 — APIM Policies
[ ] Day 68 — APIM Security & Versioning
[ ] Day 69 — Event Grid: Core Concepts
[ ] Day 70 — Event Grid: Implementation
[ ] Day 71 — Azure Event Hubs
[ ] Day 72 — Service Bus: Queues
[ ] Day 73 — Service Bus: Topics & Subscriptions
[ ] Day 74 — Messaging Selection Guide
[ ] Day 75 — Azure Monitor: Metrics & Alerts
[ ] Day 76 — Application Insights: Setup & Instrumentation
[ ] Day 77 — Application Insights: Availability & KQL
[ ] Day 78 — Distributed Tracing & End-to-End Monitoring
[ ] Day 79 — Project: Event-Driven Order Pipeline
[ ] Day 80 — Integration & Monitoring Checkpoint

Phase 6 — Exam Sprint
[ ] Day 81 — Domain Review: Compute
[ ] Day 82 — Domain Review: Storage
[ ] Day 83 — Domain Review: Security
[ ] Day 84 — Domain Review: Monitoring
[ ] Day 85 — Domain Review: Integration
[ ] Day 86 — Full Mock Exam #1
[ ] Day 87 — Weak Area Deep Dive
[ ] Day 88 — Full Mock Exam #2
[ ] Day 89 — Final Revision: Cheat Sheet
[ ] Day 90 — AZ-204 EXAM DAY 🎉
```

---

<div align="center">

**Built for [github.com/Idk507/Azure_Learning](https://github.com/Idk507/Azure_Learning)**


</div>
