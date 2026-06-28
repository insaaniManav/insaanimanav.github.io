+++
title = "Go1.26.0 Release Notes - The Green Tea GC"
date = "2026-06-28"
draft = false
author = "Manav"
tags = ["go"]
+++

This post has been in drafts slowly building up for the long time as I navigate the strange world of go runtime, reading go that looks like go runs go but isnt really go in the much deeper sense ? But then what goes inside the go runtime as my programme goes ?
This post is about a specific piece of that runtime: the new Green Tea garbage collector that shipped with Go 1.26 in February.

But before we get into into we have a couple of primers to build up with 

## Heaps and Stacks of Garbage

Whenever we write a program, all the data has to live somewhere in memory. That somewhere is split into two regions. the stack and the heap.

The stack is simple. Each function gets its own chunk of memory when it runs. Local variables, function arguments, all on the stack. When the function returns, that chunk is gone.

The heap is everything else. When you allocate something that needs to outlive the function that created it. A slice you pass around, an object returned from a constructor, a global variable it finds its place on  the heap. The heap is shared across the whole program and things on it stay alive until... well, until something decides they're no longer needed. That's the problem. The stack cleans itself up. The heap doesn't.

Different languages handle this differently. C gives you `malloc` and `free`. You manage the heap yourself. You forget to call `free`? Memory leak. You call it twice? Crash. Total control, total responsibility. Python manages the heap for you using reference counting. every object knows how many things point to it, and when that count hits zero, it gets cleaned up. Plus a cycle detector for when things point at each other in a loop. Go also manages the heap for you, but uses a garbage collector that periodically walks the entire object graph to figure out what's reachable and what isn't.

#### Fun Fact
The interesting thing about Go is that the compiler tries hard to keep things off the heap in the first place. It does something called escape analysis. If it can prove a variable won't outlive its function, it stays on the stack, no GC involved. But anything that escaped. anything that has to be handed around or live longer than its creator becomes the GC's problem.

## Go's Memory model

Now that we have some idea of stacks and heaps, Let's talk about how golang manages memory

We are not going into how a stack is managed, seperate post and discussion altogether but this post is about the Garbage collector so lets see how a heap is managed.

In go a heap contains arenas, Arenas are 64MB chunks of memory available for allocation.

Each Arena contains pages, each page is 8KB. One or more of those pages grouped together for objects of a single size is called a span. Every object inside a span is exactly the same size, and that matters quite a bit as we'll see.

## Go's Default garbage collector

Go's Garbage collector uses a simple approach in phases

### Mark phase
It starts at the *roots*. Goroutine stacks, global variables. Things the program is definitely using right now.

From there it has a work buffer, a queue of objects it knows are alive but hasn't looked inside yet. Pops one off. Okay, this object is alive, we know because something was already pointing to it and that's how we got here. Now what does it point to? It has 4 pointer fields, let me check each one. This one is already marked, skip it. This one isn't, okay mark it alive and push it onto the buffer.

Pop, scan, mark what you find, push those on, repeat until the buffer runs dry. Anything that never got marked, nothing ever pointed to it so we never reached it, that's your garbage. The mark phase figures out what's alive. Garbage is everything left over.

### Sweep Phase
Each span has a mark bitmap(Literally just a set of bits representing an object), one bit per object slot. Bit is set? Alive. Not set? Gone. Sweeper goes span by span, frees up the unmarked slots, and the allocator can put new objects there next time around.

#### Fun Fact
If your goroutine is allocating memory faster than the GC can mark things, Go makes your goroutine stop and do some marking work itself like a little punishment before the allocation goes through. You want to make more garbage? Fine, but you're helping clean up first. These are called GC assists and they're basically Go's way of making sure allocation never outruns collection. Your innocent little `make([]byte, 1024)` might be secretly doing GC work before it hands you that slice.

## The new cup of Green Tea

So here's the actual problem with the classic GC. The work buffer is a stack, LIFO, last in first out. You mark an object, push its neighbours, pop the next one, and that next one is probably somewhere completely different in memory. The pointer chasing is essentially random. Every pop is likely a cache miss. Your CPU has to go to RAM, which is about 100x slower than reading from cache, and the Go team found that more than 35% of mark time is just the GC sitting there stalled waiting for memory to arrive.

It's also getting worse. Modern CPUs have more cores but relatively less memory bandwidth per core. NUMA architectures mean some memory accesses are slower depending on which core you're on. The classic mark algorithm was fine when none of that was really a problem.

Green Tea's answer is almost embarrassingly simple. Instead of tracking individual objects on the work buffer, track spans.

