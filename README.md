# Wazuh SIEM & Windows Endpoint Monitoring — Persistence Threat Hunting

A Security Operations Center (SOC) lab project built on **Wazuh SIEM/XDR** that monitors
a **Windows 10 endpoint** for internal, host-based adversary behaviour — native process
creation and suspicious DLL loads used for delayed-execution persistence — and pushes
severity-based, color-coded alerts to a **Discord channel** acting as a lightweight SOC
dashboard.

> **Team 2 — Digital Egypt Pioneers Builders (Rwaad Misr), 2026**

---

## Overview

| Item | Value |
|------|-------|
| Hypervisor | VMware Workstation (isolated NAT vSwitch `VMnet8`) |
| Subnet | `192.168.135.0/24` |
| Wazuh Manager (Linux) | `192.168.135.130` |
| Windows 10 Endpoint | `DESKTOP-SLN103I` (`192.168.135.131`) |
| Telemetry source | Microsoft Sysmon (Event ID 1 process create, Event ID 7 image load) |
| Automation | `custom-discord.py` → Discord Webhook (SOC dashboard) |

### Threat scenario
An **internal malware simulator** (local threat) spawns a suspicious background process
that loads `taskschd.dll` to register a scheduled task (delayed-execution persistence).
Sysmon captures the activity and the Wazuh Manager correlates it:

| Rule | Meaning | MITRE ATT&CK |
|------|---------|--------------|
| **67027** | A process was created | T1059 (Command & Scripting Interpreter) |
| **92154** | Process loaded `taskschd.dll` (persistence task) | T1053.005 (Scheduled Task) |

Because these are internal OS events with no network origin, the Discord alert reports
**Target Endpoint = `DESKTOP-SLN103I`** and **Source IP = `N/A`**.

---

## Detection Pipeline

```
Internal Malware Simulator  (DESKTOP-SLN103I)
        │  spawn process / load taskschd.dll
        ▼
Microsoft Sysmon  ──(Event ID 1 & 7)──► Wazuh Agent
        │  encrypted log stream (TCP 1514)
        ▼
Wazuh Manager  ──► decode → rules 67027 / 92154 → MITRE T1053.005 → Wazuh Indexer
        │  trigger integration (ossec.conf)
        ▼
custom-discord.py  ──► format embed (Source IP = N/A) ──► Discord Webhook API
        │
        ▼
SOC Analyst  (color-coded incident card: Green / Orange / Red)
```

---

## Repository Contents

| File / Folder | Description |
|---------------|-------------|
| [`main.tex`](main.tex) | Full LaTeX technical report (Ch. 1–7: Planning, Literature Review, Requirements, Design, Implementation, Testing, Presentation). |
| [`diagrams_master.md`](diagrams_master.md) | All 9 architecture diagrams as compile-ready PlantUML / Mermaid code. |
| [`images/`](images/) | Rendered diagrams, UI screenshots, and evidence images referenced by `main.tex`. |
| `1. The Environment & IP Addresses.txt` | Source notes: environment, tech stack, script logic, rule IDs. |
| `Project Documentation.md` | Official Rwaad Misr documentation guidelines & deadlines. |

### Diagrams in `diagrams_master.md`
1. Use Case · 2. ER Diagram · 3. Data Flow (DFD) · 4. Sequence · 5. Activity ·
6. Agent State · 7. Gantt Chart · 8. Deployment · 9. Component

---

## Building the Report (PDF)

The report uses `pdflatex` (the title page and figures assume the `images/` folder is
present).

```bash
# from the project root
pdflatex main.tex
pdflatex main.tex        # run twice to resolve the ToC / figure references
```

### Rendering the diagrams
The diagram code lives in `diagrams_master.md`. Export each block to a PNG in `images/`
using the matching filename (e.g. `use_case.png`, `sequence.png`, `deployment.png`, …):

- **Mermaid** (DFD, Gantt) — paste into <https://mermaid.live> or use the `mmdc` CLI.
- **PlantUML** (all others) — use <https://www.plantuml.com/plantuml> or the PlantUML CLI.

---

## Images Checklist

`main.tex` references the following in `images/`:

**Rendered diagrams:** `use_case.png` · `erd.png` · `dfd.png` · `sequence.png` ·
`activity.png` · `state.png` · `deployment.png` · `component.png` · `gantt_chart.png`

**Logos:** `mcit_logo.png` · `rwaad_misr_logo.png`

**Screenshots / evidence:** `wazuh_dashboard.png` · `mitre_framework.png` ·
`discord_ui.png` · `sysmon_config.png` · `process_simulation.png` ·
`wazuh_hits_timeline.png`

> **Note:** `sysmon_config.png` and `process_simulation.png` are illustrative mockups.
> Replace them with genuine screenshots from your live Wazuh environment before final
> submission.

---

## Wazuh Integration (`ossec.conf`)

Registered on the Manager at `/var/ossec/etc/ossec.conf`:

```xml
<integration>
  <name>custom-discord.py</name>
  <hook_url>YOUR_DISCORD_WEBHOOK_URL_HERE</hook_url>
  <alert_format>json</alert_format>
</integration>
```

The `custom-discord.py` script parses the JSON alert, performs smart source-IP extraction
(falling back to `N/A` for internal OS events), maps the Wazuh severity level to an embed
color (Green 1–4 / Orange 5–9 / Red 10+), and POSTs the embed to the webhook.

> ⚠️ Keep the live `hook_url` out of version control — it is a secret.

---

## Team 2

| Name | Role |
|------|------|
| Ahmed Ossama | Lead Technical Architect & Engineer |
| Mohamed Deyaa | Project Manager & Documentation Lead |
| Ali Mohamed | Detection & Telemetry Specialist |
| Hoda Alaa | System Integration Specialist |
| Eman Ayman | Penetration Tester & Threat Analyst |
| Mariam Salama | Quality Assurance & Technical Writer |
