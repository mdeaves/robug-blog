---
title: "Calculating Joint Forces"
date: 2026-07-10
tags: ["design", "structural", "analysis", "articulation"]
summary: "Working out the forces on RoBug's central articulation joint using free body diagrams."
---

To properly size the articulation joint, I need to understand what forces it will experience. The first step is drawing a free body diagram of the full vehicle and solving for the wheel reaction forces.

![Full Robot Free Body Diagram](/images/full-robot-fbd.svg)

Here is the full vehicle. The joint in the middle is represented by 3 ball joints forming the triangle. You can see the basic set-up of the joint here [in this post](/posts/articulation-joint-model/).

The vertical forces all have to sum to zero so we can write,

$$W_r + W_f = F_r + F_f$$

And we can sum moments around any point to get a second equation. Let's sum the moments around the Rear Half COG to get,

$$F_r \cdot X_r + W_f \cdot (X_f - X_r) = F_f \cdot (L - X_r)$$

Solving by substitution, the front wheel reaction is simply:

$$F_f = \frac{W_r X_r + W_f X_f}{L}$$

and the rear follows from the first equation:

$$F_r = W_r + W_f - F_f$$

This has a nice intuitive interpretation — each wheel reaction is just the sum of each weight multiplied by its distance from the opposite wheel, divided by the wheelbase.
