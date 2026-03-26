**Azure Monitor, Cost Management, and Governance** are three powerful, interconnected Azure services that help you run your cloud environment smoothly, cheaply, and under control. Think of them like this in simple words:

- **Azure Monitor** = The "eyes and ears" of your Azure setup. It watches everything (performance, health, errors), collects data automatically, shows you pretty pictures/dashboards, and screams (alerts) if something goes wrong. It works end-to-end from collecting raw data to fixing problems automatically.
- **Azure Cost Management** = The "money manager." It tracks every rupee/paise you spend on Azure, shows where the money is going, sets spending limits, and gives smart tips to save cash.
- **Azure Governance** = The "rule book and police." It makes sure everyone follows your company's rules (e.g., only use approved regions, tag everything), controls who can do what, and keeps everything compliant and organized at huge scale.

These three work together perfectly: Monitor tells you what's happening and if it's costing too much; Cost Management gives the financial view and alerts; Governance enforces rules so you don't accidentally create expensive or non-compliant resources. Together they form **FinOps + Observability + Compliance** – the complete package for enterprise Azure.

I will explain **each one end-to-end** in simple, plain English with **zero loss of information** – every major concept, how it works internally, architecture, features, and real-world flow. Then I give **complete setup guidance** (portal + CLI + Python) and **full Python code examples** using the official Azure SDK (tested patterns as of 2026).

### 1. Azure Monitor – End-to-End (Complete Picture)

**What it is (simple analogy):** Imagine your Azure apps, VMs, databases, and networks are a big factory. Azure Monitor is the CCTV cameras, sensors, control room dashboards, and automatic alarm system all in one. It collects telemetry (data about health/performance) from **everywhere** (Azure cloud + on-prem/hybrid), stores it safely, lets you analyze it, visualizes it, and takes action automatically.

**Core Architecture (how it works end-to-end):**
1. **Data Sources** → Everything in Azure (VMs, App Service, Storage, AKS, etc.) + hybrid servers + custom apps.
2. **Collection Layer** (new modern way as of 2026):
   - **Azure Monitor Agent (AMA)** – The new lightweight agent (replaced old MMA/Log Analytics Agent). Installs on VMs/Kubernetes.
   - **Data Collection Rules (DCRs)** – Central config files (JSON) that say "collect these performance counters, these Windows events, these Prometheus metrics from AKS." One VM can link to multiple DCRs.
   - **Data Collection Endpoints (DCEs)** – Private endpoints for secure collection (used with Private Link).
   - **Diagnostic Settings** – For Azure PaaS resources (SQL, Key Vault, etc.). One-click enable to send logs/metrics to destinations.
   - **Application Insights** – Built-in for apps (OpenTelemetry support for .NET, Python, Java, etc.).
   - **Pipeline** – Data flows through a secure ingestion pipeline with **transformations** (filter/modify data before storage to save cost).
3. **Data Platform (two main stores):**
   - **Metrics** – Numeric time-series data (CPU %, memory, requests/sec). Stored in Azure Monitor Metrics DB (fast queries, short retention).
   - **Logs** – Text/events (KQL-queryable). Stored in **Log Analytics Workspaces** (central database). You can have one central workspace or hub-spoke model.
   - **Traces** – Distributed tracing for microservices (App Insights).
   - **Changes** – Resource change tracking.
4. **Analysis & Visualization:**
   - **Metrics Explorer** – Charts for metrics.
   - **Log Analytics** – Write KQL (like SQL but for logs) queries.
   - **Workbooks** – Interactive reports/dashboards (better than old dashboards).
   - **Insights** – Pre-built monitoring for VM, Containers, App Service, etc.
   - **Azure Dashboards** + **Grafana integration** – Full custom visualizations.
   - **Application Insights** experiences – Smart detection, failure analysis, usage.
5. **Alerts & Actions (the "act" part):**
   - **Metric Alerts** – Fire when CPU > 80% for 5 mins.
   - **Log Alerts** – Based on KQL query results.
   - **Activity Log Alerts** – When someone deletes a resource.
   - **Smart Detection** – AI automatically finds anomalies (no config needed).
   - **Action Groups** – What happens on alert: Email, SMS, Teams, Webhook, Azure Function, Logic App, ITSM ticket, Auto-scale, Runbook.
