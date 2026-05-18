**Azure App Service Overview – Complete End-to-End (Simple Words)**

**What is Azure App Service?**  
Imagine you want to host a website or API but don’t want to manage servers, operating systems, patching, load balancing, or scaling. Azure App Service is Microsoft’s **fully managed Platform-as-a-Service (PaaS)** that lets you focus only on your code. Azure handles everything underneath: infrastructure, OS updates, security patches, high availability, and basic scaling.

It supports:
- Web Apps (websites)
- API Apps (RESTful backends)
- Mobile App backends
- WebJobs / Functions (background tasks)

**Supported Languages & Runtimes (2026):**  
.NET, .NET Core, Java (SE, Tomcat, JBoss), Node.js, Python (Flask, Django, FastAPI), PHP, Ruby, Go, and **custom Docker containers**.

### App Service Plan – The Heart of Everything

Every App Service app **must** run inside an **App Service Plan** (think of it as a “hosting package” or VM group).

The plan decides:
- How much CPU, RAM, and storage you get.
- Features available (custom domains, scaling, slots, VNet integration, etc.).
- Billing (you pay for the plan, not per app – multiple apps can share one plan).

**Current Pricing Tiers (Simplified Comparison):**

| Tier              | Compute Type     | Best For                  | Key Features                          | Scaling                  | Approx. Cost Example (B1/P1v3) |
|-------------------|------------------|---------------------------|---------------------------------------|--------------------------|--------------------------------|
| **Free (F1)**     | Shared           | Testing, learning         | Basic hosting, 60 CPU min/day        | No                       | Free                           |
| **Shared (D1)**   | Shared           | Light dev/test            | 240 CPU min/day                      | No                       | ~$9–10/month                   |
| **Basic (B1–B3)** | Dedicated        | Small apps, dev/test      | Custom domains, SSL, manual scale    | Manual (up to 3)         | B1 ~$55/month                  |
| **Standard (S1–S3)** | Dedicated     | Production moderate load  | Auto-scale, slots, backups           | Auto (up to 10)          | S1 ~$74/month                  |
| **Premium v3 (P0v3–P3v3)** | Dedicated (better hardware) | Most production workloads | Faster CPUs, VNet integration, higher scale | Auto (up to 30+)     | P1v3 much better value than older tiers |
| **Isolated v2**   | Fully Isolated   | High security / compliance| Runs in your VNet (ASE v3)           | Highest scale            | Highest (dedicated hosts)      |

**Premium v3** is currently the sweet spot for most real-world apps (better performance per dollar than v2).

### Core Concepts & Features (Explained Simply)

1. **Deployment Options**:
   - GitHub Actions / Azure DevOps (CI/CD)
   - `az webapp up` (one-command deploy)
   - Zip deploy, FTP, Local Git
   - Docker / Container Registry
   - VS Code extension (easiest for Python)

2. **Deployment Slots** (Staging Environments):
   - Create “production” + “staging” slots.
   - Deploy to staging → test → swap with zero downtime.
   - Available from Standard tier upward.

3. **Scaling**:
   - **Scale Out** (add more instances) – horizontal.
   - **Scale Up** (bigger VM size) – vertical.
   - Auto-scale rules based on CPU, memory, HTTP queue, custom metrics.

4. **Custom Domains & SSL**:
   - Add yourdomain.com easily.
   - Free managed certificates or bring your own.
   - HTTPS enforced.

5. **Networking & Security**:
   - **VNet Integration** (Regional): Allows your app to reach resources inside your **Project_VNet** (private DBs, VMs, etc.) without public exposure. Uses a delegated subnet.
   - **Private Endpoints**: Make the app completely private (no public internet access).
   - **Access Restrictions**: IP allow/deny lists, Service Tags, or subnets (perfect with Application Gateway).
   - **Managed Identity**: Secure access to Key Vault, Storage, etc. without secrets.
   - **Authentication**: Built-in Azure AD, Google, Facebook, etc.

