**Azure App Service Configuration & Diagnostics – Complete End-to-End Explanation (2026)**

This covers **everything** in extreme detail: all configuration sections, how they work with slots, best practices, security, code examples, Python SDK, and full diagnostics/monitoring stack. Explanations use simple words while leaving no gaps.

### 1. App Service Configuration – The Big Picture

Configuration in App Service is managed centrally in the Azure control plane and injected into your running app at startup. It overrides anything in your code’s `appsettings.json`, `web.config`, or environment files.

**Main Sections (Portal → Settings → Configuration)**:

#### A. **Environment Variables / App Settings**
- These are **key-value pairs** turned into **environment variables** for your app.
- Available to all languages (Python: `os.getenv('KEY')`, .NET: `Configuration["KEY"]`, Node.js: `process.env.KEY`, etc.).
- **Encrypted at rest** and transmitted securely.
- **No size limit per setting**, but practical limits apply (keep values reasonable).

**How to Add**:
- Portal: Configuration → Environment variables (or App settings tab).
- They appear as `APPSETTING_` prefixed internally in some contexts.

**Common Examples**:
- `ASPNETCORE_ENVIRONMENT=Production`
- `APPLICATIONINSIGHTS_INSTRUMENTATIONKEY=...`
- Feature flags, API URLs, timeouts.

#### B. **Connection Strings**
- Special section for database/connection info.
- **.NET apps** get them automatically injected into `Configuration.GetConnectionString("Name")`.
- Prefixes for auto-detection:
  - `SQLCONNSTR_`
  - `SQLAZURECONNSTR_`
  - `MYSQLCONNSTR_`
  - `CUSTOMCONNSTR_`
- Values are hidden in the portal by default (click "Show values").

**Best Practice**: Store secrets in **Azure Key Vault** and reference them using `@Microsoft.KeyVault(...)` syntax instead of plain text. This avoids storing secrets in App Service config.

#### C. **General Settings**
- **Platform**: 32-bit or 64-bit.
- **Always On**: Keeps your app loaded (prevents cold starts). Recommended for production.
- **HTTP Version**: 2.0 (better performance).
- **Web Sockets**: Enable for real-time (SignalR, WebSocket apps).
- **ARR Affinity**: Cookie-based session stickiness (useful with multiple instances).
- **Managed Pipeline Mode**: Integrated or Classic (rarely needed).
- **Remote Debugging**: Temporary (48 hours max).
- **Default Documents**: For static sites (e.g., index.html).
- **Handler Mappings**, **Virtual Applications & Directories**.

#### D. **Language Stack Settings**
- Python: Version, startup command (`python -m gunicorn ...` or module).
- .NET: Framework version.
- Node.js, Java, PHP, etc.

#### E. **Path Mappings** (for Linux/Container)
- Startup command, virtual file mappings.

### 2. Slot-Specific (Sticky) Settings – Critical with Deployment Slots

When you swap slots, most settings **swap** with the code. Some must **stay with the slot** (e.g., Production DB vs Staging DB).

**How to Make Sticky**:
- In Portal → Configuration for a specific slot → Check "**Deployment slot setting**" checkbox next to the key.
- Via API/SDK: Use `slotConfigNames` resource.

**Typical Sticky Settings**:
- Connection strings (different DBs per environment).
- `APPINSIGHTS_INSTRUMENTATIONKEY` (if separate resources).
- Environment-specific secrets.
- `WEBSITE_` prefixed settings for behavior.

**Swappable Settings**:
- Feature flags, API endpoints (if same), logging levels.

**Override Behavior** (Advanced): Set `WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS=0` to revert to legacy behavior.

### 3. Python SDK – Complete Configuration Management

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.web import WebSiteManagementClient
import os

SUBSCRIPTION_ID = os.getenv("AZURE_SUBSCRIPTION_ID")
RESOURCE_GROUP = "ProjectRG"
APP_NAME = "myprojectwebapp"
SLOT = "staging"  # or "production"

credential = DefaultAzureCredential()
web_client = WebSiteManagementClient(credential, SUBSCRIPTION_ID)

# Update App Settings + Connection Strings
config = web_client.web_apps.get_configuration(RESOURCE_GROUP, APP_NAME)

new_settings = {
    "ASPNETCORE_ENVIRONMENT": "Production",
    "FEATURE_FLAG_NEW_UI": "true",
    "KEY_VAULT_URL": "https://myvault.vault.azure.net/"
}

# Key Vault Reference Example (secure)
new_settings["SECRET_CONN"] = "@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MyDbConn/...)"