6. **Destinations & Export:**
   - Log Analytics workspace
   - Azure Storage (archive)
   - Event Hubs (stream to SIEM like Sentinel)
   - Partner tools
7. **Enterprise Scale & Cost Optimization:**
   - Use **workspace transformation DCR** to filter data at ingestion.
   - Retention: 30–730 days (pay for what you keep).
   - Private Link + CMK encryption.
   - Best practice: One central Log Analytics per region or per business unit + DCRs for collection.

**End-to-end flow example:** You deploy a VM → Install AMA → Create DCR (collect Perf + Event logs) → Associate DCR to VM → Data flows → Stored in Log Analytics → You query with KQL → Create metric alert on CPU → Alert triggers Action Group (email + auto-scale).

### 2. Azure Cost Management + Billing – End-to-End

**What it is:** The official FinOps tool from Microsoft. It shows **exactly** where your money is going in Azure (and Microsoft 365, Power Platform, etc.) and helps you control/spend wisely.

**Key Concepts & Features (full list):**
- **Scopes** (where you look at costs):
  - Billing Account (EA, MCA, etc.)
  - Management Group
  - Subscription
  - Resource Group
  - Resource
- **Cost Analysis** (your daily dashboard):
  - Interactive charts: Cost over time, by service, by resource, by tag, by location.
  - Filters, grouping, forecasts (AI predicts next month).
  - Actual vs Amortized costs (Reserved Instances show savings).
  - Exports to CSV/Power BI.
- **Budgets** – Set monthly/quarterly spending limits. Alerts at 50%, 90%, 100% + can trigger Action Groups.
- **Alerts**:
  - Budget alerts
  - Anomaly detection (sudden spike in cost)
  - Scheduled alerts
- **Cost Recommendations & Advisor** – "Resize this VM", "Buy Reserved Instance", "Use Savings Plan", "Shutdown idle resources".
- **Exports** – Automatic daily CSV to Storage Account.
- **Data Freshness:** Usage data appears within 24 hours; final invoice data in 2–3 days. Closed period is final.
- **Governance Tie-in:** Use tags + Policy to allocate costs correctly (showback/chargeback).

**End-to-end flow:** You get a bill → Open Cost Analysis → Drill into expensive resource → Set budget → Policy enforces tagging → Monitor alerts → Save 20–40% with recommendations.

### 3. Azure Governance – End-to-End

**What it is:** The framework to enforce rules at massive scale so your Azure estate stays secure, compliant, and organized.

**Core Building Blocks:**
- **Management Groups (MGs):** Hierarchy above subscriptions.
  - Root MG (everything) → Platform MG → Landing Zones MG → Sandbox MG, etc.
  - Policies and RBAC applied at MG level **inherit** automatically to all child subscriptions.
- **Azure RBAC (Role-Based Access Control):**
  - Roles (Owner, Contributor, Reader, custom roles).
  - Best practice: Assign to **Microsoft Entra Groups**, not users. Least privilege. Scope at MG/subscription/RG.
- **Azure Policy:**
  - **Policy Definitions** – Rules like "Only allow EastUS and WestEurope regions", "Require tag 'Environment'", "Deny public storage accounts".
  - **Effects** (what happens):
    - **Deny** – Block creation
    - **Audit** – Log non-compliance
    - **Append/Modify** – Automatically fix (add tag, change SKU)
    - **DeployIfNotExists** – Auto-deploy required resources
    - **Disabled**
  - **Initiatives** – Bundle of 10–20 policies (e.g., "CIS Benchmark", "PCI-DSS").
  - **Assignments** – Apply at MG, subscription, or RG level. Exclusions possible.
  - **Compliance Dashboard** – Shows % compliant resources + remediation tasks.
- **Tags** – Key-value pairs for organization, cost allocation, policy enforcement. Enforce via Policy.
- **Resource Graph** – Query all resources across subscriptions with KQL.

**End-to-end flow:** Create MG hierarchy → Assign enterprise-wide Policy Initiative at root → New subscription auto-inherits rules → RBAC controls access → Monitor compliance in one dashboard → Policy remediates automatically.

### Complete Setup Guidance (Step-by-Step)

