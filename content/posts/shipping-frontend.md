+++
title = "Rectangles and Lines"
date = "2026-07-05"
draft = false
author = "Manav"
tags = ["ai"]
aliases = ["/posts/shipping-frontend-with-claude-code/"]
+++

I am a backend engineer. I have written maybe four hundred lines of production React in my life, most of them poorly stitched-together code. Last month, I was asked to ship a full frontend feature at work.
This is the story of how that went, and what I learned about working with a Coding Agent on a stack I have no business writing.

The idea behind the feature was simple. We were analyzing hundreds of millions of datapoints, out of which a small fraction were actually alerted about. All the customers ever saw were the alerts, not the actual work happening behind the scenes.

Second, even the alerts that we were detecting were being shown as individual rows in a table, which made it harder to see the patterns emerging in the data.

So we decided to build a single pane of glass where a customer could monitor their entire pipeline — the number of alerts, alert patterns, and whatnot.

The backend was engineered as a simple ingestion pipeline into a columnar DB. The frontend was, well, just a set of boxes with lines. Right?

Turns out those boxes and lines had opinions. About width, about design tokens, about what happens when you resize a browser. Here's what I learned trying to convince an LLM to see what I was seeing.


## Spatial awareness

- "Add 3 boxes in the first section of the page showing three different aggregate numbers. Width equally divided."
  - Three boxes appeared. Two of them broad, the third tiny.
- "Make sure the 3 boxes have equal width in proportion to the screen length."
  - Three boxes appeared, equal width yes, but only occupying half of my screen.
- "Make sure the boxes occupy the entirety of my screen."
  - Three boxes appeared, equal width, filling the entire screen on my ultrawide. The moment I made the browser smaller, a horizontal scroll appeared.
- Okay, no more AI. Let me actually read the code and understand which React framework we use for grid layouts.
  - Okay, this isn't that hard actually. Let me build this part of the layout myself.

**Conclusion**: AI has no spatial awareness. You can't ask it to build things in terms of what you see; you need to talk in numbers.

## Design systems

- "Okay, now the top layout is done. Add 2 boxes on each row with the following headings."
  - Layout actually looks okay this time. Hmm, maybe I can trust AI with building layouts.
  - Look inside.
    - No grid. Top 2 boxes 10rem width each, good. Second layer some value in px. Third layer no measurement, just numbers and vibes for width.
    - I'm not a frontend engineer, but even I know that can't be correct.

**Conclusion**:
AI doesn't seem to be good at following existing design systems. For backend work I have learned to prompt it by default to use existing utils. Even if I don't for AI it's kind of a friction to create a new util each time.

But for frontend, I think it's too easy for the model to add a new type of size measurement as long as it fits. New font size. New way to measure. And so on. Every new component introduced a new way of measuring things — some in rem, some in px, and never the actual design system.

For AI, the goal was the final layout. I had to struggle a lot to get it to use our existing standard measurements inside our theme package.

## Responsive layouts

- "Okay, I have the entire layout, phew. Let me quickly verify how it looks on literally any other screen except my ultrawide."
  - Whoops. Collapsed into a sad pile of rectangles almost immediately.

**Conclusion**:
This last lesson was kind of a biggie and took the most time to fix — I had to restructure most of the layout myself.

AI also isn't good at responsive layouts. 

## How I work around all three

I still don't especially enjoy making boxes to display JSON that a user could have read themselves, but the job needs doing, so I need to work with AI to build frontends. At least for the time being. Below are some solutions I've found that work for me. These will of course change as models evolve.

- **For layouts**: most layouts are pretty standard in nature. Just tell the model to copy the topBar layout from an existing page, change the names, and go from there.
- **For components**: again, point at a file that seems to work and ask the model to follow the same level of granularity when breaking things down into components.
- **For responsiveness and design systems**: no clean solution except a fixed set of instructions or best practices — Claude.md or whatever version your agent likes to read from.

## What AI was immediately good at

Now that I've berated coding agents this much, I have to give credit where credit is due.

It was outright amazing at API integration. Single-shotted the entire thing. Had to make minor tweaks here and there, but nothing major.

State management too — and I have to admit I know little to no frontend state management, or how to use it, or what it even is (I hope my employer doesn't read this post). AI single-shotted that as well.

Loading states aligned in the right places. Text debouncing added in search fields. Enter key triggering search by default. All the small things, handled outright. Sane defaults everywhere.

## What I actually learned

As an engineer, I think the biggest thing separating a junior engineer from a staff engineer is foresight. 

Fresh out of college, your goal is to make it work, and make it work right now, maybe for the next few days. 

A staff engineer knows how to make it work right for years, in a single pass.

That intuition for foresight is exactly what AI is lacking. 

It has no situational awareness, no concept of time, no memory of what happened before or intuition about what happens next . At least out of the box. It's a junior engineer who happens to know every syntax rule ever written.

Which is why frontend was the pitfall it was. Somewhere between the third grid layout and the fourth attempt at adjusting graph axes, it hit me: frontend isn't just a set of rectangles. It's how those rectangles talk to each other. Which one scrolls, which one collapses, which one holds its ground when the browser shrinks. That's the part AI doesn't see yet, because seeing it requires knowing what the layout is for, not just what it looks like.

Finally no, I'm not a frontend engineer now. I shipped a frontend feature with a lot of help, and I learned about the parts of the job I used to underestimate. Which is probably enough for one project.

---
Unrelated: the Classic Pepperoni at Leo's Pizzeria in Vasant Vihar. Highly recommended. Buffalo mozzarella keeps it creamy and the Spanish chorizo carries the whole thing.
