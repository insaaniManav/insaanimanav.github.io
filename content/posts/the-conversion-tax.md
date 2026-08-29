+++
title = "The conversion tax(This post is about physics I swear)"
date = "2026-08-27"
draft = false
author = "Manav"
+++

Last weekend my printer's Z-axis lead screw was kind of misbehaving and I spent some time debugging it. Once I was done, I asked myself why is it that most actuation we see on a day to day basis is circular by nature.
Every single time, we want to move something linearly (at scale), we end up paying a small tax. The tax of ball bearings, lubricants, gearboxes. Why is it that the actuation gods hate moving sideways      ?

Seriously take a whole minute and think where was the last time you saw linear actuation at scale (Manav, who even notices that??).
.
.
.
.
.
The most common answer would be hydraulics, the JCB outside of your home which needs to lift several tonnes of sand and cement off the ground, uses a naturally linear actuator.

In my research for this post I came across a common pattern of things, anything that needs to hold a lot of weight and might need to do it for a long time, uses hydraulics and hence linear actuators.
Below are a couple of reasons I found -:
* Being able to hold a huge force is much easier, it's just keep the liquid where it is. In case of rotary to linear it means running huge amounts of current to a motor and hoping the gearbox won't snap which brings me to my second point
* It's difficult to predict how much load your gearbox can take, with a hydraulic actuator worst case, the liquid flows out and it slowly retreats, with gear based ones, you never know when the gearbox will start skipping or straight up snapping on you leading to all sorts of unpredictable issues.

Everything else that we see around us being actuated, cars, 2d and 3d printers, the elevator you took this morning. All of them paying the conversion tax. Natural circular rotation converted into linear motion, over and over.

### But why is that?
Don't know for sure, this isn't the kind of thing that sits as a question on Physics Stack Exchange. But from what I read

* A rotor can spin indefinitely in a fixed, compact space — the stator only has to wrap around it once. A linear motor needs stator material (windings + iron) running the entire length of travel, so cost and bulk scale with stroke length. Rotary motors don't have that problem.
* Continuous rotation lets you keep applying force every instant via commutation (brushes, or electronic switching) without ever "running out of track."  - similar point as above
* Bearings for rotation (ball/roller) have far lower friction than the sliding or rolling guides needed for long linear travel.
* Manufacturing a cylindrical winding is easier and more material-efficient than a flat/extended one.

> Fun Fact: Biology has the exact opposite thing happening here. All motion is linear (muscle fibers contracting) converted into rotary

### Do we see linear actuation naturally anywhere else?
* **Maglev trains:** the "stator" is embedded in the track itself running the whole length of the rail, this is why Maglevs are far more expensive than steel rails per KM. Only Japan seems to have nailed it at scale
* **Voice coils:** speakers, every single speaker is a tiny linear actuator powered by an electromagnet
* **Piezo stages:** the strange little crystal which moves when current is applied to its ends. Yup. Linear actuation at scale

Common thread across all four: the stator-length tax only bites when the stroke is long. Short stroke or high value-per-unit-distance is where linear motors win outright due to no conversion tax.

But none of this above helps me, I still gotta pay the tax and make sure my Z-axis lead screw doesn't bind :(

---
No Manav you didn't bore me to death tell me more

[Actuators in robotics](https://www.firgelli.com/blogs/news/the-physics-of-humanoid-motion)
[Difference between linear motors and mechanical devices](https://www.machinedesign.com/mechanical-motion-systems/linear-motion/article/21831837/the-difference-between-linear-motors-and-linear-mechanical-devices)
[Rotary vs. Linear Electric Actuators](https://jhfoster.com/automation-blogs/rotary-vs-linear-electric-actuators/)
[A History of the Steam Engine](https://brewminate.com/of-pistons-and-combustion-a-history-of-the-steam-engine/)

--- 
If you're ever visiting Hauz khas, make sure to try the Ema Datshi at Llama Kitchen. Their service is painfully slow but the cheesy dish is so damn comforting especially in Delhi rains 