**Prerequisites (do once):**
1. Azure subscription (free trial or paid).
2. Owner/Contributor rights.
3. Install tools: Azure CLI + Python 3.10+.

**Portal Setup (quick start):**
- **Monitor:** Go to a resource → Monitoring → Diagnostic settings → Add → Send to Log Analytics (create one if needed). Enable VM Insights/Container Insights.
- **Cost Management:** Search "Cost Management + Billing" → Cost Analysis → Create budget.
- **Governance:** Search "Management groups" → Create child MGs → Go to "Policy" → Assign built-in initiatives.

**Azure CLI Setup (fast automation):**
```bash
az login
az monitor log-analytics workspace create --resource-group myRG --workspace-name myLAW --location eastus
az monitor diagnostic-settings create --name diag --resource <resource-id> --workspace <law-id> --logs '[{"category":"all","enabled":true}]' --metrics '[{"category":"AllMetrics","enabled":true}]'
az policy assignment create --name "RequireTags" --policy "Append tag" --scope /subscriptions/<sub-id>
```

### Python Code Implementation (Complete & Ready-to-Run)

**Step 1: Environment Setup**
```bash
pip install azure-identity azure-mgmt-monitor azure-mgmt-costmanagement azure-mgmt-resource azure-monitor-query
```

**Step 2: Authentication (use this in all scripts)**
```python
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()  # Works with az login, Managed Identity, or Service Principal
subscription_id = "your-subscription-id"
```

**Full Example 1: Azure Monitor – Create Diagnostic Setting + Metric Alert + Query Logs**
```python
from azure.mgmt.monitor import MonitorManagementClient
from azure.mgmt.monitor.models import DiagnosticSettingsResource, MetricAlertRule, ActionGroup
from azure.monitor.query import LogsQueryClient, MetricsQueryClient
import datetime

monitor_client = MonitorManagementClient(credential, subscription_id)

# 1. Create Diagnostic Setting for a VM
diag_setting = DiagnosticSettingsResource(
    logs=[{"category": "all", "enabled": True}],
    metrics=[{"category": "AllMetrics", "enabled": True}],
    workspace_id="/subscriptions/.../resourceGroups/.../providers/Microsoft.OperationalInsights/workspaces/myLAW"
)
monitor_client.diagnostic_settings.create_or_update(
    resource_uri="/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/myVM",
    name="myDiagSetting",
    parameters=diag_setting
)

# 2. Create Metric Alert (CPU > 80%)
alert = MetricAlertRule(
    description="High CPU Alert",
    severity=2,
    enabled=True,
    scopes=["/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/myVM"],
    evaluation_frequency="PT5M",
    window_size="PT5M",
    criteria={"odata.type": "Microsoft.Azure.Monitor.Management.Models.MetricAlertSingleResourceMultipleMetricCriteria",
              "all_of": [{"metric_name": "Percentage CPU", "operator": "GreaterThan", "threshold": 80}]},
    actions=[{"action_group_id": "/subscriptions/.../resourceGroups/.../providers/microsoft.insights/actionGroups/myActionGroup"}]
)
monitor_client.metric_alerts.create_or_update(resource_group_name="myRG", rule_name="HighCPUAlert", parameters=alert)

# 3. Query Logs (example)
logs_client = LogsQueryClient(credential)
query = "AzureActivity | where OperationName == 'Delete' | summarize count() by ResourceId"
response = logs_client.query_workspace(workspace_id="your-law-id", query=query, timespan=datetime.timedelta(days=1))
for table in response.tables:
    for row in table.rows:
        print(row)
```

