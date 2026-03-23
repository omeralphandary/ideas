# Ideas

Product ideas worth exploring. Each idea has a problem statement, existing solutions, the gap, and what a good product looks like.

---

## Idea Index

| # | Name | Domain | Status |
|---|------|--------|--------|
| 1 | [AI Code Security Scanner](#1-ai-code-security-scanner) | Security / DevTools | Exploring |
| 2 | [Workload Placement Intelligence](#2-workload-placement-intelligence) | Data Center / Infra | Exploring |
| 3 | [Hardware Failure Prediction + Auto-Drain](#3-hardware-failure-prediction--auto-drain) | Data Center / Infra | Exploring |
| 4 | [Vision-Based Auto-Tracking Spotlight](#4-vision-based-auto-tracking-spotlight) | Hardware / Live Events / Security | Exploring |

---

## 1. AI Code Security Scanner

**Tagline:** Run this before you ship — plain-English security report for AI-generated code.

### Problem
"Vibe coders" — developers using Cursor, Claude, Copilot, or ChatGPT to generate entire apps — ship insecure code without knowing it. LLMs generate plausible-looking but insecure patterns. Existing security tools (Semgrep, Snyk, CodeQL) are built for security engineers, not non-experts.

### Common AI-generated code vulnerabilities
- Raw SQL string interpolation (SQLi)
- Hardcoded secrets and API keys
- Disabled CORS/CSRF protections
- Missing ownership checks (IDOR) — "delete any user's data, not just your own"
- JWT with `alg: none`
- Over-permissioned auth (wildcard permissions)
- Dependency hallucination → phantom packages registered by attackers (supply chain)
- `eval()` usage, unvalidated redirects, XSS sinks

### What Exists
- **Semgrep** — best open-source SAST, rule-based, CLI/CI. Built for engineers.
- **Snyk** — code + deps + containers. Free tier. Still requires setup knowledge.
- **Aikido Security** — markets around AI code but still enterprise-oriented.
- **Socket.dev** — supply chain / malicious package focus.
- **Bearer** — SAST with data-flow / privacy angle.

### The Gap
- No tool has a **curated ruleset specifically for LLM output patterns**
- None explain issues in plain English to a non-expert ("this lets anyone delete any user's data")
- Zero-setup experience is missing — existing tools need config, CI, YAML
- No severity framing for vibe coders ("this will get you breached in 48 hours")
- Fix suggestions are generic, not framework/context-aware

### Product Vision
- **Zero config** — `npx scan` or drag-and-drop, works immediately
- **LLM-aware ruleset** — patterns specifically common in GPT/Claude/Copilot output
- **Plain English** — explains the vulnerability in human terms + shows the fix in your framework
- **Severity for non-experts** — ranked by "how fast will this get you breached"
- **Dependency check** — flag hallucinated or recently-registered packages
- **Business model** — free CLI → paid CI integration / team dashboard / PDF report

### Moat
1. Original research: curated ruleset for LLM output (nobody has published this comprehensively)
2. UX: feels like a smart friend reviewing code, not a compliance scanner
3. Distribution: get into the Cursor/Windsurf/Claude Code workflow before someone else does

---

## 2. Workload Placement Intelligence

**Tagline:** Continuously decide where each workload runs — bare metal, VM, or cloud — and act on it.

### Problem
In hybrid data centers, workload placement is mostly manual or rule-based. Engineers decide once at deploy time where something runs, and it stays there forever. Nobody continuously re-evaluates based on changing conditions: utilization, spot prices, latency SLAs, egress costs, compliance zone requirements.

### What Exists
- **Morpheus Data** — hybrid cloud management, has placement policies. Complex, enterprise.
- **Densify** — cloud resource optimization, ML-based rightsizing. Cloud-only focus.
- **VMware vRealize** — large VMware shops only, expensive.
- **Spot.io (NetApp)** — spot instance optimization, cloud-native only.

### The Gap
- None continuously re-evaluate across **on-prem bare metal + VMs + multi-cloud** in a unified way
- Cost models rarely account for **egress**, **data gravity**, and **compliance zones** simultaneously
- Recommendations without automation — most tools tell you what to do, not do it
- No real-time re-placement triggered by events (cost spike, SLA breach, hardware risk signal)

### Product Vision
- **Continuous evaluation engine** — scores every running workload against placement criteria on a schedule
- **Multi-dimensional scoring** — latency SLA, current utilization, spot/reserved pricing, egress cost, compliance zone, hardware health signal
- **Automated migration** — triggers live migration via vSphere vMotion, K8s node affinity, or cloud APIs when threshold crossed
- **Human-in-the-loop mode** — recommendation + one-click approval for risk-averse ops teams
- **Integration surface** — Prometheus metrics, cloud billing APIs, IPMI telemetry, K8s scheduler

### Moat
- Unified placement model that spans on-prem and cloud (nobody owns this well)
- Closed-loop: detect → decide → act (not just detect → report)
- Tie-in with Hardware Failure Prediction (Idea #3) — drain workloads off at-risk hardware proactively

---

## 3. Hardware Failure Prediction + Auto-Drain

**Tagline:** Predict hardware failures days in advance and drain workloads before they cause downtime.

### Problem
Data centers react to hardware failures. A disk dies, a DIMM fails, a NIC degrades — and workloads crash, SLAs breach, on-call gets paged at 3am. The telemetry to predict most of these failures hours or days in advance already exists (SMART data, IPMI/BMC metrics, thermal trends, error counters) — it's just not acted on.

### What Exists
- **DCIM tools** (Nlyte, Sunbird, Raritan) — physical inventory + power + thermals. No ML, no prediction.
- **Disk vendor tools** (Seagate SeaTools, WD Dashboard) — per-vendor, not fleet-wide, no automation.
- **Backblaze** — publishes research on HDD failure prediction from their fleet. No product.
- **Cloud providers** — AWS/Azure do this internally for their own hardware. Not sold externally.
- **DellEMC/NetApp/Pure** — storage arrays have some internal health scoring, but narrow scope (their hardware only) and no workload drain integration.

### The Gap
- No vendor-agnostic, fleet-wide failure prediction product
- No closed loop: predict → drain workloads → schedule replacement → verify
- IPMI/Redfish is well-standardized but nobody has built a lightweight SaaS on top of it
- Colocation providers (Equinix, Digital Realty customers) have heterogeneous hardware — no single vendor tool covers them

### Product Vision
- **Telemetry ingestion** — IPMI/BMC via Redfish API, storage array health APIs (NetApp ONTAP, Pure Storage, Dell EMC), NIC error counters via SNMP
- **Per-component failure models** — disk (SMART + vibration + thermal), DIMM (correctable ECC error rate), NIC (error rate trends), PSU (voltage ripple, fan RPM), CPU (thermal throttling patterns)
- **Prediction output** — risk score per component, estimated time-to-failure range, confidence level
- **Automated response** — when score crosses threshold: trigger vSphere vMotion / K8s drain, open ticket in PagerDuty/Jira, page on-call with context
- **Fleet health dashboard** — visual map of all hardware, color-coded by risk, upcoming maintenance calendar
- **Business model** — per-node SaaS pricing, or per-rack for colo providers

### Moat
- Redfish API is standardized — works across Dell, HPE, Supermicro, Lenovo without custom integrations
- Failure model improves with fleet size (data flywheel)
- Tie-in with Workload Placement (Idea #2) — hardware risk is an input to placement decisions
- Clear, measurable ROI: downtime hours averted × cost per hour of downtime

---

---

## 4. Vision-Based Auto-Tracking Spotlight

**Tagline:** A spotlight that follows a person automatically — natural, smooth, real-time. No operator needed.

### Problem
In live shows (concerts, theater, corporate events, sports), following a performer or speaker with a spotlight requires a dedicated human operator — or an expensive motorized fixture controlled manually. The result is either high labor cost, operator fatigue causing lag and jerky movement, or lights that simply stay static.

For security, PTZ (pan-tilt-zoom) cameras track people but lights do not. A space that needs to illuminate a specific person or intruder — a dark warehouse, a stage perimeter, a large outdoor venue — has no intelligent lighting option.

### What Exists

**Live events:**
- **Follow spot operators** — standard solution. Human sits at a manual spotlight, follows the performer. Labor-intensive, skill-dependent, expensive for long shows.
- **Robert Juliat, Strong, Clay Paky** — high-end motorized follow spots. Remote-controlled by an operator via joystick. Still requires a human, just positioned differently.
- **Robe RoboSpot** — closest to this idea. A camera-assisted remote follow spot system. Operator still controls it, camera just helps with framing. ~$15–30K per unit. Sold to large touring productions.
- **MA Lighting / grandMA consoles** — can automate moving heads along pre-programmed paths, but no real-time human tracking.

**Computer vision / robotics:**
- **PTZ camera tracking** (Lumens, PTZOptics, Obsbot) — vision-based auto-tracking exists for cameras. They track faces/bodies. But these are cameras, not lights.
- **Spot robots with lights** — Boston Dynamics Spot has been used at events with mounted lights but it's a $75K robot, not a lighting product.
- **Research prototypes** — academic papers exist on vision-guided robotic lighting, none productized at scale.

### The Gap
- **No affordable, standalone auto-tracking spotlight exists** — the gap between a $200 LED moving head and a $25K RoboSpot is completely empty
- PTZ camera tracking algorithms are proven and cheap — nobody has applied them to control a light beam instead of a camera lens
- "Natural movement" is unsolved — existing motorized fixtures move in hard, mechanical steps. A good follow spot operator adds anticipation, smooth arcs, and body-language prediction. No product does this.
- Dual use (events + security) means two addressable markets with one hardware platform

### Product Vision

**Hardware:**
- Pan-tilt motorized mount (2-axis, high torque for heavy fixtures) with sub-degree precision
- Mounted wide-angle camera for scene capture (separate from the light beam)
- Edge compute module (Jetson Nano / Orin equivalent) running inference on-device
- Works with standard DMX/Art-Net lighting fixtures — doesn't require proprietary light heads
- Optional: laser rangefinder for depth-aware beam focus adjustment

**Vision / Tracking:**
- Person detection + pose estimation (MediaPipe, YOLOv8-pose) running at 30fps+
- Subject selection: tap-to-lock on a person via tablet app, or automatic "most active person" heuristic
- **Predictive smoothing** — model the subject's movement trajectory, not just current position; anticipate direction changes
- Multi-subject handoff — when tracking a group, smoothly transition between subjects on cue
- Low-light optimization — the camera needs to work even when the scene is dark except for the spotlight itself (IR illuminator on the camera, separate from the main beam)

**Control:**
- Tablet/phone app — tap subject to lock, adjust tracking aggressiveness, beam width, speed limits
- DMX integration — lighting board operators can override or blend with automation
- Scene memory — "follow whoever is at the podium" zone-based logic for corporate/theater use

**Security variant:**
- Motion-triggered activation (PIR or camera motion zone)
- Flood beam instead of spot — illuminate entire area around detected person
- Integration with VMS (video management systems) via ONVIF or RTSP
- Alert + illuminate simultaneously

### Dual Market

| | Live Events | Security |
|---|---|---|
| **Buyer** | Event rental companies, venues, touring productions | Commercial security integrators, facility managers |
| **Use case** | Follow performer, speaker, athlete | Illuminate intruder, deter trespassers |
| **Key feature** | Smooth natural tracking, multi-subject | Motion trigger, alarm integration, flood mode |
| **Price tolerance** | $2K–8K per unit | $1K–3K per unit |
| **Distribution** | AV rental houses, lighting dealers | Security system integrators |

### Technical Risks & Mitigations
- **Latency** — tracking lag makes the light feel mechanical. Mitigation: predictive model + optimize inference pipeline to <50ms end-to-end
- **Low-light camera performance** — tracking in dark environments where only the spotlight illuminates the subject. Mitigation: dedicated IR camera array, separate from beam
- **Mechanical precision** — cheap steppers have backlash and vibration. Mitigation: use BLDC motors with encoders, same as camera gimbal industry
- **Subject loss** — performer exits frame, reacquisition must be smooth. Mitigation: re-ID via pose + appearance embedding

### Moat
- The smoothing/prediction algorithm is the core IP — this is what separates it from "servo + YOLO"
- Hardware + software as a system — hard to replicate with off-the-shelf parts
- Network effects in rental ecosystem — once a few large AV rental houses standardize on it, it becomes the default

### Build Path
1. Proof of concept — off-the-shelf PTZ mount + Jetson Orin + MediaPipe person tracking controlling a DMX moving head
2. Validate tracking smoothness and latency in live conditions
3. Custom mount design for production (weight, torque, weather resistance for outdoor)
4. Tablet control app
5. Pilot with 2–3 AV rental companies or corporate venues

---

## How These Connect

Ideas #2 and #3 are naturally complementary — hardware failure risk is one of the strongest signals for workload placement decisions. Together they form a **Hybrid Infrastructure Intelligence Platform**:

```
Hardware Telemetry (IPMI/Redfish/Storage APIs)
        ↓
Failure Prediction Engine  ──→  Risk Score per Node
        ↓
Workload Placement Engine  ──→  Placement Decision
        ↓
Automation Layer  ──→  vMotion / K8s drain / Cloud burst
        ↓
Unified Dashboard  ──→  Fleet health + cost + SLA status
```

Idea #1 (AI Code Security) is independent — different buyer (developer vs. infra ops), different GTM.