6. **Monitoring & Diagnostics**:
   - Application Insights integration.
   - Log streaming, Kudu console, diagnostics.
   - Azure Monitor alerts.

### Integration with Previous Components (Load Balancer + Application Gateway + VNet)

- **Recommended Architecture**:
  - Internet → **Application Gateway** (with WAF) → **App Service** (via FQDN or Private Endpoint).
  - App Service → **VNet Integration** to reach resources in **Project_VNet** (Backend-Subnet VMs, Azure SQL with private endpoint, etc.).
  - Use **Access Restrictions** on the App Service to allow traffic **only** from the Application Gateway subnet → blocks direct public access.

- App Service works great as a **backend target** for both Load Balancer and Application Gateway.
- For highest security → Use **App Service Environment v3 (ASE)** (Isolated plan) inside your VNet.

### Complete Python Code – Create App Service + Integrate with Existing VNet

**Prerequisites**: Same Resource Group and VNet from previous scripts.

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.web import WebSiteManagementClient
from azure.mgmt.network import NetworkManagementClient
import os

# CONFIG
SUBSCRIPTION_ID = os.getenv("AZURE_SUBSCRIPTION_ID")
RESOURCE_GROUP = "ProjectRG"
LOCATION = "eastus"
APP_SERVICE_PLAN_NAME = "ProjectAppPlan"
WEB_APP_NAME = "myprojectwebapp"   # Must be globally unique
VNET_NAME = "Project_VNet"
INTEGRATION_SUBNET_NAME = "AppService-Integration-Subnet"  # New subnet for VNet integration

credential = DefaultAzureCredential()
web_client = WebSiteManagementClient(credential, SUBSCRIPTION_ID)
network_client = NetworkManagementClient(credential, SUBSCRIPTION_ID)

# 1. Add a dedicated subnet for VNet Integration (must be /27 or larger, delegated)
print("Adding integration subnet...")
vnet = network_client.virtual_networks.get(RESOURCE_GROUP, VNET_NAME)
subnet = network_client.subnets.begin_create_or_update(
    RESOURCE_GROUP, VNET_NAME, INTEGRATION_SUBNET_NAME,
    {
        "address_prefix": "10.0.3.0/27",   # Small dedicated subnet
        "delegations": [{"name": "delegation", "service_name": "Microsoft.Web/serverFarms"}]
    }
).result()

# 2. Create App Service Plan (Premium v3 recommended)
print("Creating App Service Plan (P1v3)...")
plan = web_client.app_service_plans.begin_create_or_update(
    RESOURCE_GROUP, APP_SERVICE_PLAN_NAME,
    {
        "location": LOCATION,
        "sku": {"name": "P1v3", "tier": "PremiumV3", "capacity": 1},
        "kind": "linux",                    # or "windows"
        "reserved": True                    # Required for Linux
    }
).result()

# 3. Create Web App (Python example)
print("Creating Web App...")
web_app = web_client.web_apps.begin_create_or_update(
    RESOURCE_GROUP, WEB_APP_NAME,
    {
        "location": LOCATION,
        "server_farm_id": plan.id,
        "kind": "app,linux",
        "site_config": {
            "linux_fx_version": "PYTHON|3.12",   # Change as needed
            "app_settings": [
                {"name": "WEBSITE_RUN_FROM_PACKAGE", "value": "1"},  # Good practice
                {"name": "SCM_DO_BUILD_DURING_DEPLOYMENT", "value": "true"}
            ]
        }
    }
).result()

# 4. Enable Regional VNet Integration
print("Enabling VNet Integration...")
web_client.web_apps.begin_create_or_update_vnet_connection(
    RESOURCE_GROUP, WEB_APP_NAME, "vnetIntegration",
    {
        "vnet_resource_id": vnet.id,           # or specific subnet
        "subnet_resource_id": subnet.id
    }
).result()

print(f"Web App https://{WEB_APP_NAME}.azurewebsites.net created and integrated with VNet!")
```