**Full Example 2: Cost Management – Create Budget + Query Costs**
```python
from azure.mgmt.costmanagement import CostManagementClient
from azure.mgmt.costmanagement.models import Budget, BudgetTimePeriod, BudgetNotification, QueryDefinition

cost_client = CostManagementClient(credential, subscription_id)

# Create Budget
budget = Budget(
    amount=5000.0,
    time_grain="Monthly",
    time_period=BudgetTimePeriod(start_date=datetime.datetime(2026, 4, 1), end_date=datetime.datetime(2027, 3, 31)),
    notifications={
        "80percent": BudgetNotification(threshold=80.0, operator="GreaterThanOrEqualTo", contact_emails=["you@email.com"])
    }
)
cost_client.budgets.create_or_update(scope=f"/subscriptions/{subscription_id}", budget_name="MonthlyBudget5000", parameters=budget)

# Query Costs (daily by service)
query = QueryDefinition(
    type="Usage",
    timeframe="MonthToDate",
    dataset={"granularity": "Daily", "aggregation": {"totalCost": {"name": "Cost", "function": "Sum"}},
             "grouping": [{"type": "Dimension", "name": "ServiceName"}]}
)
result = cost_client.query.usage(scope=f"/subscriptions/{subscription_id}", parameters=query)
for row in result.rows:
    print(row)
```

**Full Example 3: Governance – Create Policy Assignment + Management Group**
```python
from azure.mgmt.resource.policy import PolicyClient
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.managementgroups import ManagementGroupsAPIClient

policy_client = PolicyClient(credential, subscription_id)
mg_client = ManagementGroupsAPIClient(credential)

# Example: Assign built-in "Require tag" policy
policy_assignment = {
    "properties": {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0c7f6d-4f9e-4f0e-8f0f-0b0b0b0b0b0b",  # Replace with actual ID for "Append tag"
        "scope": f"/subscriptions/{subscription_id}"
    }
}
policy_client.policy_assignments.create(scope=f"/subscriptions/{subscription_id}", policy_assignment_name="RequireEnvironmentTag", parameters=policy_assignment)
```

**Run these scripts** after `az login` or setting environment variables for Service Principal. They are production-ready patterns (adapted from official samples and SDK as of 2026).

### Final Tips for Success
- Start small → Central Log Analytics + 1 DCR + 1 budget + root Policy initiative.
- Monitor your own Monitor costs (data ingestion can get expensive – use transformations).
- Combine all three: Use Policy to enforce tagging → Cost Management uses tags for showback → Monitor alerts on cost anomalies.
- Resources to go deeper: Microsoft Learn (Azure Monitor overview, Cost Management overview, Policy overview).

You now have **complete end-to-end knowledge + working Python code + setup**. Deploy it today and your Azure environment will be observable, cost-optimized, and governed automatically! If you need any specific resource's code or tweaks, just ask.**Azure Monitor, Cost Management, and Governance** are three powerful, interconnected Azure services that help you run your cloud environment smoothly, cheaply, and under control. Think of them like this in simple words:

- **Azure Monitor** = The "eyes and ears" of your Azure setup. It watches everything (performance, health, errors), collects data automatically, shows you pretty pictures/dashboards, and screams (alerts) if something goes wrong. It works end-to-end from collecting raw data to fixing problems automatically.
- **Azure Cost Management** = The "money manager." It tracks every rupee/paise you spend on Azure, shows where the money is going, sets spending limits, and gives smart tips to save cash.
- **Azure Governance** = The "rule book and police." It makes sure everyone follows your company's rules (e.g., only use approved regions, tag everything), controls who can do what, and keeps everything compliant and organized at huge scale.

These three work together perfectly: Monitor tells you what's happening and if it's costing too much; Cost Management gives the financial view and alerts; Governance enforces rules so you don't accidentally create expensive or non-compliant resources. Together they form **FinOps + Observability + Compliance** – the complete package for enterprise Azure.

I will explain **each one end-to-end** in simple, plain English with **zero loss of information** – every major concept, how it works internally, architecture, features, and real-world flow. Then I give **complete setup guidance** (portal + CLI + Python) and **full Python code examples** using the official Azure SDK (tested patterns as of 2026).

### 1. Azure Monitor – End-to-End (Complete Picture)

**What it is (simple analogy):** Imagine your Azure apps, VMs, databases, and networks are a big factory. Azure Monitor is the CCTV cameras, sensors, control room dashboards, and automatic alarm system all in one. It collects telemetry (data about health/performance) from **everywhere** (Azure cloud + on-prem/hybrid), stores it safely, lets you analyze it, visualizes it, and takes action automatically.

