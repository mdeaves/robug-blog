---
title: "Drivetrain Concept"
date: 2026-05-18
tags: ["mechanical", "suspension", "drivetrain", "hub motors", "rover"]
summary: "Working out the power requirements, then evaluating every drivetrain and suspension concept — from tracks to Mars rovers — before landing on a 6WD rocker-bogie derivative."
---

## Starting with the numbers

Before choosing a drivetrain, I needed to know how much power to drive it with.

RoBug's specs call for a 250 kg payload capacity. I estimate the unloaded machine will weigh around 200 kg — so all-up loaded mass is **450 kg**. The terrain requirement is a **20% slope** (roughly 11°).

Running the numbers on traction power for a wheeled vehicle:

```
F = m × g × sin(θ) + m × g × Crr × cos(θ)
```

Where rolling resistance coefficient (Crr) for rough terrain is around 0.15. With a 20% slope and some margin for acceleration and poor ground conditions, you land at roughly **6,000 W of traction power**. That's the target.

Now — how do you deliver that to the ground, across Canadian Shield terrain, with a machine you can actually build in a garage?

## Option 1: Tracks

![A tracked utility carrier climbing over a log in the woods](/images/skid-steer-robot.jpg)

Tracks are just plain cool. They are also work really well. They distribute ground pressure over a large area, provide excellent traction, and handle soft ground far better than wheels. The Ferocarrier I looked at in the first post uses tracks precisely because of this.

But the problems are real. Building a functional track system from scratch — idlers, drive sprockets, track tensioners, the track itself — is a significant fabrication challenge. And once built, tracks are maintenance-intensive. Mud and debris pack into the drive mechanism, and the tracks get knocked off the drive wheels often if not properly designed.

For these reasons, unfortunately, tracks are out.

## Option 2: Fixed-wheel 4WD skid steer

The simplest wheeled option: four wheels, each with its own motor, no suspension. Turn by running one side faster than the other — classic skid steer, the same principle used by tanks and countless small robot platforms.

![A 4WD skid steer wheeled robot — compact, flat, no suspension](/images/tracked-carrier.jpg)

Lots of robot rovers use this successfully. It's cheap, easy to build, and easy to control. The problem is that without suspension on uneven terrain, the rigid frame means only three of the four wheels are in firm contact with the ground at any time. One wheel is always bridging a gap or floating over a rock. On the Canadian Shield, that's most of the time.

Three-point contact isn't the end of the world, but it limits traction. We can do better.

## Option 3: ATV-style independent suspension

If you want proven off-road performance, look at what ATVs and UTVs do.

![A UTV with independent suspension navigating a rocky trail](/images/atv-suspension.jpg)

Independent suspension at each corner — coilovers, control arms, uprights — gives you great wheel travel and keeps tyres planted on uneven ground. It's a tested, mature design.

The problem is the mechanical complexity. Control arms, uprights, bearings, spring perches, shock absorbers — each of these components needs to be designed, fabricated, and aligned correctly. It's doable for a professional fabricator. For a garage build, it's a substantial scope creep risk.

## Option 4: Rock crawler 4-bar suspension

Rock crawlers use a simpler suspension than ATVs — a 4-bar linkage at each corner rather than independent control arms. These vehicles are genuinely impressive off-road.

![A rock crawler Jeep flexing its suspension over large boulders](/images/rock-crawler.jpg)

The 4-bar is simpler to fabricate. But rock crawlers need axles to transmit power to the wheels. If I'm going to use hub motors — which at this point I think makes a lot of sense — I don't need axles. But the 4-bar suspension *uses* the axle housing as a structural element, constraining the wheel to move vertically. Without an axle, I'd need to build a fake structural axle just to mount the hub motors and give the suspension something to locate against.

## Slow Speed, Independent, Off-Road Suspension with Hub Motors?

What vehicles are designed specifically for hub motors, off-road terrain, and low speeds?

The Mars Rovers.

![NASA Mars Exploration Rover — 6-wheeled rocker-bogie suspension](/images/nasa-mars-rover.jpg)

Curiosity and Perseverance use a **rocker-bogie suspension** system that keeps all six wheels in constant contact with the ground regardless of terrain. The geometry is elegant: two rockers on either side, connected at a pivot point on the chassis, with a bogie at the rear of each rocker carrying two wheels. The front of each rocker has a single wheel.

![Rocker-bogie suspension geometry keeping all wheels in contact as the vehicle crosses uneven ground](/images/bogie-diagram.png)

The catch is that pivot point. Because the two rockers pivot independently, and the chassis is only supported at those two pivot points, there's nothing to constrain the pitch of the chassis relative to the rockers — the chassis would just flop forward or backward. NASA solves this with a differential in the pivot connecting the two rocker shafts. The differential transmits chassis pitch between both sides and provides the third constraint.

That differential is not something I want to fabricate.

## The ESA solution: 3-bogie suspension

Fortunately there is another way!

![ExoMars rover with 3-bogie suspension on Martian terrain](/images/exomars-rover.jpg)

The **3-bogie system** uses three independent bogies — one on each side of the vehicle, and one at the rear — each carrying two wheels (six total). The chassis mounts to all three bogies through pivoting connections. The European Space Agency took this approach with the ExoMars rover.

The rear bogie is rotated 90° relative to the side bogies. Together, all three bogies constrain the chassis in roll *and* pitch — no differential required.

![3 bogies conceptual design — two side bogies and one rear bogie rotated 90°](/images/3-bogie-concept.png)

Three bogies, six wheels, six hub motors, full terrain contact. No springs, differentials, tracks, or anything too copmlicated to manufacture. This could work.

## Why no springs?

One thing puzzled me at first: rock crawlers and ATVs need springs and dampers, but Mars rovers don't. What's going on?

I think it comes down to two things: **speed** and **passengers**.

Springs and dampers exist to manage energy. When a wheel hits a rock at speed, the spring stores and releases that energy gradually instead of transmitting it as a shock through the chassis. The damper bleeds off the energy so the chassis doesn't bounce. At low speeds, the energy involved is small enough that passive geometry can handle it without dedicated spring elements.

The other reason is that Mars rovers don't carry humans. Suspension on a vehicle with a driver needs to protect the driver from vibration and shock. RoBug carries rocks and firewood. Payload doesn't care.

## Power: 6 × 1,000 W

With six driven wheels, I need each motor to deliver 1,000 W to meet the 6,000 W traction target. A 1,000 W hub motor per wheel gets me exactly there.

Six hub motors is a clean fit. No gearboxes, no chains, no sprockets. We'll dive into selecting motors in a later post.

## Steering: tank steer, not corner servos

There's one significant way RoBug diverges from the NASA/ESA rover design: steering.

Every six-wheeled Mars rover steers by pivoting the corner wheels around their vertical axis — active steering servos in the corner hubs. This gives precise, low-scrub turning. It also means four steering actuators and pivot bearings at each corner.

I'm not building that.

Instead, RoBug will **tank steer**. It's less precise and causes some tyre scrub but it requires zero additional hardware beyond the drive motors I already need.

To reduce scrub, I'll design RoBug with a **short wheelbase relative to its track width**. A more square footprint reduces the turning radius and the scrub distance on the outer wheels. The prototype will let me measure how much scrub actually happens on different surfaces before I commit to the full RoBug dimensions.

## Where we are

The mechanical concept:

- **6WD, 6 hub motors** — 1,000 W each, 6,000 W total traction power
- **3-bogie suspension** (ExoMars-derived) — all six wheels in constant ground contact, no differential, no springs
- **Tank steering** — differential drive, no corner servos
- **Compact L/W ratio** — to keep tank-steer scrub manageable
