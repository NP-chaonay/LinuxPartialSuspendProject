# LinuxPartialSuspendProject

> REMARK: This documentation is incomplete and currently under development. Sections marked [DRAFTING <...> PART] are work-in-progress and may lack polish, but they are available if you want early access to upcoming content. Please note that these sections are subject to change.

> REMARK: Except for the "DRAFTING PART" and "DRAFTING -- COPY FROM AI PART" sections, the text in this documentation has been refined by AI to ensure a professional standard.

> หมายเหตุสำหรับผู้อ่านภาษาไทย: สำหรับคนใช้ภาษาไทย พวก README/Wiki/ส่วนสำคัญ จะแปลให้นะครับแต่อาจจะช้าหน่อย เพราะผมจะเขียนเป็นภาษาอังกฤษก่อน

## Welcome

This project is part of a larger initiative: We Will Make Linux Desktop Great Again!

- If you have any questions or feedback, feel free to open an Issue.
- Want to contribute or volunteer? Feel free to start a discussion in the "Issue" tab.

## Plan: Project Components
- Main daemon
  - run as root
  - run per user
- Desktop Environemnt Modification
- Systemd Integration
  - <TBA>
- <TBA>

## Experiments Done
- Does freezing an audio-playing app pause its audio?
> works normally
- Freezing cgroups for both "system.slice" and "user.slice"
> **Issue:** Sometimes buggy—causes user screen access to fail (likely an issue with seat management).
> 
> **Symptom:** The system switches to the login screen $\rightarrow$ entering the password causes an infinite checking loop $\rightarrow$ manually switching via Virtual Terminal (VT) leads to a partially frozen state (stuck on the login UI, un-interactive, though the mouse cursor still moves).
> 
> **Next Step:** We need to find a solution (possibly via selective cgroup freezing); otherwise, User Suspending won't be viable.

## FAQ

**What is this project for?**  
<TBA>

**What is the goal of this project?**
- **First:** Build base code that allows users to run specific background tasks while suspending all others.
- **Second:** Improve the system for better UX, deeper integration, and higher application stability.
- **Third:** Go further by promoting and advocating for this as an out-of-the-box (OOBE) feature for Linux Desktop via a freedesktop.org RFC proposal.
  - The starting point: https://www.freedesktop.org/wiki/NewProject/
**Is this project the opposite of battery saving?**  
Nope. Reducing unnecessary background tasks is useless if power consumption remains high. This project is battery-focused. It guarantees that you can control the tradeoff between power savings and functionality—you can choose power savings similar to traditional full suspend while retaining selective functionality (or even while fully suspended).  
<TBA>

**Is my system compatible?**  
- This project is compatible with Linux distributions featuring the following components:
  - systemd
  - cgroup v2 (or newer)
  - Desktop environment integrated with systemd's cgroup management (recommended for full features)
- However, our primary target and testing environment is the latest Ubuntu LTS release (x86_64) running GNOME.
- We don't know how beneficial this is on non-s0ix systems, but it is definitely useful on systems supporting s0ix.

**What is s0ix?**  
A modern power state on x86 architectures, implemented at the hardware level, that allows unused system sub-components to power down their circuits dynamically.

**How does this project plan power states in user-space?: (see below)**
> REMARK: These power states are used internally and meant for power-users or developers. End-users have a different perspective and set of terminology.
- **Q1: Screen On**
- **Q2: Screen Off** (DPMS Off / DRM Suspended)
- **Q3: Partial Suspending** (New state proposed by us)
- **Q3L: Low-Power Partial Suspending** (New state proposed by us; same as Q3, but capped at a very low maximum power draw, e.g., $\le$ 500mW)
- **Q4: Userspace Suspending** (New state proposed by us; almost all of userspace is frozen except vital services like our system and init. Because the kernel stays running, system resume is much faster)
- **Q5: Full Suspending** (Standard kernel-level suspend API; `s2idle` on modern systems)

Sub-states (Q3+):
- Network Status: Connected | Disconnected (Allowed Wi-Fi / Ethernet)
- Allow Periodic Wakeup: Yes | No
  * Enables background applications and services to execute periodic tasks efficiently during brief wake intervals to conserve battery.

# [DRAFTING PART]

## Breif StateMap
> Screen On (Smart BG Power Save) >> Screen Off (Smart BG Power Save but more aggressive policy + depends on app... can be resource-throttled or freezed-and-resume-shortly-then-be-freezed) >> Partial Suspending (App must request, to continue in this state + app that doesnt request can request perodic wakeup) >> Full Suspending (We can make it smart by allow perodic wakeup to do something shortly quick and battery-save... and quickly return to full suspending)

# [DRAFTING - COPY FROM AI PART]

Here is a clean, technical spec draft ready to paste directly into your GitHub Wiki, README, or project documentation.

---

# Architecture Specification: Power Menu & State Mapping

## Overview

