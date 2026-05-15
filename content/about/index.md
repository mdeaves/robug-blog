---
title: "About RoBug"
date: 2026-05-15
layout: "single"
---

## What is RoBug?

RoBug is a smart 4WD electric utility carrier I'm building for our rural homestead in Ontario. It's designed to haul heavy loads across rough terrain — stumps, firewood, tools, sheet goods — and eventually follow me around the property autonomously, carrying whatever I'm working with. I will use it to plow snow, Zamboni the ice on our pond, power tools when I'm away from the house, and pull the kids on their toboggans.

## Why build it?

Commercial utility carriers exist, but nothing quite hits the combination of price, capability, and hackability I wanted. More importantly, this is a chance to build something genuinely useful while learning a stack I've wanted to dig into for years — Li-Ion batteries, brushless DC motors, motor control, computer vision, autonomous navigation.

## The (current) hardware plan

- **Motors:** 4× wheelbarrow hub motors
- **Steering:** Tank turn
- **Controllers:** 4× VESC motor controllers
- **Battery:** 48V 200Ah LiFePO4
- **Brain:** Raspberry Pi 5
- **Vision:** OAK-D Lite depth camera for obstacle avoidance and person tracking
- **RC Control:** Flysky GT5 pistol-grip transmitter
- **Safety:** Hardware e-stop, power-off brakes on motor shafts
- **Max. Payload:** 250 kg
- **Max Speed:** 10–15 km/h
- **Overall Dimensions:** 100cm × 89cm (fits in the back of a Model Y)

## The prototype first

Before committing to the full build, I'm building a small prototype using salvaged hoverboard motors, two VESC controllers, a Raspberry Pi, and an OAK-D Lite. Same software stack, smaller scale.

## About me

I'm a mechanical engineer based in Ontario. This is a personal project — part homestead utility, part robotics education, part excuse to have fun with the kids. Follow along as I figure it out.
