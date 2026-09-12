+++
title = "Couldn't find it, built it"
date = "2026-09-11"
draft = false
author = "Manav"
+++

Last year, there was a cohort of builders where I decided to participate, put in my application that I was gonna build an E-INK display with g-cal integration which would show you your next meeting at a glance.
Since it was E-Ink it would also blend in your environment and need much less power to run.

As projects do, it started with a bang with me printing a case putting the display inside, spending a whole evening building a layout from scratch for the display and then...

Then just like every project started by a builder with undiagnosed ADHD, it went into the special farm where it was going to live happily ever after.

The difference this time was, I had spent money on this and it was taking up real space in my almirah and environment so for a while every time I opened my almirah I thought to myself, I should really do something about that display.

Cut to 2026 I bought a nice bookshelf to keep near my bed and my wall clock in my room had died recently so as one does I started looking for a decent bedside clock, I didn't really need an alarm clock just something that would tell me the time and date. Not that hard to find right?

[Portronics - Too bright for the night](https://www.amazon.in/Portronics-Multi-Function-Temperature-Sensitive-Brightness/dp/B0GSWJJXF4/)

[AERYS - Not bright enough for the day](https://www.amazon.in/AERYS-Warranty-Vintage-Battery-Powered-Decoration/dp/B0GQGXCGC5/)

You get the vibe

Nothing on the market fulfilled my very basic condition - BOLD big legible time, works on battery, doesn't blind me at night

Just as I was about to give up and about to start thinking of something else to fill that space on my nightstand, I opened my almirah and found the unused display and Eureka.

First step was to go online and order an ESP32. Now from my old hardware / robotics days I was about to order one, when I found the ESP32-C3. Same as the ESP32 less than half the size and USB-C and cheaper?? All hail Chinese tech.

So got the ESP32-C3 and took 2 full days to write out the firmware since AI models still haven't caught up to writing proper firmware for embedded systems.

Now began the struggle for moving this off the breadboard -:

### Step 1: Solder everything

### Step 1 Actually: Learn that you have never used a battery in any project before and you always outsourced it to someone who knew how to do it

Undercharging is a thing so you need a charging circuit with undercharging protection, a buck converter, a whole host of things. You can't just put positive and negative of a battery in the ESP and call it a day.

Hence, in the middle of this soldering drama I had to go online and get a TP4056. Found out I really didn't need a buck converter (Sidenote: anyone willing to buy a lightly used buck converter)

Okay. Back to Step 1.

1. I didn't own a soldering iron and I didn't want to buy one just for one project, so I asked my brother to bring (borrow) the one from his office. He brought it yes, but he didn't bring any soldering wire because his office didn't have any??? Nearest date for an Amazon delivery for soldering wire was next week so I gave up on that.

2. Visit the Robotics Lab in my college and get a junior to do it for me (He/She gets the exposure, duh.)

So I did, while I was there I realized (wisdom from the robotics days) I should mount this on a perfboard. You know, if the ESP goes bad replace it off the board, no need to de-solder and re-solder things.

Good idea for Robots / Things that move. Bad idea if you wanna make a static clock that doesn't move and needs to look aesthetic.

So I came back with the perfboard mounted things and for a few days let it sit on my nightstand, worked beautifully but sadly didn't look as beautiful. It was chunky and when I tried to build a case for it, well I didn't want to print a 5cm thick case for an E-ink display, a tiny ESP and a battery.

### Step 2: Remove the perfboard

As I said it was ugly and didn't give [Teenage Engineering](https://teenage.engineering/) made a clock, I removed the perfboard.

I then realized I don't own a soldering iron and I will need to go to college again to get this shit soldered or cave and buy a soldering iron.

Guess who found out Blinkit sells a soldering kit?? A whole ass kit with an iron, wire, flux, stripper (wire) for 200 rupees??

### Step 3 onwards

Order the soldering kit because too lazy to go to college

Spend some time inhaling fumes and also soldering (Ooh I should use that old fan to make an exhaust for a soldering station)

Finally solder it all together. Sigh

At the end of it all I had spent several weeks building this so I double taped everything to the back of the display and called it a day

A few days later, wanted to maybe hang this on a wall so printed a backplate for it.

Wait this has an ESP32 I can make an API call to fetch today's weather

Oh the weather API returns AQI too nice

Finally a bedside clock made by Teenage Engineering.

End of ordeal

PS: Did some optimizations with ESP32's inbuilt deep sleep, calculated power draw and figure out 2000mAh will last me 3-4 months

---

Manav you wrote a whole damn post about the clock what does it look like though. [Here you go](https://x.com/InsaaniManav/status/2098067543726673978)

---

The Tteokbokki (Taboki) At Kori's. Any Kori's. Excellent warm comforting cheesy and very calming when you are trying to build a bedside clock