**Azure App Service Deployment Slots – Everything in Extreme Detail**  
(Combined with CI/CD, Blue-Green, Troubleshooting, and Full Integration with Application Gateway + VNet)

### 1. CI/CD Integration with GitHub Actions + Deployment Slots (Production-Grade)

**Why this combination is powerful**: GitHub Actions builds/tests your code automatically. Deployment slots let you deploy the new version safely to a non-production slot, run automated tests, warm up the app, and then swap with **zero downtime**.

**Recommended Workflow Pattern** (Push to `main` → Staging → Swap):
- On push to `main`: Build → Test → Deploy to `staging` slot → (Optional: Run smoke tests) → Swap to production.
- Use **OpenID Connect** (recommended, no secrets) or Publish Profile.

#### Complete GitHub Actions Workflow YAML (`.github/workflows/deploy.yml`)

```yaml
name: Deploy to Azure App Service with Slots

on:
  push:
    branches: [ main ]
  workflow_dispatch:  # Manual trigger

env:
  AZURE_WEBAPP_NAME: "myprojectwebapp"      # Your App Service name
  AZURE_WEBAPP_SLOT: "staging"              # Target slot
  RESOURCE_GROUP: "ProjectRG"

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python (example for Python app)
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest  # or your test command

      - name: Zip artifact
        run: |
          zip -r app.zip . -x '*.git*'

      - name: Azure Login (OIDC - Recommended)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      # Optional: Use Azure CLI for advanced steps
      - name: Deploy to Staging Slot
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          slot-name: ${{ env.AZURE_WEBAPP_SLOT }}   # Critical: deploys to staging
          package: app.zip

      - name: Run Smoke Tests on Staging
        run: |
          # Example: curl health check
          curl -f https://${{ env.AZURE_WEBAPP_NAME }}-${{ env.AZURE_WEBAPP_SLOT }}.azurewebsites.net/health || exit 1

      - name: Swap Slots (Staging → Production)
        uses: azure/cli@v2
        with:
          inlineScript: |
            az webapp deployment slot swap \
              --resource-group ${{ env.RESOURCE_GROUP }} \
              --name ${{ env.AZURE_WEBAPP_NAME }} \
              --slot ${{ env.AZURE_WEBAPP_SLOT }} \
              --target-slot production \
              --verbose
```

**Key Points**:
- Always specify `slot-name` in the deploy action.
- For **Auto Swap**: In the portal, go to the staging slot → Configuration → General settings → Enable Auto Swap (target = production). Every successful deploy to staging will auto-swap.
- Use **slot-specific settings** for environment differences (e.g., DB connections).
- Grant the identity **Website Contributor** role on both the app and the slot.

**PR-specific slots** (advanced): Create dynamic slots per PR for preview environments (see Azure Samples repo).

### 2. Blue-Green Deployment Full Workflow (Using Slots)

**Blue-Green Concept**: Two identical environments.  
- **Blue** = Current live (Production slot).  
- **Green** = New version (Staging slot).

**End-to-End Workflow** (Zero Downtime + Rollback):

1. **Prepare Slots**:
   - Production slot = Blue (live traffic).
   - Staging slot = Green (new code).

2. **Deploy New Version**:
   - Build & deploy to **staging** slot only (via GitHub Actions above).
   - Warm-up happens automatically during swap prep.

3. **Validation**:
   - Manual: Browse `https://myapp-staging.azurewebsites.net`.
   - Automated: Smoke/load tests, synthetic monitoring.
   - Optional: Traffic splitting (10-20% to staging for canary testing) via portal → Deployment slots → Traffic %.

4. **Swap** (Instant cutover):
   - Azure warms up Green slot with Blue’s settings.
   - Swaps routing pointers.
   - Old Blue becomes the new staging (ready for next release).

5. **Post-Swap**:
   - Monitor metrics (CPU, errors, latency) for 15-60 mins.
   - If issues: **Swap back immediately** (rollback in seconds).

6. **Repeat**: Next release uses the current staging as Green.

**Slot-Specific (Sticky) Settings** (Critical for Blue-Green):
- Mark as **Deployment slot setting** (checkbox in portal or via API): Connection strings, API keys, environment flags, `WEBSITE_` settings that differ.
- These **never swap**.
- Non-sticky settings (code, most app settings) **do swap**.

**Best Practices**:
- Keep slots as identical as possible except for code and sticky settings.
- Use feature flags for risky changes.
- Keep old slot for at least 24 hours.
- Enable local cache + always-on for faster warm-up.