web_client.web_apps.update_configuration(
    RESOURCE_GROUP, APP_NAME,
    {
        "app_settings": new_settings,
        "connection_strings": [
            {
                "name": "DefaultConnection",
                "value": "...",
                "type": "SQLAzure"  # or MySQL, Custom, etc.
            }
        ]
    }
)

# Make settings sticky (slot-specific)
web_client.web_apps.update_slot_configuration_names(
    RESOURCE_GROUP, APP_NAME,
    {
        "app_setting_names": ["SECRET_CONN", "DefaultConnection"],  # These stay with slot
        "connection_string_names": ["DefaultConnection"]
    }
)

print("Configuration updated!")
```

**For Slots**: Use `begin_create_or_update_slot_configuration` or specify slot name.

### 4. Diagnostics & Monitoring – Full Stack

#### A. **Built-in App Service Diagnostics**
- **Diagnose and Solve Problems** (Portal left menu): AI-powered troubleshooter for CPU, memory, availability, HTTP errors, etc.
- **Metrics** (in Monitor section): CPU %, Memory %, HTTP Queue Length, Requests, Data In/Out, etc.
- **Log Stream** (real-time console output).
- **Application Logs**, **Web Server Logs**, **Detailed Error Messages**, **Failed Request Tracing**.

**Enable Logs** (Portal → Monitoring → App Service Logs):
- Application Logging (File System or Blob) – levels: Error, Warning, Information, Verbose.
- Web Server Logging (Storage or File System).
- Detailed Error Logging.
- Failed Request Tracing (FREB logs – very detailed for 500 errors).

#### B. **Application Insights (Recommended for Production)**
- Full APM (Application Performance Monitoring).
- Auto-collects: Requests, Dependencies (SQL, HTTP calls), Exceptions, Page Views, Custom Events/Telemetry.
- Smart Detection (anomalies, performance degradation).
- Availability Tests (ping tests from multiple regions).
- Profiler, Snapshot Debugger.

**Setup**: Portal → Application Insights → Turn on (or add SDK manually for full control). Use OpenTelemetry for vendor-neutral.

#### C. **Diagnostic Settings (Export Logs & Metrics)**
- Send to:
  - Log Analytics Workspace (for queries/alerts).
  - Storage Account (long-term archive).
  - Event Hub (for SIEM).
  - Partner solutions.

**Categories**: AppServiceHTTPLogs, AppServiceConsoleLogs, AppServicePlatformLogs, etc.

#### D. **Kudu / Advanced Tools**
- Go to **Advanced Tools** (Kudu) → `https://myapp.scm.azurewebsites.net`.
- Features: Console (CMD/Bash), Debug console, Log files download, Process explorer, Diagnostic dump (memory dumps), Environment variables viewer, etc.

### 5. Best Practices & Security (End-to-End)

**Configuration Best Practices**:
- Never put secrets in code — use Key Vault references or Managed Identity.
- Use different sticky settings per slot (Prod vs Staging).
- Version config via Infrastructure-as-Code (Terraform/Bicep/Python).
- Limit access: Use RBAC (Contributor, Website Contributor).
- For VNet-integrated apps: Use Private Endpoints + Service Endpoints.

**Diagnostics Best Practices**:
- Always enable Application Insights + Log Analytics.
- Set retention (30-90 days typical; longer for compliance).
- Create Alerts: High CPU, Failed Requests > threshold, Exception rate.
- Use Log Stream + Kudu for live debugging.
- Enable Failed Request Tracing for production troubleshooting.
- Monitor **HTTP Queue Length** + **Requests** to detect overload before it affects users.
- For containers: Enable Docker logging.

**Integration with Previous Components**:
- **Application Gateway + Access Restrictions**: Route only through AGW and log WAF blocks.
- **VNet Integration**: Monitor outbound calls in Application Insights dependencies.
- **Deployment Slots**: Ensure logging keys are sticky if using separate AI resources per slot.
- **NSG / Project_VNet**: Logs help debug blocked probes or traffic.

**Common Issues & Fixes**:
- Settings not visible in code → Restart app or check prefix.
- Connection string not injected → Use correct name/prefix for language.
- Logs not appearing → Enable logging + wait a few minutes; check correct slot.
- High costs → Sample telemetry, filter noisy logs, use retention policies.

This is the **complete, production-ready** knowledge of App Service Configuration & Diagnostics. You can now manage, secure, monitor, and troubleshoot any App Service confidently, integrated with VNet, Application Gateway, Slots, and CI/CD.

**Azure App Service Configuration & Diagnostics – Ultra-Complete End-to-End Reference (2026)**

This guide covers **everything** with no omissions: all configuration sections, environment variables, WEBSITE_ settings, Key Vault/App Configuration references, slot behavior, security, management methods, and the full diagnostics/monitoring/logging stack.

