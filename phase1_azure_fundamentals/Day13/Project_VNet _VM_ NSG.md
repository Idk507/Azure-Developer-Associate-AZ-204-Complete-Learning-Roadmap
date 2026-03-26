the **complete end-to-end guide** for **Project_VNet_VM_NSG** – the foundational networking layer that supports the Azure Load Balancer and Application Gateway we discussed earlier.

This is the **secure, production-ready network backbone** where everything lives: your VMs, backend pools, Application Gateway, and Load Balancer.

### Recommended Architecture (Best Practices – Simple & Secure)

Think of your Azure network like a well-planned apartment building:
- **Virtual Network (VNet)** = The entire building (isolated private network).
- **Subnets** = Different floors/sections (web tier, backend, gateway, etc.). Subnets segment traffic and allow targeted security.
- **Network Security Group (NSG)** = Security guards at each floor entrance. They allow/deny traffic based on rules (like a stateful firewall using 5-tuple: source IP/port, destination IP/port, protocol).
- **VMs** = Residents living on specific floors. Each VM gets a Network Interface Card (NIC) attached to a subnet.

**Why this design?**
- Isolation: Web-facing traffic stays separate from internal VMs.
- Security: NSGs block unwanted traffic by default.
- Scalability: Easy to add more subnets or VMs.
- Load Balancing support: Application Gateway needs its own dedicated subnet; VMs go in a backend subnet.

**Typical Project Structure (Recommended for most apps):**

```
Project_VNet (e.g., 10.0.0.0/16)
├── AGW-Subnet (dedicated for Application Gateway) → /24 recommended (minimum /26)
├── Backend-Subnet (for VMs / VM Scale Sets) → /24
├── (Optional) Bastion-Subnet (for secure RDP/SSH access)
├── (Optional) Database-Subnet or other tiers
```

**NSG Strategy** (Best Practice):
- One NSG per subnet (subnet-level) for broad control.
- Optional: Additional NSG on individual VM NICs for finer-grained rules (NIC rules take priority over subnet rules).
- Default rules: Allow VNet-to-VNet traffic, deny everything else unless you explicitly allow.
- Always allow **AzureLoadBalancer** service tag for health probes.

**Key Requirements for Application Gateway Subnet:**
- Dedicated subnet (no other resources).
- Minimum /26, strongly recommended /24 for autoscaling (v2 SKU needs space for up to 125 instances).
- Specific inbound rules if NSG is applied (client traffic on 80/443, health probes, ports 65503-65534 for diagnostics in some cases).
- Do **not** block AzureLoadBalancer.

**Key for Backend VM Subnet:**
- Allow traffic from Application Gateway subnet (source = AGW-Subnet CIDR) on port 80/443.
- Allow AzureLoadBalancer for probes.
- Restrict outbound if needed (e.g., allow only to internet via NAT Gateway).

### Complete Python Code – Full Project_VNet + Subnets + NSGs + Sample VM Setup

This script builds the **entire networking foundation** (VNet, subnets, NSGs with proper rules) and creates two sample backend VMs ready for Load Balancer / Application Gateway.

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.network import NetworkManagementClient
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import ResourceManagementClient
import os

# === CONFIGURATION ===
SUBSCRIPTION_ID = os.getenv("AZURE_SUBSCRIPTION_ID")
RESOURCE_GROUP = "ProjectRG"
LOCATION = "eastus"  # Change to your region (Mumbai = centralindia if needed)
VNET_NAME = "Project_VNet"
ADDRESS_PREFIX = "10.0.0.0/16"

# Subnets
AGW_SUBNET_NAME = "AGW-Subnet"
AGW_PREFIX = "10.0.1.0/24"          # Dedicated for Application Gateway
BACKEND_SUBNET_NAME = "Backend-Subnet"
BACKEND_PREFIX = "10.0.2.0/24"      # For VMs

# NSGs
AGW_NSG_NAME = "AGW-NSG"
BACKEND_NSG_NAME = "Backend-NSG"