This document outlines the User Experience (UX) abstractions and their mapping to underlying system power states ($Q_1$–$Q_5$). The design hides complex kernel and cgroup operations behind intuitive power actions, delivering a modern smartphone-like standby experience while maintaining strict battery safeguards.

---

## Technical Power States Summary

The system orchestrates power states by dynamically managing process freezing, hardware run-time power management (Runtime PM), and suspend targets.

```
[Screen On (Q1)] ──> [Screen Off (Q2)] ──> [Partial Suspend (Q3)] ──> [Userspace Suspend (Q4)] ──> [Full Suspend (Q5)]

```

* **$Q_1$ (Screen On):** Active session with smart background throttling.
* **$Q_2$ (Screen Off):** DPMS/DRM Display Suspended; aggressive cgroup throttling active.
* **$Q_3$ (Partial Suspend):** Non-whitelisted applications frozen; explicit background task holds (e.g., audio streaming, active downloads) permitted.
* **$Q_4$ (Userspace Suspend):** Near-complete userspace freeze (all non-essential cgroups halted; core system and init preserved). Kernel remains active for near-instantaneous resume ($<1\text{s}$).
* **$Q_5$ (Full Suspend):** Deep hardware suspend (`s2idle`/`S3`). Supports controlled, periodic timers (RTC/AlarmTimer) for background checks before re-entering deep sleep.

---

## Power Menu UX Actions & State Mapping

### 1. `Screen Off and Standby`

Designed as the default short-press response (equivalent to a mobile power button).

* **User Intent:** Instantly blank the screen while maintaining full background continuity.
* **State Transition:** $Q_1 \longrightarrow Q_2 \longrightarrow Q_3 / Q_4 \text{ (on idle)}$
* **Behavior:**
* Displays are powered down via DPMS/DRM.
* System enters an idle state, progressively escalating to $Q_3$ or $Q_4$ based on background locks.



---

### 2. `Suspend` (Main Profile)

The primary intelligent power-saving action for general laptop usage.

* **User Intent:** Maximum power savings with minimal resume delay and zero risk of unexpected battery drain.
* **State Transition:** Dynamic escalation through $Q_3 \longrightarrow Q_4 \longrightarrow Q_5$
* **Behavior Matrix:**
* **Phase 1 ($Q_3$):** If background operations request a hold (e.g., media playback), active holds are respected while all unrequested applications are frozen.
* **Phase 2 ($Q_4$):** Once background holds release, the daemon freezes all userspace processes, leaving the system in a low-latency sleep state (~$75\text{--}100\text{mW}$ draw, $<1\text{s}$ resume latency).
* **Phase 3 ($Q_5$):** Upon reaching a configurable timeout or battery threshold, the system escalates to kernel `s2idle` (~$50\text{mW}$ draw, ~$3\text{--}4\text{s}$ resume latency).


* **Power Safety Guard:** Active power monitoring tracks current draw during $Q_3$. If power draw exceeds expected thresholds, the user is alerted or forced into $Q_4$/$Q_5$.

---

### 3. `Full Suspend` (s2idle + Periodic Wakeup)

Optimized for extended off-grid storage or long transport.

* **User Intent:** Guaranteed deep battery preservation with smart background capabilities.
* **State Transition:** Direct $Q_5$ entry.
* **Behavior:**
* Enforces kernel-level deep suspend (`s2idle`).
* Integrates periodic RTC/AlarmTimer interrupts to perform short, batched background checks before immediately re-entering $Q_5$.



---

### 4. `System Standard Actions`

Handled via standard desktop environment integration or direct `systemd-logind` calls.

* **Supported Actions:** `Power Off`, `Reboot`, `Hibernate`, `Logout`, `Switch User`.

---

### 5. `Advanced / Developer Overrides`

Exposed in sub-menus or CLI utilities for debugging and power profiling.

* **Force State Overrides:** `Force Q3`, `Force Q4`, `Force Q5 (Static / No Periodic Wake)`.
* **Systemd Hybrids:** Native passthrough for `suspend-then-hibernate` and `hybrid-sleep` (handled entirely by `systemd`).

---

## State Comparison Reference

| Metric / State | $Q_3$ (Partial) | $Q_4$ (Userspace) | $Q_5$ (Full / s2idle) |
| --- | --- | --- | --- |
| **Userspace Status** | Selected Apps Active / Rest Frozen | Entirely Frozen (Except Init/Daemon) | Entirely Frozen |
| **Kernel Status** | Running | Running | Suspended |
| **Estimated Draw** | Variable ($>100\text{mW}$) | $\sim 75\text{--}100\text{mW}$ | $\sim 50\text{mW}$ |
| **Resume Latency** | Instant ($<0.1\text{s}$) | Near-Instant ($<1\text{s}$) | Hardware Latency ($\sim 3\text{--}4\text{s}$) |
| **EC LED Indicator** | Active / Solid | Active / Solid | Pulled low / Blinking (Hardware dependent) |