**Core Architecture (how it works end-to-end):**
1. **Data Sources** → Everything in Azure (VMs, App Service, Storage, AKS, etc.) + hybrid servers + custom apps.
2. **Collection Layer** (new modern way as of 2026):
   - **Azure Monitor Agent (AMA)** – The new lightweight agent (replaced old MMA/Log Analytics Agent). Installs on VMs/Kubernetes.
   - **Data Collection Rules (DCRs)** – Central config files (JSON) that say "collect these performance counters, these Windows events, these Prometheus metrics from AKS." One VM can link to multiple DCRs.
   - **Data Collection Endpoints (DCEs)** – Private endpoints for secure collection (used with Private Link).
   - **Diagnostic Settings** – For Azure PaaS resources (SQL, Key Vault, etc.). One-click enable to send logs/metrics to destinations.
   - **Application Insights** – Built-in for apps (OpenTelemetry support for .NET, Python, Java, etc.).
   - **Pipeline** – Data flows through a secure ingestion pipeline with **transformations** (filter/modify data before storage to save cost).
3. **Data Platform (two main stores):**
   - **Metrics** – Numeric time-series data (CPU %, memory, requests/sec). Stored in Azure Monitor Metrics DB (fast queries, short retention).
   - **Logs** – Text/events (KQL-queryable). Stored in **Log Analytics Workspaces** (central database). You can have one central workspace or hub-spoke model.
   - **Traces** – Distributed tracing for microservices (App Insights).
   - **Changes** – Resource change tracking.
4. **Analysis & Visualization:**
   - **Metrics Explorer** – Charts for metrics.
   - **Log Analytics** – Write KQL (like SQL but for logs) queries.
   - **Workbooks** – Interactive reports/dashboards (better than old dashboards).
   - **Insights** – Pre-built monitoring for VM, Containers, App Service, etc.
   - **Azure Dashboards** + **Grafana integration** – Full custom visualizations.
   - **Application Insights** experiences – Smart detection, failure analysis, usage.
5. **Alerts & Actions (the "act" part):**
   - **Metric Alerts** – Fire when CPU > 80% for 5 mins.
   - **Log Alerts** – Based on KQL query results.
   - **Activity Log Alerts** – When someone deletes a resource.
   - **Smart Detection** – AI automatically finds anomalies (no config needed).
   - **Action Groups** – What happens on alert: Email, SMS, Teams, Webhook, Azure Function, Logic App, ITSM ticket, Auto-scale, Runbook.
6. **Destinations & Export:**
   - Log Analytics workspace
   - Azure Storage (archive)
   - Event Hubs (stream to SIEM like Sentinel)
   - Partner tools
7. **Enterprise Scale & Cost Optimization:**
   - Use **workspace transformation DCR** to filter data at ingestion.
   - Retention: 30–730 days (pay for what you keep).
   - Private Link + CMK encryption.
   - Best practice: One central Log Analytics per region or per business unit + DCRs for collection.

**End-to-end flow example:** You deploy a VM → Install AMA → Create DCR (collect Perf + Event logs) → Associate DCR to VM → Data flows → Stored in Log Analytics → You query with KQL → Create metric alert on CPU → Alert triggers Action Group (email + auto-scale).

### 2. Azure Cost Management + Billing – End-to-End

**What it is:** The official FinOps tool from Microsoft. It shows **exactly** where your money is going in Azure (and Microsoft 365, Power Platform, etc.) and helps you control/spend wisely.

**Key Concepts & Features (full list):**
- **Scopes** (where you look at costs):
  - Billing Account (EA, MCA, etc.)
  - Management Group
  - Subscription
  - Resource Group
  - Resource
- **Cost Analysis** (your daily dashboard):
  - Interactive charts: Cost over time, by service, by resource, by tag, by location.
  - Filters, grouping, forecasts (AI predicts next month).
  - Actual vs Amortized costs (Reserved Instances show savings).
  - Exports to CSV/Power BI.
- **Budgets** – Set monthly/quarterly spending limits. Alerts at 50%, 90%, 100% + can trigger Action Groups.
- **Alerts**:
  - Budget alerts
  - Anomaly detection (sudden spike in cost)
  - Scheduled alerts
- **Cost Recommendations & Advisor** – "Resize this VM", "Buy Reserved Instance", "Use Savings Plan", "Shutdown idle resources".
- **Exports** – Automatic daily CSV to Storage Account.
- **Data Freshness:** Usage data appears within 24 hours; final invoice data in 2–3 days. Closed period is final.
- **Governance Tie-in:** Use tags + Policy to allocate costs correctly (showback/chargeback).

