---
title: "Rethinking the Architecture"
date: 2026-05-21
tags: ["mechanical", "steering", "articulated", "rover", "lego"]
summary: "The skid-steered mars rover concept turns out to have two fatal flaws — structural rigidity and steering. Here's how I worked through both."
---

The last few days have really been about refining the requirements, and properly investigating whether the skid-steered mars rover concept is actually workable.

## The Lego Prototype

I built a number of Lego prototypes, and I found it was a challenge to build the mars rover type vehicle with any sort of rigidity. I also discovered that 6 year old boys are incredible at rooting out any structural problems your design might have. The main issue is that the bogie legs attach to the chassis at one concentrated pivot point. That pivot has to handle all the forces from the wheels acting on it from a distance, producing large twisting moments that are hard to control.

During my research I came across the [Sawppy](https://newscrewdriver.com/category/projects/sawppy-the-rover/) project — an open source mars rover that many [people](https://newscrewdriver.com/category/projects/sawppy-the-rover/) have built and tested. These projects are incredible, and well done to anyone who has built a functional rover. However, what's clear from these projects is that these rovers are not exactly beefy machines designed to carry a lot of weight. You can see what I mean [here](https://newscrewdriver.com/2021/03/05/sawppy-rover-dances-like-real-rovers/).

This makes a lot of sense in the context of something you're sending to Mars, where every kilogram of payload has been relentlessly optimized. In our case of carrying heavy things around on our homestead, I don't think this architecture is up to the challenge.

## Tank Steering is Out Too

The other thing learned from the Lego prototype — as well as from watching many videos of tank turn robots, skid steers, UGVs, and other vehicles — is that while technically possible, tank steering is incredibly jerky, bumpy, hard to control, tough on tires, introduces large bending moments on the structure of the robot, and generally just a less optimal way to get a robot oriented the way you need. This is especially true over rough and rocky terrain, where you might be asking a wheel to magically hop sideways over a rock in order for the vehicle to turn.

I fully anticipate having enough problems controlling the robot autonomously as it is — somehow programming the controller to account for the messiness of a tank turn is going to be too much to ask of my almost non-existent control, autonomous navigation, and coding skills.

So unfortunately tank steering is out.

Which means that we are basically back to the drawing board.

## So What Do We Do?

As I see it there are really only three ways to steer a robot that might actually work:

**Tank steer** — ruled out above.

**Conventional steering** — front, rear, or both sets of wheels rotate about a near vertical axis, like on a road car. This would require a steering knuckle for each wheel we want to steer, complicates the suspension design, introduces a number of new failure modes, and unless we repeat that effort front and rear (no thanks), doesn't give us much of a turning radius. The lack of turning radius might come back and bite us when I'm doing Austin Powers-style 10-point turns trying to unload the firewood behind our house. Mainly because of the mechanical complexity involved, I think this approach is out.

![Austin Powers 10-point turn](/images/austin-powers.gif)

That leaves us with only one other option.

**Articulated steering** — each set of wheels is rigidly connected to their axles or frame members, and the entire vehicle pivots around a center point.

This approach has a number of benefits. The mechanism is simple. It's just a set of bearings bolted onto each half chassis, a shaft connecting them, and a linear actuator that pushes and pulls control arms to articulate the joint. This joint is not space or weight constrained, so it can be designed with a ton of strength. It seems like something I might actually be able to build too!. As a bonus it also produces a dramatically tighter turning radius. Most vehicles built this way can turn 180 degrees within their own length. This approach has also been successfully used by many heavy lift trucks, bull dozers, road rollers, etc., vehicles that need to be very maneuverable and carry heavy things. Sounds like just what we need.

One more thing to note is that the robot could be easily disassembled into two halfs by simple removing the center pivot shaft. This might make it easier to fit into the trunk of our car.

## The Trade-off

The main downside is that articulated steering splits the carrying bed into two pieces that move relative to each other, which could make carrying large items awkward as one of the beds would have to slide relative to the payload to go around a corner. If you've ever been on a bendy bus or streetcar you know how unsettling this can be. For loose materials, like firewood or wood chips, we'd probably need two separate bins that can move independently. Although it might be possible to pack the heavy batteries and electronics entirely into one chassis half, freeing up the other half to be used only for payload.

In any case, this problem seems much more solvable than building a full set of steering knuckles and push rods, and we get a much tighter turning circle by default. Whether that turns out to be true or not is yet to be seen.

## The New Lego Prototype

To test the new concept I built another Lego prototype — you can see it below.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;">
<iframe src="https://www.youtube.com/embed/X1Kj9zFLtOA" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allowfullscreen></iframe>
</div>

I was impressed by how structural sound the vehicle feels, and by how tight it's turning radius is. However, this model is still lacking in one important way: suspension. That'll be the topic of the next update.
