---
title: "About RoBug"
date: 2026-05-15
layout: "single"
---

## What is RoBug?

RoBug is a 4WD electric utility carrier I'm building for our rural homestead in Ontario. It's designed to haul heavy loads across rough terrain — stumps, firewood, tools, sheet goods — and eventually follow me around the property autonomously, carrying whatever I'm working with.

My son Charlie named it. He's six, he's into karting, and he thinks the robot should talk. He's not wrong.

## Why build it?

Commercial utility carriers exist, but nothing quite hits the combination of price, capability, and hackability I wanted. More importantly, I'm a software engineer and this is a chance to build something genuinely useful while learning a stack I've wanted to dig into for years — ROS2, motor control, computer vision, autonomous navigation.

## The hardware plan

- **Motors:** 4× wheelbarrow hub motors (2,000W, 48V, integrated planetary gearbox and tyres)
- **Controllers:** 4× VESC (Vedder Electronic Speed Controller) — open source, FOC, CAN bus coordinated
- **Battery:** 48V 200Ah LiFePO4 with 500A continuous JK BMS
- **Brain:** Raspberry Pi 5 or Jetson Orin Nano (TBD based on compute needs)
- **Vision:** OAK-D Lite depth camera for obstacle avoidance and person tracking
- **RC backup:** Flysky GT5 pistol-grip transmitter — always available regardless of Pi state
- **Winch:** 5,000 lb front-mounted drum winch for stump pulling
- **Safety:** Hardware e-stop, power-off electromagnetic brakes on motor shafts, VESC signal-loss failsafe

## The software plan

- **ROS2 Jazzy** — the backbone of everything
- **vesc_driver** — VESC telemetry and velocity commands
- **diff_drive_controller** — translates cmd_vel to per-wheel speeds
- **Nav2** — path planning and obstacle avoidance
- **DepthAI** — OAK-D Lite integration for obstacle height detection and person tracking
- **Foxglove Studio** — telemetry dashboard streamed to phone over WiFi hotspot
- **BLE beacons** — property boundary markers

## The prototype first

Before committing to the full build, I'm building a small prototype using salvaged hoverboard motors, two VESC controllers, a Raspberry Pi, and an OAK-D Lite. Same software stack, smaller scale. Validate the code before spending thousands on hardware.

## About me

I'm a software engineer based in Ontario. This is a personal project — part homestead utility, part robotics education, part excuse to spend time in the shop. Follow along as I figure it out.