### 1. Configuration Overview & Architecture

Azure App Service configuration is stored in the Azure control plane (not on the VM/container itself). At app startup (or restart), the platform injects settings as **environment variables** into your runtime. These override values in your code files (`appsettings.json`, `web.config`, `application.properties`, etc.).

**Key Principles**:
- Settings are **encrypted at rest** and in transit.
- Changes usually require a restart (except some dynamic ones).
- **Slot-aware**: Most settings swap; sticky ones do not.
- Supports references to external stores (Key Vault, Azure App Configuration) for better security and centralization.

### 2. All Configuration Sections (Portal + Details)

#### A. **Application Settings (Environment Variables)**
Key-value pairs injected as environment variables.

**Access in Code**:
- Python: `os.getenv('MY_KEY')`
- .NET: `Configuration["MY_KEY"]` or `Environment.GetEnvironmentVariable`
- Node.js: `process.env.MY_KEY`
- Java: `System.getenv("MY_KEY")`

**Management**:
- Portal → Configuration → Application settings.
- Can be set at app level or per slot.

#### B. **Connection Strings**
Special handling, especially for .NET. Automatically prefixed internally.

**Supported Types**:
- SQLAzure, MySQL, SQLServer, PostgreSQL, Custom, etc.

**Best Practice**: Avoid plain text. Use Key Vault references.

#### C. **General Settings**
- **Platform**: 32/64-bit (Windows).
- **Always On**: Prevents idle shutdown (critical for production, reduces cold starts).
- **HTTP Version**: 1.1 or 2.0 (default 2.0 in newer plans).
- **WebSockets**: Enable for real-time apps.
- **ARR Affinity**: Session stickiness via cookie (`ARRAffinity`).
- **Managed Pipeline Mode**: Integrated (default) vs Classic (IIS legacy).
- **Default Documents**: index.html, default.aspx, etc.
- **Error Pages**: Custom error handling.
- **Handler Mappings**: For custom ISAPI/CGI (rare).
- **Virtual Applications & Directories**: Map paths like `/api` to sub-directories.

#### D. **Language & Runtime Stack Settings**
- Python: Version + Startup Command (e.g., `gunicorn --bind=0.0.0.0:${PORT} app:app`).
- .NET: Framework version, .NET Core version.
- Node.js: Version, npm command.
- Java: Java version, server (Tomcat, JBoss), etc.
- PHP, Ruby, Go, Docker.

#### E. **Other Important Tabs**
- **Scale Out / Scale Up** (part of App Service Plan but visible).
- **Custom Domains + TLS/SSL**.
- **Authentication** (Easy Auth / Microsoft Identity).
- **Backup & Restore**.
- **Networking** (VNet Integration, Private Endpoints, Access Restrictions, Hybrid Connections).

### 3. Important WEBSITE_* & System Environment Variables

Azure injects many read-only or configurable variables. Full list (key ones):

**Read-Only System Variables**:
- `WEBSITE_SITE_NAME`, `WEBSITE_RESOURCE_GROUP`, `WEBSITE_OWNER_NAME`, `REGION_NAME`, `WEBSITE_PLATFORM_VERSION`, `HOME` (D:\home on Windows), `SERVER_PORT`, etc.

**Configurable WEBSITE_ Settings** (via App Settings):
- `WEBSITE_TIME_ZONE` (e.g., "India Standard Time").
- `WEBSITE_RUN_FROM_PACKAGE=1` (run from zip, faster & recommended).
- `WEBSITE_LOCAL_CACHE_OPTION=Always` (local cache for better performance).
- `WEBSITE_SWAP_WARMUP_PING_PATH=/health` (custom warm-up path for slots).
- `WEBSITE_WEBDEPLOY_USE_SCM=true`.
- `WEBSITE_DYNAMIC_CACHE=0/1`.
- `WEBSITE_NODE_DEFAULT_VERSION`, `PYTHON_VERSION`, etc.
- `SCM_DO_BUILD_DURING_DEPLOYMENT=true`.
- `WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS=0` (legacy slot behavior).

Many more exist — check official reference for your stack.

### 4. Advanced References (No Secrets in Config)

#### Key Vault References
Syntax: `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MySecret/)`

Requires: Managed Identity + proper Key Vault access policy / RBAC.

#### Azure App Configuration References
Syntax: `@Microsoft.AppConfiguration(Endpoint=https://myconfig.azconfig.io;Key=MyKey;Label=Production)`

Great for centralized, feature-flag-enabled config.

### 5. Deployment Slots & Configuration Behavior