When the GC finds a pointer to an object, instead of immediately queuing that object to chase next, it just sets a bit in the span's bitmap and queues the span if it isn't already there. That's it. Move on.

The span sits in a FIFO queue (not LIFO, this is important) while the GC keeps doing other work. More pointers to objects in that same span get discovered. More bits get set. When the span finally comes off the queue, it might have 10, 20, 30 objects marked inside it. Now scan all of them together, in order, on one warm page.

One cold trip to pull the page into cache, shared across dozens of objects, instead of one cold trip per object. That's the whole bet.

The FIFO part matters quite a bit too. LIFO would process the span almost immediately after queuing it, before neighbours have had a chance to accumulate. FIFO lets it sit and collect more, which is why Green Tea uses a queue for spans and not a stack.

To make this work, Green Tea needs two bitmaps per span instead of one. The classic GC had one mark bit per object. Green Tea has two: a "seen" bit (we found a pointer to this object) and a "scanned" bit (we've actually looked inside it and followed its pointers). Grey objects are seen but not scanned. The GC computes the difference at scan time to figure out exactly which objects in the span still need work.

And for spans that are dense enough, Go 1.26 on newer x86 hardware uses AVX-512 vector instructions to process the whole span's metadata in just a few CPU instructions. This wasn't possible with the old approach where you were jumping between random objects of all different sizes. Same span, same size, predictable layout. Vector hardware loves that.

## So did it work ?

Yes, according to the [Go 1.26 release notes](https://go.dev/doc/go1.26) the team expects somewhere between 10 to 40% reduction in GC overhead in programs that make heavy use of the garbage collector. Worth clarifying though: 10% is the modal improvement, most programs land there. The 40% end is for workloads that were already spending a lot of CPU in the GC. If your program barely touches the GC, you won't see much.

The Go team had [already rolled it out at Google](https://go.dev/blog/greenteagc) before it shipped to everyone and saw similar results at scale, which is a reasonable signal that the numbers aren't just benchmark noise.

On newer hardware you also get an additional ~10% on top from the vector instructions path.

Green Tea was available as an experiment in Go 1.25 and shipped as the default in Go 1.26 in February this year. If for some reason you want the old behaviour back, `GOEXPERIMENT=nogreenteagc` at build time, though that option is expected to go away in Go 1.27.

## Should I upgrade to Go 1.26 just for this ?

Probably not *just* for this, but you should upgrade anyway.

If you're already on Go 1.25, you can try Green Tea right now with `GOEXPERIMENT=greenteagc go build ./...` and see what it does for your specific workload before committing to anything.

If you're on an older version, Go 1.26 has a lot more going on beyond the GC: cgo calls are ~30% faster, the `go fix` command got a complete overhaul, there are some language changes, and small allocations under 512 bytes got their own size-specialised routines, more on that in maybe a seperate post. The GC is probably not the headline reason to upgrade but it's a nice addition on top of everything else.

#### Fun fact
The name comes from Austin Clements, one of Go's runtime engineers, who worked out the prototype while cafe crawling in Japan in 2024 drinking a lot of matcha. The seeds of the idea actually go back to 2018 and took years of dead ends to get here. Everyone on the team apparently thinks someone else had the original idea. [(source)](https://go.dev/blog/greenteagc)

## My take
What I find interesting about Green Tea isn't the algorithm itself, it's that the insight is basically "stop doing work in random order, do it in batches." The GC was already correct. It just had terrible cache behaviour. The fix wasn't a smarter algorithm, it was a more hardware-aware one. As some who writes programmes that run on layersssss of abstractions this was an interesting thing to think about

The Go runtime has always been this strange thing to me where it's Go code but also not really Go code, it implements the language but can't fully use the language itself. Green Tea fits right into that strangeness. A garbage collector that batches its own work to be kinder to the hardware running it, a program thinking about how it uses memory, inside the program that manages memory for everything else.

---
But wait, The go team already wrote a blog about this [detailing it and its not even that complex](https://go.dev/blog/greenteagc), why another blog post about it ? Yes I know I forgot to check if there was an official blog post before I went down the go runtime rabbit hole ¯\_(ツ)_/¯


No Manav I love learning about this and definitely not sleepy now 
### Related reading

[A Guide to the Go Garbage Collector](https://go.dev/doc/gc-guide) — if you want to go deeper on how the GC is tuned and what GOGC and GOMEMLIMIT actually do

[Getting to Go: The Journey of Go's Garbage Collector](https://go.dev/blog/ismmkeynote) — a 2018 talk by Rick Hudson on the history of Go's GC, interesting to read knowing Green Tea was already being thought about around the same time
