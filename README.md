# Node2Shell exposure dashboard for Dynatrace

**CVE-2026-111497 (aka HTTP2Shell) — Node.js version control across Kubernetes and Compute** Provided by **PacketPerf** 

---

## What it is

This is a Dynatrace dashboard you can import as is. It answers three questions about Node2Shell, a flaw fixed on 21 October 2026:

1. **Are we exposed?**
2. **Where?** Which Kubernetes clusters, namespaces and workloads, and which servers.
3. **What is left to fix?** And is the remediation moving forward?

It uses only data Dynatrace already collects: the Smartscape topology and OneAgent process detection. You don't need an extra agent or an extension, and no data leaves your tenant.

## Prerequisites

- Dynatrace SaaS with Grail: the Dashboards app and DQL.
- OneAgent on the hosts and Kubernetes nodes that run Node.js. Processes without OneAgent do not show up.
- A user allowed to create dashboards and run DQL queries on Smartscape. A standard environment user is enough.
- For the Kubernetes columns (cluster, workload, image): Kubernetes API monitoring connected on the cluster. Without it, containers show up as "unknown cluster" or "unknown workload".

## Import

1. Download [`node2shell-cve-2026.upload.json`](node2shell-cve-2026.upload.json).
2. In Dynatrace, open the **Dashboards** app, click **Upload** and select the file.
3. Four variables appear at the top of the dashboard:

| Variable | Purpose |
|---|---|
| **Cluster** | Kubernetes clusters in scope (all by default) |
| **Namespace** | Namespaces in scope, limited to the selected clusters |
| **Host** | Classic servers (VMs, bare metal, containers outside Kubernetes) |
| **Mode** | *Whole timeframe* (default) includes processes that have since stopped. *Running now* keeps only processes seen in the last 15 minutes |

> `node2shell-cve-2026.dashboard.json` is the same dashboard inside a `{name, type, content}` wrapper, for `dtctl` or the API. Do not use it with the **Upload** button, which rejects it with "invalid version".

## How to read it

The status comes from the Node.js runtime version that OneAgent detects:

| Status | Meaning | Action |
|---|---|---|
| `VULNERABLE` | Supported line (22, 24, 26) without the fix | Update to the fixed release, then restart |
| `VULNERABLE_EOL` | Line 25, 20, 18 or older, no fix exists | Migrate to 22.x or 24.x |
| `PATCHED` | Fixed release or newer | Nothing to do |
| `UNKNOWN` | Version not readable | Check manually with `node --version` |

- **The number to act on: Workloads / hosts to fix.** It counts the images to rebuild and the servers to update.
- **KPIs:** vulnerable processes, impacted environments and compliance rate, split between Kubernetes and Compute.
- **Tables:**
  - **Status by environment:** one row per namespace or host.
  - **Remediation targets:** one row per workload or host, with its image, versions and process names. Use the process names to tell an exposed server from a CLI tool such as `npm` or `npx`.
  - **Vulnerable processes:** the detail, with state *running* or *ended*.
  - **Unreadable versions.**
  - **Application Security findings** for the CVE.
- **Charts:** vulnerable processes over time, by scope and by environment. They must fall to zero as the updates roll out.
- **Sanity tiles:** *Smartscape freshness* must stay under 15 minutes, otherwise zero values cannot be trusted. *Containerized processes not attached to Kubernetes* should stay close to 0.

## Limits

- The dashboard reads the Node.js **runtime** version. npm dependencies are not covered.
- Not covered: **Bun** (≤ 1.4.3 is vulnerable, fixed in 1.4.4), other software that bundles Brotli 1.0.3–1.2.0, and runtimes without OneAgent (CI runners, serverless functions).
- Distribution packages linked to the system libbrotli can show a "patched" version and still be vulnerable.
- A namespace is attributed only when Kubernetes monitoring is in place. Otherwise the process is counted under its host, in the Compute scope.

## See also

- **PHP2Shell** (CVE-2026-109618 / CVE-2026-109619): the same dashboard for PHP.

## Support

Need help with a cluster that shows "unknown workload", a version that is not detected, or the same dashboard for another runtime (Java, .NET)? Contact PacketPerf.

---
