---
title: "Rethinking the Architecture"
date: 2026-05-21
tags: ["mechanical", "steering", "articulated", "rover", "lego"]
summary: "The skid-steered mars rover concept turns out to have two fatal flaws — structural rigidity and steering. Here's how I worked through both."
---

The last few days have really been about refining the requirements, and properly investigating whether the skid-steered mars rover concept is actually workable.

## The Lego Prototype

I built a Lego prototype and it was a challenge to build the mars rover type vehicle with a decent amount of rigidity. The problem is that the bogie legs attach to the chassis at one pivot. That pivot has to handle all the forces from the wheels acting on it from a distance, producing large twisting moments that are hard to control.

During my research I came across the [Sawppy](https://newscrewdriver.com/category/projects/sawppy-the-rover/) project — an open source mars rover that many [people](https://newscrewdriver.com/category/projects/sawppy-the-rover/) have built and tested. These projects are incredible, and well done to anyone who has built a functional rover. However, what's clear from these projects is that these rovers are not exactly beefy machines designed to carry a lot of weight. You can see what I mean [here](https://newscrewdriver.com/2021/03/05/sawppy-rover-dances-like-real-rovers/).

This makes a lot of sense in the context of something you're sending to Mars, where every kilogram of payload has been relentlessly optimized. In our case of carrying heavy things around on our homestead, I don't think this architecture is up to the challenge.

## Tank Steering is Out Too

The other thing learned from the Lego prototype — as well as from watching many videos of tank turn robots, skid steers, UGVs, and other vehicles — is that while technically a solution, tank steering is incredibly jerky, bumpy, hard to control, tough on tires and the structure of the robot, and generally just a less optimal way to get a robot oriented the way you need. Especially over rough terrain, where you might be asking a wheel to magically hop sideways over a rock in order for the vehicle to turn.

I'm going to have enough problems controlling the robot autonomously as it is — asking the controller to somehow account for the messiness of a tank turn is too much to ask of my almost non-existent control, autonomous navigation, and coding skills.

So tank steering is out.

## So What Do We Do?

As I see it there are really only three ways to steer a robot that might actually work:

**Tank steer** — ruled out above.

**Conventional steering** — front, rear, or both sets of wheels rotate about a near vertical axis, like on a road car. This would require a steering knuckle for each wheel we want to steer, complicates the suspension design, and unless we repeat that effort front and rear (no thanks), doesn't give us much of a turning radius. Something that might come back and bite us when I'm doing Austin Powers-style 10-point turns trying to unload the firewood in between our house and the hill behind it. For mainly the mechanical complexity involved, I think this approach is out.

![Austin Powers 10-point turn](/images/austin-powers.gif)

That leaves us with only one other option.

**Articulated steering** — each set of wheels is rigidly connected to their axles or frame members, and the entire vehicle pivots around a center point.

This approach has a number of benefits. The mechanism is just a set of bearings bolted onto each half chassis, and an actuator that pushes and pulls control arms to articulate the joint. This seems like something I might actually be able to build. It also gives us a dramatically tighter turning radius. Most of these vehicles can turn within their own length. This approach has also been successfully used by many heavy lift trucks, bull dozers, road rollers, etc., vehicles that need to be very maneuverable and carry heavy things. Sounds like just what we need.

## The Trade-off

The main downside is that articulated steering splits the carrying bed into two pieces that move relative to each other, which could make carrying large items like sheet goods awkward as one or both of the beds would have to slide relative to the payload to go around a corner. For loose materials like firewood or wood chips we'd probably need two separate bins that can move independently. It might be possible to pack the heavy batteries and electronics entirely into one chassis half, freeing up the other half for load. That way the load ever only need interact with one hafl chassis.

In any case, these problems seem much more solvable than building a full set of steering knuckles and push rods, and we get a much tighter turning circle by default. Whether that turns out to be true or not is yet to be seen.

## The New Lego Prototype

To test the new concept I built another Lego prototype — you can see it below.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;">
<iframe src="https://www.youtube.com/embed/X1Kj9zFLtOA" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allowfullscreen></iframe>
</div>

However, this model is still lacking in one important way: suspension. That'll be the topic of the next update.
