# NebulaOps-RAT Feature Guide
## Next-Generation Windows RAT / Red Team Command Platform

> Operate like a product. Strike like a platform. Present like a war room.

This document is the **operator-facing capability breakdown**.  
It explains what each module actually does - including build-time evasion, stealth mining controls, memory tradecraft, beacon packaging, and campaign delivery.

Search keywords: `RAT`, `Remote Access Trojan`, `Windows RAT`, `C2`, `NebulaOps-RAT`

---

## 1. Product Promise

NebulaOps-RAT is a Windows Remote Access Trojan (RAT) Control Center for authorized red-team / adversary-simulation workflows.

It is designed for operators who want:

- **Build -> Harden -> Package -> Deliver**
- **Beacon -> Mission -> Memory -> Lateral -> Report**
- A UI that looks like a commercial command platform, not a weekend script

---

## 2. Builder + Dynamic Evasion Pipeline (the "generate and harden" path)

This is the feature most operators care about:  
**you do not just compile an agent - you run a post-build evasion pipeline and get a scored artifact.**

### 2.1 What happens when you click Build
1. Panel takes the base agent payload (`client.bin`)
2. Rewrites connection / tag / install options
3. Applies PE version metadata (product/company/copyright/filename/version)
4. Runs **EvasionPipeline** transforms via Mono.Cecil
5. Optionally generates shellcode exports
6. Emits a `.nebula-report.txt` score report next to the output

### 2.2 Evasion switches (implemented)
| Switch | Default behavior | What it does |
|---|---|---|
| **Junk Methods** | ON | Injects a private type with ~10 junk static methods (noise / entropy / signature disruption) |
| **String Noise Pool** | ON | Injects fake telemetry/compliance strings (e.g. `Aurora.Telemetry.*`, `EndpointHealthCheck`) |
| **Method Proxying** | ON | Creates proxy wrappers around selected static methods to alter call-graph shape |
| **Control-Flow Noise** | ON | Inserts opaque branch/nop patterns into method bodies |
| **Anti-Sandbox Stub** | ON | Injects `EnvironmentGate` with startup delay + CPU count gate helper |
| **Overlay Padding** | ON | Appends random padding (default ~64-96 KB) after PE to change size/hash profile |
| **Random Vendor Metadata** | ON (recommended) | Rotates fake vendor identities such as CloudSync Utilities / Aurora Device Care / Contoso Endpoint Helper |

### 2.3 Anti-sandbox / startup stealth details
- Configurable **startup delay** (Builder defaults around **2500-3200 ms**)
- Injected gate helper can sleep before continuing
- Includes a simple environment check path (`ProcessorCount`-style gate stub)
- Goal: reduce immediate detonation behavior in automated sandboxes

### 2.4 Metadata camouflage details
Randomized assembly info can include:

- Product name
- File description
- Company name
- Copyright
- Internal / original filename
- Product + file version

This makes the built agent look like a generic maintenance / endpoint utility instead of a known RAT brand.

### 2.5 Residual marker scan
After transforms, NebulaOps scans for legacy brand markers (e.g. Quasar / AsyncRAT / LimeRAT style strings).  
Clean scan improves the score; leftover markers reduce it.

### 2.6 Build score report
Every hardened build can emit:

- Output path
- SHA256
- Score `/100`
- Applied steps
- Findings

This is intentional: operators can show "what was applied" and "what the artifact looks like after hardening."

> Practical note: this is a **build-time evasion / packaging pipeline**, not a magical forever-undetected guarantee. AV ecosystems change. NebulaOps-RAT gives you the operator workflow to regenerate, reshuffle metadata, and re-score quickly.

---

## 3. Shellcode Factory (export what injectors need)

### 3.1 Generation strategy
1. Prefer **donut.exe** when present -> real PIC shellcode
2. If donut is missing -> emit **Nebula stub pack** (encrypted PE + loader stub) so exports still work