- **Swapped**: Code, most app settings, connection strings (unless sticky).
- **Sticky (Slot-Specific)**: Mark checkbox "Deployment slot setting" or via API (`slotConfigNames`).
  - Typical sticky: Connection strings, secrets, environment-specific flags, `WEBSITE_` behavior settings.

**During Swap**:
- Target (production) settings are applied to source slot for warm-up.
- Routing pointer swaps.
- Sticky settings remain with their original slot.

### 6. Configuration Management Methods (All Ways)

- **Portal**
- **Azure CLI** (`az webapp config appsettings list/set/delete`)
- **PowerShell**
- **Python SDK** (as in previous response)
- **ARM / Bicep / Terraform**
- **VS Code Azure extension**

**CLI Example (Save/Load)**:
```bash
az webapp config appsettings list --name myapp --resource-group rg > settings.json
az webapp config appsettings set --name myapp --resource-group rg --settings @settings.json
```

### 7. Diagnostics & Monitoring – Full Stack

#### A. **Built-in App Service Diagnostics**
- **Diagnose and Solve Problems** (AI-powered): Availability & Performance, App Down, Slow App, CPU/Memory, HTTP Errors, etc.
- Tiles with guided workflows.
- Integrates with Application Insights for deeper root cause.

#### B. **Diagnostic Logs (Built-in Logging)**
Enable via: Portal → Monitoring → App Service Logs.

**Types**:
1. **Application Logging** (File System or Blob Storage)
   - Levels: Error, Warning, Information, Verbose.
2. **Web Server Logging** (IIS logs – HTTP transactions).
3. **Detailed Error Messages** (HTML error pages).
4. **Failed Request Tracing** (FREB – XML traces for failed requests, very detailed).

**Retention**: File System (limited), Blob (configure days).

**Access Logs**:
- Log Stream (real-time).
- Kudu (`https://<app>.scm.azurewebsites.net`): LogFiles folder, Process Explorer, Memory Dumps, CMD console.

#### C. **Diagnostic Settings (Azure Monitor Export)**
Send platform logs & metrics to destinations.

**Key Log Categories for Microsoft.Web/sites**:
- `AppServiceHTTPLogs`
- `AppServiceConsoleLogs`
- `AppServicePlatformLogs`
- `AppServiceFileAuditLogs`
- `AppServiceAuditLogs`
- `AppServiceIPsecLog`
- `AppServiceAntivirusScanAuditLog`
- Function-specific if using Functions.

**Metrics**: CPU, Memory, Requests, Data In/Out, HTTP Queue Length, etc.

**Destinations**: Log Analytics Workspace (recommended), Storage Account, Event Hub, Partner.

#### D. **Application Insights (Full APM)**
- **Auto-instrumentation** (codeless) or SDK/OpenTelemetry.
- Telemetry: Requests, Dependencies (SQL, HTTP, etc.), Exceptions, Traces, Page Views, Custom Events/Metrics.
- **Smart Detection**: Anomalies, performance degradation, failure anomalies.
- **Availability Tests** (web tests from multiple locations).
- **Profiler**, **Snapshot Debugger**, **Live Metrics**.
- **Application Map**, **Performance**, **Failures**, **Browser**, **Usage**.

**Best Practice**: Use OpenTelemetry for future-proofing.

#### E. **Other Monitoring Tools**
- **Azure Monitor Metrics** + **Alerts** (e.g., CPU > 80%, Failed Requests > 5).
- **Activity Log** (management operations).
- **Quotas & Limits** view.
- **Kudu Advanced Tools**.
- **Log Analytics Queries** (KQL) for deep analysis.

### 8. Security, Best Practices & Common Issues

**Security**:
- Use Managed Identity + Key Vault/App Configuration references.
- Mark sensitive settings as sticky.
- Enable Access Restrictions + Private Endpoints.
- Minimum TLS 1.2+.
- RBAC least privilege for management.

**Best Practices**:
- Run from package (`WEBSITE_RUN_FROM_PACKAGE=1`).
- Always On + local cache for performance.
- Separate settings per slot.
- Version configuration as code.
- Enable comprehensive logging + retention policies.
- Monitor HTTP Queue Length to catch scaling needs early.
- Use Azure App Configuration for large/complex apps.

**Common Issues & Fixes**:
- Settings not picked up → Restart app / check slot / case sensitivity.
- Cold starts → Always On + pre-warmed slots.
- Logs missing → Enable correctly + wait; check correct slot.
- High costs → Sample data, filter logs, shorter retention.
- Swap failures → Warm-up path, sticky settings, health checks.

This covers **every major and minor aspect** of App Service Configuration and Diagnostics based on current Microsoft documentation and capabilities. You now have production-enterprise level knowledge.

