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

We're going to have enough problems controlling the robot autonomously as it is — asking the controller to somehow account for the messiness of a tank turn is too much to ask of my almost non-existent control/autonomous navigation/coding skills.

So tank steering is out.

## So What Do We Do?

As I see it there are really only three ways to steer a robot that might actually work:

**Tank steer** — ruled out above.

**Conventional steering** — front, rear, or both front and rear, like on a road car. This would require a steering knuckle for each wheel we want to steer, complicates the suspension design, and unless we repeat that effort front and rear, doesn't give us much of a turning radius. Something that might come back and bite us when I'm doing Austin Powers-style 10-point turns trying to unload the firewood.

![Austin Powers 10-point turn](/images/austin-powers.gif)

**Articulated steering** — each set of wheels is rigidly connected to their axles or frame members, and the entire vehicle pivots around a center point.

Articulated it is. It only requires one pivot bearing and one actuator — things that can both be sourced and bolted onto the robot (hopefully). It also gives us a dramatically tighter turning radius. Most of these vehicles can turn within their own length. Not bad.

## The Trade-off

The main downside is that articulated steering splits the carrying bed into two pieces that move relative to each other, which could make carrying large items like sheet goods awkward. For loose materials like firewood or wood chips we'd probably need two separate bins that can move independently. It might be possible to pack the heavy batteries and electronics entirely into one chassis half, freeing up the other half for load.

In any case, these problems are much more solvable than building a full set of steering knuckles and push rods, and we get a much tighter turning circle by default. We also know that many heavy-duty machines — steam rollers, mining excavators — that work in tight spaces and carry heavy loads successfully use this exact design, so that's a decent vote of confidence.

## The New Lego Prototype

I built another Lego prototype to see how these machines work — you can see it below.

<!-- YOUTUBE VIDEO PLACEHOLDER -->

This model is still lacking in one important way: suspension. That'll be the topic of the next update.
