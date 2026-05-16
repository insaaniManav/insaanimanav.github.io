+++
title = "Why all your servers should have an 8gb empty file"
date = "2021-04-15"
draft = false
author = "Manav"
+++

A wise man once said:

> Nothing good happens when the storage space hits 0% on a linux machine

I completely agree with this statement. Whenever I have seen 0% storage on a linux server, all logic flies out the window and panic just comes flowing in. Some essential commands stop working and everything seems haywire just in general.

This is where the `dd` command comes in. Using `dd` I always try to put an empty `space.img` file in the root of my server — whenever storage on my machine accidentally runs out I just delete that file to buy some time to figure out where it went wrong.

```bash
dd if=/dev/zero of=/space.img bs=1M count=8192
```

And that is why all your machines should also have an 8gb empty file.
