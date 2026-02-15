# devops-terraform-modules-nsx

<p align="center">
  <img src="https://img.shields.io/badge/VMware-NSX--T-607078?style=for-the-badge&logo=vmware&logoColor=white" alt="VMware NSX-T" />
  <img src="https://img.shields.io/badge/Terraform-%3E%3D1.5-7B42BC?style=for-the-badge&logo=terraform&logoColor=white" alt="Terraform" />
  <img src="https://img.shields.io/badge/Provider-%3E%3D3.4.0-844FBA?style=for-the-badge&logo=terraform&logoColor=white" alt="Provider Version" />
</p>

<h1 align="center">🌐 Terraform Module for VMware NSX-T</h1>

<p align="center">
  A comprehensive Terraform module for managing VMware NSX-T Data Center infrastructure using the <strong>Policy API</strong>.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/IaC-Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white" alt="IaC" />
  <img src="https://img.shields.io/badge/Platform-NSX--T_3.x_%7C_4.x-607078?style=flat-square&logo=vmware&logoColor=white" alt="Platform" />
  <img src="https://img.shields.io/badge/API-Policy_API-informational?style=flat-square" alt="API" />
  <img src="https://img.shields.io/badge/Status-Production_Ready-success?style=flat-square" alt="Status" />
</p>

---

## 📦 Architecture

```
terraform-nsx-module/
├── main.tf              # Root module - composes sub-modules
├── variables.tf         # Root input variables
├── outputs.tf           # Root outputs
├── README.md
├── modules/
│   ├── tier1_gateway/   # 🔀 Tier-1 gateway management
│   ├── segments/        # 🔗 Overlay segment management
│   ├── security_groups/ # 🛡️ NS Group / security group management
│   ├── firewall_rules/  # 🔥 Distributed Firewall (DFW) policy & rules
│   └── load_balancer/   # ⚖️ LB service, pools, monitors, virtual servers
└── examples/
    └── three-tier-app.tf  # 📋 Full 3-tier application example
```

---

## 🧩 Resources Managed

| | Sub-Module | NSX-T Resources |
|---|---|---|
| 🔀 | **tier1_gateway** | `nsxt_policy_tier1_gateway` |
| 🔗 | **segments** | `nsxt_policy_segment` with optional DHCP |
| 🛡️ | **security_groups** | `nsxt_policy_group` (tag, IP, segment criteria) |
| 🔥 | **firewall_rules** | `nsxt_policy_security_policy` with dynamic rules |
| ⚖️ | **load_balancer** | `nsxt_policy_lb_service`, `nsxt_policy_lb_pool`, `nsxt_policy_lb_monitor`, `nsxt_policy_lb_virtual_server` |

---

## ✅ Prerequisites