### 3. Troubleshooting Common Swap Issues (Very Detailed)

Swap failures are common during initial setup. Here are the **most frequent issues + fixes**:

| Issue | Symptoms | Root Cause | Fix |
|-------|----------|------------|-----|
| **HTTP Ping Failure** | "Did not respond to http ping" | App doesn't respond on root `/` or takes >90s | Set `WEBSITE_SWAP_WARMUP_PING_PATH=/health` (custom health endpoint). Ensure app starts fast. |
| **Timeout / Hanging** | Swap takes forever or times out | Slow startup, large app, missing warm-up | Enable Always On, use local cache, optimize startup (e.g., precompile .NET). |
| **Configuration Mismatch** | Swap fails on settings | Incompatible sticky vs swapped settings | Review slot-specific settings. Use `WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS=0` if needed. |
| **Auto-Swap Conflicts** | Unexpected behavior | Auto-swap enabled + manual actions | Auto-swap only works on direct deploy to the slot with auto-swap enabled. |
| **Local Cache Issues** | Swap fails after enabling cache | Quota exceeded or misconfig | Adjust `WEBSITE_LOCAL_CACHE_OPTION=Never` or ensure enough space. |
| **VNet / Networking** | Post-swap connectivity breaks | VNet Integration not consistent | Ensure VNet integration is configured identically or as sticky. |
| **Custom Domains / SSL** | Bindings missing after swap | Bindings are slot-specific | Re-apply or use managed certificates properly. |
| **Database / External Connections** | New version fails after swap | Wrong connection strings | Make DB settings **sticky** per slot. |

**Debugging Steps**:
- Check **Activity Log** + **Diagnose and Solve Problems** in portal.
- Stream logs from both slots during swap.
- Look at `D:\home\LogFiles\eventlog.xml`.
- Use Kudu (`https://myapp.scm.azurewebsites.net`) to inspect each slot.

**Prevention**: Always test swap in lower environments first. Use health checks and monitoring.

### 4. Combining Everything with Application Gateway + Project_VNet (End-to-End Secure Architecture)

**Recommended Production Architecture**:
- Internet → **Application Gateway v2 (WAF enabled)** (in AGW-Subnet) → **App Service** (via FQDN or Private Link).
- App Service → **Regional VNet Integration** (to Backend-Subnet or dedicated integration subnet in **Project_VNet**) for outbound to VMs, Azure SQL (private), etc.
- **Access Restrictions** on App Service: Allow **only** traffic from Application Gateway subnet → Block direct public access.

**Why this is secure & scalable**:
- WAF protects against OWASP threats.
- No direct public exposure of App Service.
- Slots work seamlessly (swap keeps the backend target the same for AGW).

**Configuration Details**:

1. **App Service Side**:
   - Enable VNet Integration (dedicated `/27+` subnet with delegation to `Microsoft.Web/serverFarms`).
   - Add Access Restriction rule: Allow source = AGW subnet CIDR, priority 100. Deny all else (or use Service Tag).

2. **Application Gateway Side**:
   - Backend Pool target: Use the App Service’s `.azurewebsites.net` FQDN (not IP).
   - HTTP Settings: Pick hostname from backend address, enable cookie affinity if needed.
   - Health Probe: Path `/health` or root, match status 200.
   - Routing: Basic or path-based.
   - WAF: Enabled in Prevention mode.

3. **During Slot Swap**:
   - The FQDN remains the same → AGW continues routing without changes.
   - Test new version via staging hostname or temporary routing.

**Python Code Snippet** (Add Access Restriction):

```python
# After creating web app
web_client.web_apps.begin_create_or_update_access_restriction(
    RESOURCE_GROUP, WEB_APP_NAME, 
    # Rule to allow only AGW subnet
    {
        "name": "AllowFromAGW",
        "action": "Allow",
        "priority": 100,
        "ip_address": "10.0.1.0/24",  # AGW subnet
        "description": "Allow only from Application Gateway"
    }
)
```

**Full Flow**:
- Users hit custom domain → AGW (WAF) → App Service Production slot.
- Deploy new version to staging slot → Test via staging URL or % traffic.
- Swap → New version instantly live behind same AGW.

This gives you **enterprise-grade** zero-downtime, secure, observable deployments.


Need the full GitHub repo structure, Terraform version, or specific language (e.g., .NET/Java) examples? Let me know!