### 3.2 Export formats
- `.bin` raw
- HEX
- C array
- PowerShell
- Python
- C#

### 3.3 Why this matters
Operators can take one built agent and immediately produce injector-ready artifacts for:

- Memory Workbench
- Injector panel
- Campaign packs
- External loaders

---

## 4. Injector

Direct shellcode dispatch to selected agents.

### Techniques
- **LocalThread**
- **RemoteThread**

### Options
- Load `.bin` / `.donut.bin` / raw payloads
- Optional **XOR-encoded** payload mode
- Multi-agent fan-out

Use case: "I already have shellcode - push it now."

---

## 5. Memory Tradecraft Workbench

This is the in-memory execution console.

### Techniques
| Technique | Intent | OPSEC posture (built-in scoring) |
|---|---|---|
| **AssemblyLoad** | Managed in-memory load path | Higher score / lower noise preference |
| **EarlyBirdAPC** | Early execution style chain | Strong preferred middle/high path |
| **Hollow** | Process-hollowing style chain | Medium |
| **RemoteThread** | Classic remote thread | Lower score / noisier |
| **LocalThread** | Local thread execution | Lowest score / noisier |

### Operator controls
- Payload browse (`.bin` / `.exe` / `.dll`)
- Target process name (default `notepad.exe`)
- Target PID
- XOR key + "payload XOR-encoded"
- Assembly args
- Multi-agent selection
- Live **OPSEC score + advice**

### OPSEC advice engine examples
- High score: prefer as primary in-memory path
- Medium: pair with delay / off-peak execution
- Low: switch to EarlyBirdAPC / AssemblyLoad and reassess

---

## 6. Mining Console - stealth / concealment oriented controls

NebulaOps-RAT does **not** present mining as a loud desktop app.  
The Mining Console is built around **low-profile operator controls**.

### 6.1 Concealment and stealth features
| Control | What it does |
|---|---|
| **Process Name** | Runs miner under a camouflage name (default `runtime-svc`) |
| **Hidden window** | Starts with `CreateNoWindow=true` + `WindowStyle=Hidden` |
| **Idle-only mode** | Intended to mine when user appears idle (default ON) |
| **CPU Cap %** | Soft-limits aggressiveness (priority class mapping) |
| **Watchdog** | Auto-restart if process dies (default 15s) |
| **Persist profile** | Saves miner profile on agent for reuse |
| **Remote Miner URL** | Agent can fetch miner binary on demand instead of shipping it visibly |
| **Working directory camouflage** | Uses ProgramData-style runtime folder (`NebulaRuntime`) |

### 6.2 Operational flow
1. Operator sets wallet / pool / worker / process name / CPU / watchdog
2. Optional miner URL for remote fetch
3. Config is pushed to agents (`DoMinerConfig`)
4. Start / Stop commands fan out to fleet
5. Status query returns running state / hashrate / process info
6. Dashboard can reflect fleet hashrate telemetry

### 6.3 Persistence details
When persist is enabled, agent stores profile fields such as:

- wallet
- pool
- CPU percent
- idle-only flag
- worker name
- process name
- miner URL
- watchdog seconds

### 6.4 Why operators use this
- Rename process to look like endpoint helper / runtime service
- Keep UI hidden
- Cap CPU to reduce user complaints
- Auto-revive with watchdog
- Fetch binary late (URL) instead of packaging loudly

---

## 7. Beacon Builder (multi-channel agent generation)

### Channels / profiles
- TCP-TLS
- HTTPS-WS
- Cloudflare CDN redirector profile
- DoH profile
- Telegram backup profile

### What "real build" means here
Beacon Builder is not only markdown docs:

1. Selects channel
2. Encodes channel into agent **TAG** (`base|ch=https-ws`)
3. Builds through ClientBuilder path
4. Applies evasion pipeline defaults
5. Emits per-channel agent outputs + summary

This lets operators generate multiple protocol personalities from one console.

---

