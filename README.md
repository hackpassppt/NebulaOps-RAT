<p align="center">
  <img src="assets/screenshots/01-control-center.png" alt="NebulaOps Control Center" width="920" />
</p>

<h1 align="center">NebulaOps-RAT</h1>

<p align="center">
  <b>Windows RAT / Red Team Command Platform</b><br/>
  <i>Research-oriented operator console for authorized adversary simulation</i>
</p>

<p align="center">
  <img alt="platform" src="https://img.shields.io/badge/Platform-Windows-0ea5e9?style=for-the-badge" />
  <img alt="runtime" src="https://img.shields.io/badge/.NET-4.5.2+-8b5cf6?style=for-the-badge" />
  <img alt="type" src="https://img.shields.io/badge/Type-RAT%20%2F%20C2%20Research-ef4444?style=for-the-badge" />
  <img alt="use" src="https://img.shields.io/badge/Use-Authorized%20Labs%20Only-10b981?style=for-the-badge" />
</p>

---

## Search Keywords

`RAT` `Remote Access Trojan` `Windows RAT` `C2` `Red Team` `NebulaOps-RAT` `Adversary Simulation`

---

## What this repository is

This repository is the **public landing page** for NebulaOps-RAT:

- product overview
- screenshots
- feature documentation

It is intended for authorized red-team research, adversary-simulation labs, and security engineering demos.

> Runnable release packages are distributed through private / forum channels, not as a public malware drop.

---

## Highlights

- Light-theme Windows Control Center
- Builder with post-build evasion pipeline and score report
- Shellcode Factory + Injector + Memory Workbench
- Stealth Mining Console (hidden process / idle / CPU / watchdog / persist)
- Multi-channel Beacon Builder
- Campaign Wizard / Mission Scheduler / Attack Graph / Lateral Lab / HVNC

---

## Screenshots

### Control Center
<img src="assets/screenshots/01-control-center.png" width="900" alt="Control Center" />

### War Room
<img src="assets/screenshots/02-war-room.png" width="900" alt="War Room" />

### Mission Scheduler
<img src="assets/screenshots/03-mission-scheduler.png" width="900" alt="Mission Scheduler" />

### Memory Tradecraft Workbench
<img src="assets/screenshots/04-memory-workbench.png" width="900" alt="Memory Workbench" />

### Attack Graph
<img src="assets/screenshots/05-attack-graph.png" width="900" alt="Attack Graph" />

### Beacon Builder
<img src="assets/screenshots/06-beacon-builder.png" width="900" alt="Beacon Builder" />

---

## Documentation

- [Full Feature Guide](docs/NebulaOps_Feature_Guide.md)
- [Feature Guide DOCX](docs/NebulaOps_Feature_Guide.docx)
- [Operator README](docs/README.txt)

---

## Capability Snapshot

| Area | Details |
|---|---|
| Evasion pipeline | junk methods, string noise, proxies, control-flow noise, anti-sandbox delay, overlay padding, metadata camouflage, score report |
| Memory tradecraft | AssemblyLoad / EarlyBirdAPC / Hollow / RemoteThread / LocalThread + OPSEC score |
| Stealth mining | camouflage process name, hidden window, idle-only, CPU cap, watchdog, persist, remote URL fetch |
| Beaconing | TCP-TLS / HTTPS-WS / Cloudflare / DoH / Telegram profiles |
| Ops surfaces | War Room, Mission Timeline, Attack Graph, Lateral Lab, HVNC |

---

## Responsible Use

NebulaOps-RAT is for:

- authorized red team engagements
- adversary simulation labs
- purple-team exercises
- security research and training

Use only on systems you own or are explicitly authorized to test.

---

<p align="center">
  <b>NebulaOps-RAT / Nebula Labs</b><br/>
  Public research landing page
</p>
