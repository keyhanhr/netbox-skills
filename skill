---
name: netbox-asset-management
description: Comprehensive skill for querying NetBox infrastructure assets including physical devices, hardware components (RAM, disks, SFPs, interfaces), sites/datacenters, device models (device types), IPAM/subnets, racks, and spare parts. IMPORTANT: Spare components in NetBox are explicitly assigned to a placeholder device named 'spare' (or starting with 'spare-'). Always query inventory items on 'spare' devices for stock counts. Use when users ask about server specs, site summaries, device types, hardware components, spare parts, IP allocations, or rack space in NetBox. Do NOT use for virtual machines, vSphere/vCenter hypervisors, or cloud instances.
---

# NetBox Infrastructure & Asset Management Skill

## Overview
This skill guides NanoCortex on executing precise, token-optimized queries against NetBox using its official MCP tools (`search_objects`, `get_objects`, `get_object_by_id`) and rendering clean, structured Markdown reports for Mattermost.

---

## Critical Rules & Performance Safeguards

### 1. Mandatory MCP Tool Usage
- ALWAYS use the available NetBox MCP tools (`search_objects`, `get_objects`, etc.).
- NEVER generate raw HTTP commands (`curl`), shell scripts, or invent fake API gateways.

### 2. CRITICAL: Spare Parts Lookup Logic ("spare" Device) ⚠️
- **Core Convention:** In NetBox, all spare/stock hardware components (such as RAM modules, SFP optical transceivers, disks, power supplies, expansion cards) are **assigned to a dedicated placeholder device named `spare` (or `spare-*` like `spare-tehran-dc`)**.
- **Execution Workflow for Spare Queries:**
  When a user asks about spare parts, stock inventory, or available spare components (e.g., *"چند تا رم یا SFP اسپر داریم؟"*):
  1. **Direct Target:** Go directly to `dcim/inventory-items` attached to the device named `spare` (or devices whose name starts with `spare`).
  2. **Filter & Count:** Filter inventory items by `device=spare` (or `device__icontains=spare`) and match the component model, `part_id`, or description requested.
  3. **Aggregate:** Count the exact number of matching items assigned to that `spare` device.
  4. **Report:** Display the stock count per site/datacenter based on which `spare` device host they are assigned to.

### 3. Strict Token & Context Optimization (`fields` Projection)
NetBox API responses are extremely dense. To prevent context window saturation and slow response times, **ALWAYS** pass the `fields` parameter in MCP query calls to project only the relevant attributes for the query type:

- **Devices (`dcim/devices`):**  
  `id,name,status,device_type,device_role,site,rack,position,primary_ip,serial,asset_tag`
- **Device Types / Models (`dcim/device-types`):**  
  `id,model,manufacturer,part_number,u_height,is_full_depth,subdevice_role`
- **Sites / Datacenters (`dcim/sites`):**  
  `id,name,slug,status,facility,region,tenant,physical_address,description`
- **Inventory Items / Components (`dcim/inventory-items`):**  
  `id,device,name,label,manufacturer,part_id,serial,asset_tag,discovered,description`
- **Interfaces (`dcim/interfaces`):**  
  `id,device,name,type,enabled,mgmt_only,mtu,mac_address,wwn,mode`
- **IPAM / IP Addresses (`ipam/ip-addresses`):**  
  `id,address,status,role,assigned_object,dns_name,description,tenant`
- **Racks (`dcim/racks`):**  
  `id,name,site,location,status,u_height,type,facility_id`

### 4. Zero-Hallucination Guardrail
If a device, site, component, or IP address is not found in NetBox, output clearly in Persian:
> *"تجهیز، سایت یا آدرس مورد نظر در سیستم NetBox یافت نشد."*

---

## Domain Query Logic & Rules

### A. Devices & Servers
- Differentiate between `device_type` (hardware model e.g., Cisco C220 M5) and `device_role` (function e.g., Compute, Core Switch, Firewall).
- Always include primary IPv4/IPv6 if available.

### B. Device Models & Types (`device-types`)
- When asked about model specifications, query `dcim/device-types`.
- Show manufacturer, exact model name, U-height, and part number.

### C. Sites & Datacenters (`sites`)
- When asked for site summaries, count total active devices, associated subnets, and physical address/facility name.

### D. Hardware Components & Spares (`inventory-items` & `spare` logic)
- Components include RAM modules, SFPs, Power Supplies, Expansion Cards, and Disks.
- **Rule for Spares:** Remember that all spare hardware is attached to `spare` or `spare-*` devices. Never search production servers when counting stock/spare parts.
- When queried about stock or spare parts:
  1. Filter `inventory-items` attached to devices named `spare` / `spare-*`.
  2. Group items by `manufacturer`, `part_id`, and `description`.
  3. Calculate total available counts per site.

### E. IPAM & Subnets (`ip-addresses` & `prefixes`)
- Format IP addresses cleanly (e.g., `10.10.20.5/24`).
- Display assignment status (`Active`, `Reserved`, `Deprecated`) and assigned device name.

---

## Gotchas & Common Traps
- **Spare Search Fallback:** If searching for `device=spare` yields no results, try searching with `device__icontains=spare` to catch site-specific spare devices (e.g., `spare-thr-01`).
- **Filter Precision:** NetBox API filters are case-sensitive. Use lower-case slugs when filtering by site or role (e.g., `site=tehran-dc1`).
- **Missing Primary IP:** `primary_ip` is an object. Extract its `address` property (e.g., `device.primary_ip.address`).
- **Large Result Sets:** If a query returns > 25 objects, state the total count and present the top 20 items with a note that results were truncated.

---

## Mattermost Output Formatting Templates

Always format responses using clean, aligned Markdown tables with Persian column headers. Choose the appropriate template below based on the user's query:

### Template 1: Device / Server Summary (خلاصه دستگاه‌ها و سرورها)
| نام دستگاه | وضعیت | سازنده و مدل | نقش | سایت / دیتاسنتر | آدرس IP اصلی |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `{{name}}` | `{{status}}` | `{{manufacturer}} {{model}}` | `{{device_role}}` | `{{site}}` | `{{primary_ip}}` |

### Template 2: Device Models & Specs (مشخصات مدل‌های سخت‌افزاری)
| سازنده | نام مدل | Part Number | ارتفاع (RU) | عمق کامل؟ |
| :--- | :--- | :--- | :--- | :--- |
| `{{manufacturer}}` | `{{model}}` | `{{part_number}}` | `{{u_height}} U` | `{{is_full_depth}}` |

### Template 3: Hardware Components & Spares (قطعات سخت‌افزاری و قطعات یدکی)
| نام قطعه / عنوان | دستگاه / انبار | سازنده | Part ID / مدل | شماره سریال | سایت |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `{{name}}` | `{{device}}` | `{{manufacturer}}` | `{{part_id}}` | `{{serial}}` | `{{site}}` |

### Template 4: IPAM & Subnets (آدرس‌های IP و شبکه‌ها)
| آدرس IP | وضعیت | نقش | تخصیص یافته به | نام DNS | توضیحات |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `{{address}}` | `{{status}}` | `{{role}}` | `{{assigned_object}}` | `{{dns_name}}` | `{{description}}` |

### Template 5: Datacenter Sites Overview (خلاصه سایت‌ها و دیتاسنترها)
| نام سایت | کد (Slug) | وضعیت | مرکز/Facility | منطقه | آدرس فیزیکی |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `{{name}}` | `{{slug}}` | `{{status}}` | `{{facility}}` | `{{region}}` | `{{physical_address}}` |