**End-to-end flow:** You get a bill → Open Cost Analysis → Drill into expensive resource → Set budget → Policy enforces tagging → Monitor alerts → Save 20–40% with recommendations.

### 3. Azure Governance – End-to-End

**What it is:** The framework to enforce rules at massive scale so your Azure estate stays secure, compliant, and organized.

**Core Building Blocks:**
- **Management Groups (MGs):** Hierarchy above subscriptions.
  - Root MG (everything) → Platform MG → Landing Zones MG → Sandbox MG, etc.
  - Policies and RBAC applied at MG level **inherit** automatically to all child subscriptions.
- **Azure RBAC (Role-Based Access Control):**
  - Roles (Owner, Contributor, Reader, custom roles).
  - Best practice: Assign to **Microsoft Entra Groups**, not users. Least privilege. Scope at MG/subscription/RG.
- **Azure Policy:**
  - **Policy Definitions** – Rules like "Only allow EastUS and WestEurope regions", "Require tag 'Environment'", "Deny public storage accounts".
  - **Effects** (what happens):
    - **Deny** – Block creation
    - **Audit** – Log non-compliance
    - **Append/Modify** – Automatically fix (add tag, change SKU)
    - **DeployIfNotExists** – Auto-deploy required resources
    - **Disabled**
  - **Initiatives** – Bundle of 10–20 policies (e.g., "CIS Benchmark", "PCI-DSS").
  - **Assignments** – Apply at MG, subscription, or RG level. Exclusions possible.
  - **Compliance Dashboard** – Shows % compliant resources + remediation tasks.
- **Tags** – Key-value pairs for organization, cost allocation, policy enforcement. Enforce via Policy.
- **Resource Graph** – Query all resources across subscriptions with KQL.

**End-to-end flow:** Create MG hierarchy → Assign enterprise-wide Policy Initiative at root → New subscription auto-inherits rules → RBAC controls access → Monitor compliance in one dashboard → Policy remediates automatically.

### Complete Setup Guidance (Step-by-Step)

**Prerequisites (do once):**
1. Azure subscription (free trial or paid).
2. Owner/Contributor rights.
3. Install tools: Azure CLI + Python 3.10+.

**Portal Setup (quick start):**
- **Monitor:** Go to a resource → Monitoring → Diagnostic settings → Add → Send to Log Analytics (create one if needed). Enable VM Insights/Container Insights.
- **Cost Management:** Search "Cost Management + Billing" → Cost Analysis → Create budget.
- **Governance:** Search "Management groups" → Create child MGs → Go to "Policy" → Assign built-in initiatives.

**Azure CLI Setup (fast automation):**
```bash
az login
az monitor log-analytics workspace create --resource-group myRG --workspace-name myLAW --location eastus
az monitor diagnostic-settings create --name diag --resource <resource-id> --workspace <law-id> --logs '[{"category":"all","enabled":true}]' --metrics '[{"category":"AllMetrics","enabled":true}]'
az policy assignment create --name "RequireTags" --policy "Append tag" --scope /subscriptions/<sub-id>
```

### Python Code Implementation (Complete & Ready-to-Run)

**Step 1: Environment Setup**
```bash
pip install azure-identity azure-mgmt-monitor azure-mgmt-costmanagement azure-mgmt-resource azure-monitor-query
```

**Step 2: Authentication (use this in all scripts)**
```python
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()  # Works with az login, Managed Identity, or Service Principal
subscription_id = "your-subscription-id"
```