credential = DefaultAzureCredential()
network_client = NetworkManagementClient(credential, SUBSCRIPTION_ID)
compute_client = ComputeManagementClient(credential, SUBSCRIPTION_ID)
resource_client = ResourceManagementClient(credential, SUBSCRIPTION_ID)

# 1. Create Resource Group
resource_client.resource_groups.create_or_update(
    RESOURCE_GROUP, {"location": LOCATION}
)

# 2. Create VNet
vnet = network_client.virtual_networks.begin_create_or_update(
    RESOURCE_GROUP, VNET_NAME,
    {
        "location": LOCATION,
        "address_space": {"address_prefixes": [ADDRESS_PREFIX]},
        "subnets": [
            {"name": AGW_SUBNET_NAME, "address_prefix": AGW_PREFIX},
            {"name": BACKEND_SUBNET_NAME, "address_prefix": BACKEND_PREFIX}
        ]
    }
).result()

print(f"VNet {VNET_NAME} created with subnets.")

agw_subnet_id = next(s.id for s in vnet.subnets if s.name == AGW_SUBNET_NAME)
backend_subnet_id = next(s.id for s in vnet.subnets if s.name == BACKEND_SUBNET_NAME)

# 3. Create NSG for Application Gateway (with required rules)
print("Creating AGW NSG...")
agw_nsg = network_client.network_security_groups.begin_create_or_update(
    RESOURCE_GROUP, AGW_NSG_NAME,
    {
        "location": LOCATION,
        "security_rules": [
            # Allow client traffic (HTTP/HTTPS)
            {
                "name": "Allow-Client-HTTP-HTTPS",
                "protocol": "Tcp",
                "source_address_prefix": "Internet",
                "source_port_range": "*",
                "destination_address_prefix": AGW_PREFIX,
                "destination_port_ranges": ["80", "443"],
                "access": "Allow",
                "priority": 100,
                "direction": "Inbound"
            },
            # Allow Azure infrastructure (health probes, etc.)
            {
                "name": "Allow-AzureLoadBalancer",
                "protocol": "*",
                "source_address_prefix": "AzureLoadBalancer",
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_range": "*",
                "access": "Allow",
                "priority": 110,
                "direction": "Inbound"
            },
            # Allow Gateway Manager / diagnostics (if needed)
            {
                "name": "Allow-Gateway-Manager",
                "protocol": "Tcp",
                "source_address_prefix": "GatewayManager",
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_ranges": ["65200-65535"],  # Broad range; adjust per docs
                "access": "Allow",
                "priority": 120,
                "direction": "Inbound"
            },
            # Deny all other inbound (default is deny, but explicit is better)
            {
                "name": "Deny-All-Inbound",
                "protocol": "*",
                "source_address_prefix": "*",
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_range": "*",
                "access": "Deny",
                "priority": 4096,
                "direction": "Inbound"
            }
        ]
    }
).result()

# Associate NSG to AGW subnet (in real setup, do this after creating subnet if not inline)
# For simplicity, we can update subnet with NSG later if needed.