| Requirement | Version / Detail |
|---|---|
| ![NSX-T](https://img.shields.io/badge/NSX--T-3.x_%7C_4.x-607078?style=flat-square&logo=vmware&logoColor=white) | NSX-T Data Center 3.x or 4.x |
| ![Terraform](https://img.shields.io/badge/Terraform-%3E%3D1.5.0-7B42BC?style=flat-square&logo=terraform&logoColor=white) | Terraform >= 1.5.0 |
| ![Provider](https://img.shields.io/badge/nsxt_provider-%3E%3D3.4.0-844FBA?style=flat-square&logo=terraform&logoColor=white) | VMware NSX-T Terraform Provider >= 3.4.0 |

**Infrastructure required:**
- 🔀 Existing Tier-0 gateway
- 🖥️ Existing Edge Cluster
- 🌐 Existing Overlay Transport Zone

---

## 🚀 Quick Start

```hcl
module "nsx" {
  source = "./terraform-nsx-module"

  nsx_manager_host            = "nsx-manager.example.com"
  nsx_username                = var.nsx_username
  nsx_password                = var.nsx_password
  overlay_transport_zone_name = "overlay-tz"
  tier0_gateway_name          = "tier0-gateway"
  edge_cluster_name           = "edge-cluster-01"

  tier1_gateways = {
    "t1-web" = {
      description = "Web Tier Gateway"
    }
  }

  segments = {
    "seg-web" = {
      tier1_gateway = "t1-web"
      subnet_cidr   = "10.10.1.1/24"
    }
  }

  security_groups = {
    "sg-web-servers" = {
      criteria = [
        { condition_type = "Tag", tag = "web", scope = "tier" }
      ]
    }
  }

  firewall_policies = {
    "policy-web" = {
      category = "Application"
      rules = [
        {
          display_name       = "allow-http"
          action             = "ALLOW"
          destination_groups = ["sg-web-servers"]
          services           = ["HTTP", "HTTPS"]
          logged             = true
        }
      ]
    }
  }
}
```

---

## 📥 Input Variables

| Variable | Type | Required | Description |
|---|---|---|---|
| `nsx_manager_host` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | NSX Manager FQDN or IP |
| `nsx_username` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | NSX Manager username |
| `nsx_password` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | NSX Manager password |
| `overlay_transport_zone_name` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | Overlay transport zone name |
| `tier0_gateway_name` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | Existing Tier-0 gateway name |
| `edge_cluster_name` | ![string](https://img.shields.io/badge/-string-blue?style=flat-square) | ![required](https://img.shields.io/badge/-required-critical?style=flat-square) | Edge cluster name |
| `tier1_gateways` | ![map(any)](https://img.shields.io/badge/-map(any)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | Tier-1 gateways to create |
| `segments` | ![map(any)](https://img.shields.io/badge/-map(any)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | Overlay segments to create |
| `security_groups` | ![map(any)](https://img.shields.io/badge/-map(any)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | Security groups to create |
| `firewall_policies` | ![map(any)](https://img.shields.io/badge/-map(any)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | DFW policies with rules |
| `load_balancers` | ![map(any)](https://img.shields.io/badge/-map(any)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | Load balancer configurations |
| `default_tags` | ![map(string)](https://img.shields.io/badge/-map(string)-blueviolet?style=flat-square) | ![optional](https://img.shields.io/badge/-optional-green?style=flat-square) | Default tags for all resources |

---

## 📤 Outputs

| Output | Description |
|---|---|
| `tier1_gateways` | 🔀 Map of Tier-1 gateway paths and IDs |
| `segments` | 🔗 Map of segment paths and IDs |
| `security_groups` | 🛡️ Map of security group paths and IDs |
| `firewall_policies` | 🔥 Map of firewall policy paths |
| `load_balancers` | ⚖️ Map of LB service IDs and virtual servers |

---

## 🛡️ Security Group Criteria Types

The module supports three types of group membership criteria:

### 🏷️ Tag-based (most common for micro-segmentation)
```hcl
{ condition_type = "Tag", tag = "web", scope = "tier", member_type = "VirtualMachine" }
```

### 🌍 IP Address-based
```hcl
{ condition_type = "IPAddress", ip_addresses = ["10.0.0.0/24", "10.0.1.0/24"] }
```

### 🔗 Segment-based
```hcl
{ condition_type = "Segment", tag = "production" }
```

---

## 🔥 Firewall Rule Services

Rules reference NSX built-in services by display name. The module automatically resolves these to their policy paths.

| Service Name | Protocol |
|---|---|
| `"HTTP"` | TCP/80 |
| `"HTTPS"` | TCP/443 |
| `"SSH"` | TCP/22 |
| `"DNS"` | TCP+UDP/53 |
| `"ICMP ALL"` | ICMP |

> 💡 **Tip:** Any service defined in NSX Manager can be referenced by its display name — these are just common examples.

---

## 🏗️ Example: 3-Tier Application

See [`examples/three-tier-app.tf`](examples/three-tier-app.tf) for a complete deployment that creates:

- 🔀 2 Tier-1 gateways (web & app tiers)
- 🔗 3 overlay segments (web, app, db)
- 🛡️ 3 security groups with tag-based criteria
- 🔥 DFW micro-segmentation policies
- ⚖️ Web tier load balancer with health monitoring

---

## 📄 License

![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=for-the-badge)

This module is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
