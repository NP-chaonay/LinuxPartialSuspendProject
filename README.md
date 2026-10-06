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
