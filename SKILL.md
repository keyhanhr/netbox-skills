---
name: netbox-asset-management
description: Handles physical infrastructure asset lookups, site server counts, IPAM queries, rack inventory, and spare parts using the official NetBox MCP Server. Use when the user asks about physical servers, device counts, spare parts/memory/disks, IP addresses, or racks in NetBox. Do NOT use for virtual machines, vCenter ESXi hosts, or hypervisor metrics.
---

# NetBox Infrastructure & Asset Management Skill

## Overview
This skill instructs NanoCortex on how to query NetBox using its MCP tools (`search_objects`, `get_device`, `get_ip_address`, etc.) for hardware, IPAM, and inventory queries.

---

## Critical Rules & Safeguards

### 1. Mandatory MCP Tool Usage
- NEVER invent API endpoints or external gateways (like OneCLI or raw HTTP curls). 
- Always execute queries using the available NetBox MCP tools in the environment.
- For general or natural language queries, use `search_objects`.

### 2. Token Optimization (`fields` parameter)
NetBox objects contain large JSON schemas. **ALWAYS** pass the `fields` parameter in MCP calls to fetch only required attributes (reduces token footprint significantly):
- **Devices:** `name`, `status`, `device_type`, `device_role`, `site`, `rack`, `position`, `primary_ip`
- **IP Addresses:** `address`, `status`, `assigned_object`, `description`
- **Inventory/Spares:** `device`, `name`, `part_id`, `manufacturer`, `description`, `site`

### 3. Spare Parts ("spare" Device Logic)
Spare hardware components (RAM, SFP modules, Disks) are registered under a dummy device named `spare` (or `spare-*`). When asked about spare/stock inventory:
1. Query items/modules attached to `spare` devices.
2. Filter by component model/description (e.g., `64GB`, `DDR5`, `SFP`).
3. Summarize counts per site.

### 4. Zero-Hallucination Policy
If an asset or IP is not found in NetBox, state clearly:
> *"تجهیز یا آدرس مورد نظر در سیستم NetBox یافت نشد."*

---

## Output Formatting
Format all responses for Mattermost using clean Markdown tables:

| نام دستگاه / قطعه | وضعیت | سایت / دیتاسنتر | مشخصات / آدرس IP |
| :--- | :--- | :--- | :--- |
| `{{name}}` | `{{status}}` | `{{site}}` | `{{details}}` |