## 8. Campaign Wizard (one-click delivery pack)

Builds a ZIP package containing selected modules:

- Hardened agent
- Shellcode pack
- Mining profile
- Lateral playbook skeleton
- HVNC shortcut notes
- README + evasion score metadata

Default campaign hardening includes junk / string noise / proxies / control-flow noise / anti-sandbox delay / overlay padding.

Result: a ready-to-hand-off delivery bundle.

---

## 9. Mission Scheduler + Live Timeline

SOAR-style offensive workflow engine.

### Example steps
- `sysinfo`
- `screenshot`
- `hvnc`
- `lateral:DiscoverOnly`
- `lateral:SprayLight`
- `lateral:ShareDrop`
- `lateral:PsRemote`
- `miner:start`
- `sleep`

### Timeline features
- Mission id / agent / step progress / state / detail
- Visual timeline canvas
- Poll status refresh
- Audit logging of dispatch events

---

## 10. Attack Graph (click-to-action)

Visual battle map for:

- Hosts
- Credentials
- Shares
- Domain Controllers

### Node playbooks (examples)
- Host -> `sysinfo|lateral:DiscoverOnly|screenshot`
- Share -> `lateral:ShareDrop|sysinfo`
- Credential -> `sysinfo|lateral:SprayLight`
- DC -> `sysinfo|lateral:PsRemote|screenshot`

Click node -> dispatch playbook.  
Graph is not decoration; it is an execution surface.

---

## 11. Lateral Lab

### Playbooks
- **DiscoverOnly**
- **SprayLight**
- **ShareDrop**
- **PsRemote**

### Safety / operator gate
- Offensive actions require **Armed** checkbox
- ShareDrop / PsRemote can require payload file
- Supports CIDR expansion and service probing (SMB/RDP/WinRM style)

---

## 12. HVNC (Hidden Desktop)

- Start / stop hidden desktop session
- Frame polling UI
- Quality controls
- Desktop name / resolution status feedback

Designed for concealed interactive access paths during engagements/labs.

---

## 13. War Room + Geo Map + Dashboard

### War Room
Fullscreen demo/ops board with live counters and shortcut launches.

### Geo Fleet Map
Country-aware fleet situational view (including demo markers for empty labs).

### Dashboard
Online fleet, listener state, hashrate telemetry, lateral activity, event stream, module shortcuts.

---

## 14. Governance: RBAC / Audit / Report

- Role skeleton: Owner / Operator / Viewer
- Persistent `audit.log`
- One-click HTML red-team style report export
- Module Store UI for installed vs marketplace narrative

---

## 15. End-to-End Operator Loops

### A) "Generate hardened agent" loop
Builder -> evasion transforms -> score report -> optional shellcode export -> Campaign ZIP

### B) "In-memory execution" loop
Shellcode Factory -> Injector / Memory Workbench -> OPSEC score -> fleet dispatch

### C) "Stealth mining" loop
Mining Console -> camouflage process name -> hidden start -> idle/CPU caps -> watchdog -> persist -> status telemetry

### D) "Command narrative" loop
War Room -> Attack Graph -> Mission Timeline -> Beacon Builder -> Audit HTML report

---

## 16. Runtime Package Contents

Forum/GitHub release package typically includes:

- `NebulaOps.exe` (Control Center)
- runtime dependencies
- Feature Guide
- Operator README

Requirements:

- Windows (x64 recommended)
- .NET Framework 4.5.2+

---

## 17. Responsible Use

NebulaOps-RAT is intended for:

- authorized red team engagements
- adversary simulation labs
- purple-team exercises
- security research / training

Use only on systems you own or are explicitly authorized to test.

---

## 18. One-Liner

**NebulaOps-RAT is the Windows RAT / red-team command platform that turns evasion packaging, stealth mining controls, memory tradecraft, multi-channel beaconing, and mission orchestration into one demo-ready operator product.**

-
NebulaOps-RAT / Nebula Labs
