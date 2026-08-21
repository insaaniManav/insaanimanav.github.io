+++
title = "A homelab in two acts"
date = "2026-08-16"
draft = false
author = "Manav"
+++

The title tells you what you need to know about the contents of this blog. The internet (at least the part of the internet where I live) is chock full of homelab guides / personal blogs on homelab setups.
Almost always proxmox, homeassistant, xeons running in basements. Well, this is another one of them

## Act 1
My tryst with building a homelab started back when the Raspberry pi 4 came out. COVID was raging and I was a bored college kid. I bought one, had a couple of old HDDs lying around, hooked all of them up. Started running Nextcloud and Jellyfin on it. 

Also ran Pi-hole (someone told me if you don't run it on your homelab, you can expect a call from law enforcement)

That was also my first playground with the blue whale boxes, nginx, VPNs, tailscale and just raw sysadmin work. 

I remember running Arch Arm on this, manually compiling Docker Images for ARM because half of what I needed didn't exist. Spending whole days just tinkering with the setup. 

### How was it 

I mean, the entire thing was 2 old external HDDs connected via USB, sitting inside a literal shoebox, every 24 hours. One of the HDDs would randomly disconnect and everything would go down. I'd just cycle the power and that was normal operations.

It was a flimsy setup but that is also why it taught me a lot of what I needed to know about sysadmin things and not just in a way that a course teaches you. 

But practically, real world stuff on why servers need NAS grade HDDs. Why can't I just connect server disks via USB and stream stuff off them. 

Eventually I got my first paid internship. The time went. The shoebox stopped being interesting. And that was Act 1.

## Act 2
Well a few years passed, I got a job and my phone started filling up with 4 years worth of memories, photos and trip pictures. 

I realized I do need to sync photos somewhere. I can't just keep buying phones with bigger storage every few years.

And well Google was doing Google things. Every few months another product on killedbygoogle.com. 
Every few weeks another feature quietly repriced or removed. 
And more recently, [training Gemini on your personal photos](https://www.howtogeek.com/people-think-google-is-feeding-their-photos-to-gemini-but-the-reality-is-worse/) 

Here's where the old me would have gotten the latest Raspi, and set up a Nextcloud in the shoebox again.

The new me decided to pay Hetzner to host them for me.

That is the whole Act 2. The Raspberry Pi era was about learning to run infrastructure. 
This one is about not wanting to. 

I don't have the patience for a disconnecting HDD anymore. 
I don't want to wake up on a Sunday and find out my photos are inaccessible because I chose the wrong USB cable in 2020. 
I want the photos backed up, the ads blocked, the documents scanned, and I want to think about it approximately once a quarter when the bill renews.

I then saw this project called Immich and figured this seems like a nice google photos replacement. Looked around a bunch for hosting solutions but nothing was more private and cheaper than Hetzner so ended up just landing here.

### What I finally decided to run on my Home Lab (in Europe) :

* Pi-hole + Wireguard (whoop whoop that's the sound of da police)
* Immich
* Paperless - NGX

### Hardware
* Arm64 Box - 2 VCPU - 4GB Ram - Hetzner calls this CAX11 - 500ish rupees a month 
* Something called a storage box? - Not fully sure what it is but it gives me 1TB of network attached high speed storage which is good enough for my use case - 300ish rupees a month 
* Total cost per month - 800 rupees

What this got me was a nice private VPN in a continent with decent data privacy laws. Adblocking + 1TB of photo + document storage. Not too bad of a deal if you ask me.
Spent like half a day setting this up via a reproducible Ansible playbook incase I had to replace the machine.

Oh also did I mention the storage box comes with 10 free snapshots rotating themselves starting with the latest ?

Although let's see how long this setup lasts with the current rising RAM and Storage prices. For now, shoebox is still on the shelf somewhere. It'll probably stay there.





