# LinuxPartialSuspendProject

# This docs is in drafting here below is completed part you can view (but it will be later added)

But if you have question or something to tell please tell in **Issues**

สำหรับคนใช้ภาษาไทย พวก README/Wiki/ส่วนสำคัญ จะแปลให้นะครับแต่ะล่าช้ากว่า เพราะผมจะเขียนเป็น English ไปก่อน

## FAQ

**What is this project for?:** <TBA>
**What is goal of this project:**
- First: We make basic code leis goal of this projecttting user able to run some bg tasks while the other tasks are suspending
- Secondly: We improve the system to have better UX + better integration + better app. stability
- Third: We can to do more than we do... we promote and advocate this into OOBE experience of Linux Desktop by make a RFC for freedesktop.org
**Is this project is opposite side of battery-saving:** Nope. if we reduce unnecessary bg tasks, but we still get high power usage, then it is useless. This project is battery-focused. My project guarantee that you can control how much power saving trade-off with functionality; you can choose to let battery saving similar to the traditional full suspending while kept few functionality (OR EVEN IN FULL SUSPENDING).
<TBA>

****
**How this project plans about power state:**
- Q1: Screen On
- Q2: Screen Off (DPMS Off / DRM Suspended)
- Q3: Partial Suspending (New state proposed by us)
- Q4: Userspace Suspending (New state proposed by us; almost userspace is freezed, except some (such as our system + init process), kernel is still running, thus makes system resuming does more quickly)
- Q5: Full Suspending (Kernel API for suspending, in modern system is s2idle)

# Below this line is heavy unstable... can be moved to above/deleted

## Breif StateMap
> Screen On (Smart BG Power Save) >> Screen Off (Smart BG Power Save but more aggressive policy + depends on app... can be resource-throttled or freezed-and-resume-shortly-then-be-freezed) >> Partial Suspending (App must request, to continue in this state + app that doesnt request can request perodic wakeup) >> Full Suspending (We can make it smart by allow perodic wakeup to do something shortly quick and battery-save... and quickly return to full suspending)

# Below this line is copied from AI

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
