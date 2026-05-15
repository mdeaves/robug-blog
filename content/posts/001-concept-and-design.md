---
title: "Entry 001 — Concept, Design Decisions, and the Software Stack"
date: 2026-05-15
tags: ["design", "ROS2", "VESC", "OAK-D", "hoverboard"]
summary: "The origin of RoBug, the hardware and software architecture decisions, and why I'm building a prototype with hoverboard motors before committing to the full build."
---

## How it started

It started with a stump problem. We have a rural property in Ontario with more stumps than I'd like, uneven terrain, and a lot of heavy hauling to do. I wanted something that could carry 200–250 kg of payload across rough ground, pull stumps with a winch, and not cost $20,000.

Then my six-year-old son Charlie said it should be able to talk. That got me thinking about what else it could do — telemetry streaming, autonomous follow-me, obstacle detection. One thing led to another.

## The name

Charlie named it RoBug. It stuck.

## Core hardware decisions

### Motors: wheelbarrow hub motors

After looking at QS138 motors with chain drive, I settled on purpose-built wheelbarrow hub motors — 2,000W continuous, 48V, with an integrated 5:1 planetary gearbox and tyre. Two motors per side, four total, each independently controlled.

The advantages are significant: no chain, no sprockets, no alignment headaches. Each motor is a complete wheel assembly. Mount the frame, bolt the motors, done.

Rated for 800 kg load capacity per motor. RoBug's all-up loaded weight is around 450 kg. Plenty of margin.

### Motor controllers: VESC

The VESC (Vedder Electronic Speed Controller) is open source motor controller firmware that runs on dedicated hardware. I'm using the Flipsky FSESC 6.7 Pro — one per motor, coordinated via CAN bus.

Why VESC over cheaper alternatives:

- **FOC (Field Oriented Control)** — sinusoidal commutation, smooth at very low speeds, efficient
- **UART telemetry** — the controller streams RPM, current, voltage, temperature over serial
- **ROS2 driver** — `vesc_driver` is a maintained ROS2 package, no custom code needed
- **CAN bus** — multiple controllers coordinate over a single bus without Pi involvement
- **Regenerative braking** — kinetic energy goes back into the battery on deceleration
- **Open source** — fully configurable, large community

The alternative controllers (Kelly, Fardriver, Sabvoton) are fine for e-bikes but have no ROS2 ecosystem. VESC is what you use when the motor controller is a component in a larger software system.

### Battery: 48V 200Ah LiFePO4

LiFePO4 (lithium iron phosphate) chemistry for outdoor utility use — more thermally stable than lithium-ion, longer cycle life, safer. 200Ah at 48V is ~9.6 kWh. At 3–4 kW typical draw, that's roughly 2.5 hours of heavy work.

The BMS (Battery Management System) needs to handle 500A continuous — the winch alone can draw ~400A peak. Most commodity BMS units are rated for 100A. I'll use a JK BMS 500A or build a pack with two 100Ah units in parallel.

### Brain: TBD between Pi 5 and Jetson Orin Nano

The Raspberry Pi 5 handles the core stack well — ROS2, VESC communication, WiFi hotspot, web dashboard. It struggles with real-time SLAM if I go down that road.

The Jetson Orin Nano adds GPU inference — needed if I want SLAM-based navigation or heavy AI object detection. It's ~4× the price.

My plan: start with Pi 5. Upgrade to Jetson if I hit compute limits.

### Vision: OAK-D Lite

The OAK-D Lite is a stereo depth camera with an onboard AI chip (Intel Myriad X). It connects via USB to the Pi/Jetson and outputs:

- Full depth maps (every pixel has a distance value)
- AI object detection with spatial coordinates (X, Y, Z in metres)
- Person detection for follow-me

For obstacle avoidance, I set a ground plane and flag anything protruding more than 5cm above it. Rocks RoBug can drive over are ignored. Stumps, logs, children — RoBug stops.

The 95° horizontal field of view covers the full front arc. Mounted low on the frame below the bed deck so it doesn't interfere with payload.

## The software stack

### ROS2

ROS2 (Robot Operating System 2) is the middleware that connects everything. It's not really an OS — it's a framework where every component runs as a "node" publishing and subscribing to named data streams called "topics."

The OAK-D node publishes obstacle data. The navigation node reads it and publishes velocity commands. The VESC driver reads velocity commands and talks to the motors. Nothing is hardwired together — swap any component without touching the others.

### The full stack

```
Nav2 (path planning, obstacle avoidance)
    ↓ /cmd_vel
diff_drive_controller (translates to per-wheel velocities)
    ↓ wheel RPM targets
vesc_driver (UART to master VESC)
    ↓ CAN bus
Individual VESCs → Motors

OAK-D Lite → DepthAI node → /obstacles → Nav2
VESC telemetry → /odom → Nav2 (position feedback)
Foxglove Studio ← ROS2 topics (telemetry dashboard on phone)
```

### Foxglove Studio

Instead of building a custom dashboard, Foxglove Studio connects to ROS2 over WiFi and streams all telemetry — motor current, voltage, wheel speeds, camera feeds — to a browser. RoBug runs a WiFi hotspot; I connect my phone and open the dashboard. No app needed.

## Follow-me mode

The OAK-D Lite detects a person and tracks their position (X, Y, Z in metres from the camera). RoBug steers to keep them centred at ~2–3 metres distance.

Activated by holding a button in the Foxglove web UI on my phone. Released, RoBug stops.

## Obstacle avoidance

The OAK-D depth map is processed to build a height map of the terrain ahead. Anything taller than ~5cm above the ground plane in the forward arc triggers a speed reduction or stop depending on proximity. RoBug can drive over small rocks but stops for stumps, logs, and anything else significant.

The collision avoidance runs as a separate ROS2 node and has priority over all other commands including follow-me. If the depth camera sees an obstacle, the stop command overrides everything.

## Property boundaries

Rather than implementing full SLAM and GPS-based geofencing (which is the right long-term approach), v1 uses BLE beacons mounted on trees at boundary points. When RoBug detects a boundary beacon's signal strength exceeding a threshold, it stops and waits.

Small Bluetooth beacons — about the size of a coin — can be tucked behind bark so they're invisible. Each costs ~$10–15.

## The prototype plan

Before ordering $1,400 in wheelbarrow motors and four VESCs, I'm building a small prototype with:

- Two salvaged hoverboard hub motors (~$50 for a dead hoverboard on Kijiji)
- Two Flipsky Mini FSESC 6.7 Pro controllers (~$57 each)
- Raspberry Pi 5
- OAK-D Lite
- A simple welded frame

Same software stack. Every line of code transfers directly to the full RoBug build. The prototype validates the architecture before I commit the hardware budget.

## What's next

1. Source a dead hoverboard (motors only needed)
2. Order two FSESC 6.7 Pro controllers
3. Fabricate a simple frame
4. Flash ROS2 Jazzy on the Pi
5. Get vesc_driver talking to both motors
6. Mount the OAK-D Lite and get obstacle detection running
7. Build the Foxglove dashboard
8. Test follow-me in the backyard

Build log continues as parts arrive.
