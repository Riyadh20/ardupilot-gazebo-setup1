# ArduPlane Fixed-Wing + Gazebo SITL — Setup & Flight Guide

> Ubuntu 24.04 | Gazebo Harmonic | ArduPlane V4.8+ | Zephyr Fixed-Wing

---

## Overview

This guide covers setting up and flying a **Fixed-Wing (Delta Wing) ArduPlane** simulation using Gazebo Harmonic and SITL on Ubuntu 24.04.

Unlike the Copter setup (see `README.md`), fixed-wing aircraft require different launch procedures, flight modes, and takeoff sequences.

---

## Prerequisites

Make sure you have completed the base setup from `README.md`:

- ArduPilot cloned and built
- Gazebo Harmonic installed
- `ardupilot_gazebo` plugin built and paths set in `.bashrc`
- Mission Planner installed via Mono

---

## Step 1 — Verify Available Worlds

```bash
ls ~/ardupilot_gazebo/worlds/
```

You should see fixed-wing worlds including:

```
zephyr_runway.sdf        ← Main fixed-wing world ✅
zephyr_parachute.sdf
gimbal.sdf
iris_runway.sdf
iris_warehouse.sdf
```

---

## Step 2 — Launch Gazebo (Terminal 1)

```bash
gz sim -v4 -r zephyr_runway.sdf
```

Wait until you see the Zephyr aircraft on the runway in the Gazebo window.

**Entity Tree should show:**
```
default
axes
runway
zephyr_with_ardupilot    ← Our fixed-wing aircraft ✅
sun
```

---

## Step 3 — Launch ArduPlane SITL (Terminal 2)

```bash
cd ~/ardupilot
sim_vehicle.py -v ArduPlane -f gazebo-zephyr \
  --model JSON --map --console \
  --out=udp:127.0.0.1:14550
```

**Wait for these messages in the console:**
```
AP: EKF3 IMU0 tilt alignment complete
AP: EKF3 IMU0 initial yaw alignment complete
AP: EKF IMU0 origin set
AP: EKF IMU0 is using GPS
pre-arm good
```

---

## Step 4 — Connect Mission Planner

1. Open Mission Planner
2. Top right: select **UDP**
3. Port: **14550**
4. Click **CONNECT**

**Verify connection:**
```
GPS: OK (10+)
Mode: MANUAL
AirSpeed sensor active
```

---

## Step 5 — Takeoff (Fixed-Wing Procedure)

> ⚠️ Fixed-wing aircraft CANNOT take off vertically like a copter.
> The Zephyr uses a catapult-style launch — requires throttle push on the runway.

### Set Takeoff Parameters

In SITL Terminal:

```bash
param set TKOFF_THR_MINACC 0
param set TKOFF_THR_DELAY 0
param set TECS_PITCH_MAX 35
param set GROUND_STEER_ALT 0
param set TKOFF_ROTATE_SPD 0
```

### Switch to FBWA Mode

```bash
mode fbwa
```

### Apply Full Throttle (Catapult Launch)

```bash
rc 3 2000
```

The aircraft will accelerate down the runway and climb automatically. ✅

---

## Step 6 — In-Flight Commands

| Command | Action |
|---|---|
| `mode fbwa` | Fly By Wire A — manual direction, safe limits |
| `mode auto` | Full autopilot — follows waypoints |
| `mode loiter` | Circle above current position |
| `mode cruise` | Maintain altitude + speed automatically |
| `mode rtl` | Return to launch point and land |
| `rc 3 2000` | Full throttle |
| `rc 3 1500` | Half throttle |
| `rc 3 1000` | Zero throttle |

---

## Step 7 — Land (RTL)

```bash
mode rtl
```

The aircraft will return to the launch point and land automatically.

---

## Step 8 — Find Your Log File

After landing and closing SITL, the BIN log is saved automatically:

```bash
ls ~/ardupilot/logs/
```

You will see:
```
00000001.BIN    ← First flight log
00000002.BIN    ← Second flight log
...
```

---

## Step 9 — Analyze Log in Mission Planner

1. Open Mission Planner
2. Click **Flight Data**
3. Click **DataFlash Logs** tab
4. Click **Review a Log**
5. Navigate to `~/ardupilot/logs/`
6. Select your `.BIN` file

**Key messages to analyze:**

| Log Group | What to check |
|---|---|
| `MODE` | Flight modes used — story of the flight |
| `ATT` | Pitch / Roll / Yaw vs Desired values |
| `ARSPD` | Airspeed — watch for Stall risk below 12 m/s |
| `TECS` | Energy management — speed and altitude control |
| `GPS` | HDop < 1.5 = good / NSats > 8 = good |
| `VIBE` | Vibration < 30 m/s² = normal |
| `RCOU` | Servo outputs — C1/C2 = Elevons / C3 = Throttle |

---

## Common Issues & Fixes

| Error | Cause | Fix |
|---|---|---|
| `Bad launch AUTO` | Copter-style takeoff used | Use `mode fbwa` + `rc 3 2000` |
| `NAV_TAKEOFF: FAILED` | ArduPlane can't take off vertically | Use runway launch procedure |
| `ArduPilot controller has reset` | SITL lost sync with Gazebo | Start Gazebo before SITL |
| `No JSON sensor message` | Plugin not loaded | Rebuild `ardupilot_gazebo` plugin |
| `zephyr_autopilot.sdf not found` | World not available locally | Use `zephyr_runway.sdf` instead |
| `Unable to find parameter FBWA_TDRAG_CHAN` | Not available in this version | Skip — not needed |

---

## Key Differences: Fixed-Wing vs Copter

| | Fixed-Wing (ArduPlane) | Copter (ArduCopter) |
|---|---|---|
| Vehicle | `-v ArduPlane` | `-v ArduCopter` |
| Frame | `-f gazebo-zephyr` | `-f gazebo-iris` |
| World | `zephyr_runway.sdf` | `iris_runway.sdf` |
| Takeoff | Runway roll + climb | Vertical |
| Hover | Not possible — uses Loiter circles | Yes |
| Control surfaces | Elevons (Pitch + Roll combined) | 4 motors |
| Stall risk | Yes — below ~12 m/s | No |

---

## Quick Start Cheatsheet

```bash
# Terminal 1 — Gazebo
gz sim -v4 -r zephyr_runway.sdf

# Terminal 2 — SITL
cd ~/ardupilot
sim_vehicle.py -v ArduPlane -f gazebo-zephyr \
  --model JSON --map --console \
  --out=udp:127.0.0.1:14550

# In SITL Terminal — Takeoff
param set TKOFF_THR_MINACC 0
param set TKOFF_THR_DELAY 0
param set TECS_PITCH_MAX 35
param set GROUND_STEER_ALT 0
param set TKOFF_ROTATE_SPD 0
mode fbwa
rc 3 2000

# Land
mode rtl

# Find Log
ls ~/ardupilot/logs/
```

---

## Related

- [ArduPlane Documentation](https://ardupilot.org/plane/)
- [Gazebo Harmonic + ArduPilot Plugin](https://github.com/ArduPilot/ardupilot_gazebo)
- [Base Setup (ArduCopter)](./README.md)
