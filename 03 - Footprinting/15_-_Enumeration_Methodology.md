# Enumeration Methodology

This methodology is structured as **6 layers** — metaphorical boundaries you work to pass during enumeration. The layers group into three levels:

| Infrastructure-based enumeration | Host-based enumeration | OS-based enumeration |
| --- | --- | --- |

> **Note:** The components listed for each layer are the main categories, not an exhaustive list. Layers 1 and 2 (Internet Presence, Gateway) don't fully apply to intranet environments like Active Directory infrastructure.

## The Six Layers

| Layer | Goal | Information Categories |
| --- | --- | --- |
| **1. Internet Presence** | Identify internet presence and externally accessible infrastructure. | Domains, subdomains, vHosts, ASN, netblocks, IP addresses, cloud instances, security measures |
| **2. Gateway** | Identify the security measures protecting external and internal infrastructure. | Firewalls, DMZ, IPS/IDS, EDR, proxies, NAC, network segmentation, VPN, Cloudflare |
| **3. Accessible Services** | Identify accessible interfaces and services, hosted externally or internally. | Service type, functionality, configuration, port, version, interface |
| **4. Processes** | Identify internal processes, sources, and destinations tied to the services. | PID, processed data, tasks, source, destination |
| **5. Privileges** | Identify internal permissions and privileges for the accessible services. | Groups, users, permissions, restrictions, environment |
| **6. OS Setup** | Identify internal components and how the system is set up. | OS type, patch level, network config, OS environment, config files, sensitive private files |

## Layer-by-Layer Goals

**Layer 1 — Internet Presence:** Identify all possible target systems and interfaces that can be tested. This is your reconnaissance footprint.

**Layer 2 — Gateway:** Understand what you're dealing with and what defensive measures you need to watch out for.

**Layer 3 — Accessible Services:** Understand the purpose and functionality of each target system, and gain the knowledge needed to communicate with it and exploit it effectively. *This is where most of the service-specific footprinting (FTP, SMB, databases, etc.) happens.*

**Layer 4 — Processes:** Every executed command or function processes data, starting a process with at least one source and one target. The goal is to understand these factors and identify the dependencies between them.

**Layer 5 — Privileges:** Each service runs as a specific user, in a particular group, with admin- or system-defined permissions. These privileges often expose functions admins overlook — especially in Active Directory and multi-role administration environments. Identify them and understand what they do and don't allow.

**Layer 6 — OS Setup:** Understand how admins manage the systems and what sensitive internal information you can glean from them.

> **How to use this:** Treat the layers as a checklist that moves you from the outside in. You rarely jump straight to exploitation — each layer builds the context that makes the next one productive. A credential found at Layer 3 (a service) becomes useful at Layer 5 (privileges) and may reveal Layer 6 (sensitive files) information.