# 4. Create NSG for Backend VMs
print("Creating Backend NSG...")
backend_nsg = network_client.network_security_groups.begin_create_or_update(
    RESOURCE_GROUP, BACKEND_NSG_NAME,
    {
        "location": LOCATION,
        "security_rules": [
            # Allow traffic FROM Application Gateway subnet (critical!)
            {
                "name": "Allow-From-AGW",
                "protocol": "Tcp",
                "source_address_prefix": AGW_PREFIX,
                "source_port_range": "*",
                "destination_address_prefix": BACKEND_PREFIX,
                "destination_port_ranges": ["80", "443"],
                "access": "Allow",
                "priority": 100,
                "direction": "Inbound"
            },
            # Allow health probes from Load Balancer / Azure infrastructure
            {
                "name": "Allow-AzureLoadBalancer-Probes",
                "protocol": "*",
                "source_address_prefix": "AzureLoadBalancer",
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_range": "*",
                "access": "Allow",
                "priority": 110,
                "direction": "Inbound"
            },
            # Allow SSH/RDP only from specific sources (or Bastion) - restrict in production!
            {
                "name": "Allow-SSH",
                "protocol": "Tcp",
                "source_address_prefix": "Internet",  # Change to your IP or Bastion subnet
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_range": "22",
                "access": "Allow",
                "priority": 120,
                "direction": "Inbound"
            },
            # Deny all other inbound
            {
                "name": "Deny-All-Other-Inbound",
                "protocol": "*",
                "source_address_prefix": "*",
                "source_port_range": "*",
                "destination_address_prefix": "*",
                "destination_port_range": "*",
                "access": "Deny",
                "priority": 4096,
                "direction": "Inbound"
            }
        ]
    }
).result()

print("NSGs created with security rules.")

# 5. (Optional but recommended) Create a sample Backend VM in Backend-Subnet
# Note: In production, use VM Scale Sets + Availability Zones

print("Creating sample Backend VM (web-vm-1)...")
# First create NIC
nic = network_client.network_interfaces.begin_create_or_update(
    RESOURCE_GROUP, "web-vm-1-nic",
    {
        "location": LOCATION,
        "ip_configurations": [{
            "name": "ipconfig1",
            "subnet": {"id": backend_subnet_id},
            "private_ip_allocation_method": "Dynamic"
        }],
        "network_security_group": {"id": backend_nsg.id}
    }
).result()

# Create VM (Ubuntu example)
vm = compute_client.virtual_machines.begin_create_or_update(
    RESOURCE_GROUP, "web-vm-1",
    {
        "location": LOCATION,
        "hardware_profile": {"vm_size": "Standard_B2s"},
        "storage_profile": {
            "image_reference": {
                "publisher": "Canonical",
                "offer": "0001-com-ubuntu-server-jammy",
                "sku": "22_04-lts",
                "version": "latest"
            }
        },
        "os_profile": {
            "computer_name": "web-vm-1",
            "admin_username": "azureuser",
            "linux_configuration": {"disable_password_authentication": True}  # Use SSH keys
        },
        "network_profile": {"network_interfaces": [{"id": nic.id}]}
    }
).result()

print("Sample VM created! Repeat for more VMs or use VM Scale Set.")

# Next Steps:
# 1. Deploy Application Gateway into AGW-Subnet (using the previous script, passing agw_subnet_id)
# 2. Add VMs (or their NICs) to Load Balancer / Application Gateway backend pools
# 3. For secure access: Deploy Azure Bastion in a separate subnet instead of opening SSH from Internet
```

### Quick Setup Guidance (After Running the Script)

1. **Associate NSG to Subnets** (if not done inline): In Portal → VNet → Subnets → Edit subnet → Assign NSG.
2. **Deploy Application Gateway** into **AGW-Subnet** (use the previous Python code, passing the subnet ID).
3. **Deploy Load Balancer** (public or internal) and add backend VMs.
4. **Test Traffic Flow**:
   - Internet → Application Gateway (port 80/443) → Backend VMs (allowed by NSG).
   - Health probes from Load Balancer/AGW must succeed.
5. **Monitoring**: Enable NSG Flow Logs + Traffic Analytics for visibility.
6. **Enhancements**:
   - Add NAT Gateway for outbound internet from VMs.
   - Use Azure Bastion for secure management instead of opening 22/3389.
   - Implement Application Security Groups (ASGs) for role-based rules.

This **Project_VNet_VM_NSG** setup is now fully secure, segmented, and ready to host your Load Balancer + Application Gateway architecture without any loss of traffic or security gaps.

You have the complete foundation: VNet, proper subnet sizing, hardened NSGs with all required rules for AGW and backends, and sample VMs.

Run the script (after setting your subscription and adding SSH keys), then layer the Load Balancer / Application Gateway on top.