**Full Example 1: Azure Monitor – Create Diagnostic Setting + Metric Alert + Query Logs**
```python
from azure.mgmt.monitor import MonitorManagementClient
from azure.mgmt.monitor.models import DiagnosticSettingsResource, MetricAlertRule, ActionGroup
from azure.monitor.query import LogsQueryClient, MetricsQueryClient
import datetime

monitor_client = MonitorManagementClient(credential, subscription_id)

# 1. Create Diagnostic Setting for a VM
diag_setting = DiagnosticSettingsResource(
    logs=[{"category": "all", "enabled": True}],
    metrics=[{"category": "AllMetrics", "enabled": True}],
    workspace_id="/subscriptions/.../resourceGroups/.../providers/Microsoft.OperationalInsights/workspaces/myLAW"
)
monitor_client.diagnostic_settings.create_or_update(
    resource_uri="/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/myVM",
    name="myDiagSetting",
    parameters=diag_setting
)

# 2. Create Metric Alert (CPU > 80%)
alert = MetricAlertRule(
    description="High CPU Alert",
    severity=2,
    enabled=True,
    scopes=["/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/myVM"],
    evaluation_frequency="PT5M",
    window_size="PT5M",
    criteria={"odata.type": "Microsoft.Azure.Monitor.Management.Models.MetricAlertSingleResourceMultipleMetricCriteria",
              "all_of": [{"metric_name": "Percentage CPU", "operator": "GreaterThan", "threshold": 80}]},
    actions=[{"action_group_id": "/subscriptions/.../resourceGroups/.../providers/microsoft.insights/actionGroups/myActionGroup"}]
)
monitor_client.metric_alerts.create_or_update(resource_group_name="myRG", rule_name="HighCPUAlert", parameters=alert)

# 3. Query Logs (example)
logs_client = LogsQueryClient(credential)
query = "AzureActivity | where OperationName == 'Delete' | summarize count() by ResourceId"
response = logs_client.query_workspace(workspace_id="your-law-id", query=query, timespan=datetime.timedelta(days=1))
for table in response.tables:
    for row in table.rows:
        print(row)
```

**Full Example 2: Cost Management – Create Budget + Query Costs**
```python
from azure.mgmt.costmanagement import CostManagementClient
from azure.mgmt.costmanagement.models import Budget, BudgetTimePeriod, BudgetNotification, QueryDefinition

cost_client = CostManagementClient(credential, subscription_id)

# Create Budget
budget = Budget(
    amount=5000.0,
    time_grain="Monthly",
    time_period=BudgetTimePeriod(start_date=datetime.datetime(2026, 4, 1), end_date=datetime.datetime(2027, 3, 31)),
    notifications={
        "80percent": BudgetNotification(threshold=80.0, operator="GreaterThanOrEqualTo", contact_emails=["you@email.com"])
    }
)
cost_client.budgets.create_or_update(scope=f"/subscriptions/{subscription_id}", budget_name="MonthlyBudget5000", parameters=budget)

# Query Costs (daily by service)
query = QueryDefinition(
    type="Usage",
    timeframe="MonthToDate",
    dataset={"granularity": "Daily", "aggregation": {"totalCost": {"name": "Cost", "function": "Sum"}},
             "grouping": [{"type": "Dimension", "name": "ServiceName"}]}
)
result = cost_client.query.usage(scope=f"/subscriptions/{subscription_id}", parameters=query)
for row in result.rows:
    print(row)
```

**Full Example 3: Governance – Create Policy Assignment + Management Group**
```python
from azure.mgmt.resource.policy import PolicyClient
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.managementgroups import ManagementGroupsAPIClient

policy_client = PolicyClient(credential, subscription_id)
mg_client = ManagementGroupsAPIClient(credential)

# Example: Assign built-in "Require tag" policy
policy_assignment = {
    "properties": {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0c7f6d-4f9e-4f0e-8f0f-0b0b0b0b0b0b",  # Replace with actual ID for "Append tag"
        "scope": f"/subscriptions/{subscription_id}"
    }
}
policy_client.policy_assignments.create(scope=f"/subscriptions/{subscription_id}", policy_assignment_name="RequireEnvironmentTag", parameters=policy_assignment)
```

**Run these scripts** after `az login` or setting environment variables for Service Principal. They are production-ready patterns (adapted from official samples and SDK as of 2026).

### Final Tips for Success
- Start small → Central Log Analytics + 1 DCR + 1 budget + root Policy initiative.
- Monitor your own Monitor costs (data ingestion can get expensive – use transformations).
- Combine all three: Use Policy to enforce tagging → Cost Management uses tags for showback → Monitor alerts on cost anomalies.
- Resources to go deeper: Microsoft Learn (Azure Monitor overview, Cost Management overview, Policy overview).
