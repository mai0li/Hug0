+++
cover = "https://cdn-images-1.medium.com/max/800/1*wLGLHAeWMCeRf-2vbboYjQ.png"
date = "2018-07-19T11:18:55-03:00"
title = "Notes on Electra's/@i41nbeer's inject_criticald"
categories = [ "iOS" ]
tags = ["Kernel", "iOS", "Programming", "jailbreak", "exploit"]
draft = false
weight = 2
+++

Understanding how your tweaks get enabled in Electra.
<!--more-->

### The trigger

It all started from this tweet:

{{% tweet 1019536048383250432 %}}

Before some developer wraps this procedure into an useful tweak, I tried to understand what's actually happening there. Luckily, CoolStar had just [open-sourced an Electra1131 build](https://github.com/coolstar/electra1131) a couple hours before Jake's tweet. Thank you, CoolStar!

---

### The magic line

`inject_criticald 1 /electra/pspawn_payload.dylib`

![](https://cdn-images-1.medium.com/max/800/1*6-soJkV8t8HBsGB8U-tS-g.png#center)

Its arguments are pretty simple: take a pid (in our case, pid 1 equals *launchd*) and inject a library into it.

--- 

### Down the rabbit hole

Most subroutines are already described in previously released Ian Beer's exploitations:

> #### Take launchd pid -> get a task port # -> achieve tfp0.

Since we have root access (and now, tfp0), the injection can be achieved.

We proceed to find the start address of *launchd* process by using Ian Beer's binary_load_address (it grabs the address from the second argument of mach_vm_region. Check it at [*triple_fetch*](https://bugs.chromium.org/p/project-zero/issues/detail?id=1247#c3) for a deep understanding).

We've arrived at the first console logging line (Address is at 0000...).

Here's the final *inject_criticald* procedure call:

`call_remote(remoteTask, dlopen, 2, REMOTE_CSTRING(loaded_dylib), REMOTE_LITERAL(RTLD_NOW));`

This procedure uses `dlopen()` function to load our library into *launchd*. The *RTLD_NOW* flag makes sure all undefined symbols in the library are resolved before dlopen() returns.

Somewhere along the call_remote road, the second console logging line warns us about the success of the `find_blr_x19_gadget()` function, which has to do with ROP and the inner workings of (again) Ian Beer's triple_fetch exploit.

> Our loaded_dylib (*/electra/pspawn_payload.dylib*, if you remember well) is successfully injected :)

---
### Why dylibbin'?

Here's a brief look into *pspawn_payload* bin:

{{< figure src="https://cdn-images-1.medium.com/max/800/1*B233CVfARjI4GyeFZANJLg.png" caption="Cool. Let's check where `getpid() == 1` happens." >}}

There it is:

{{< figure src="https://cdn-images-1.medium.com/max/800/1*wLGLHAeWMCeRf-2vbboYjQ.png" caption="Mind the creation of a pthread with the function `thd_func` passed as argument." >}}

{{< figure src="https://cdn-images-1.medium.com/max/800/1*3ODREdZfvbgl4syICEAsew.png" caption="Our exploit now knows how to spawn whatever Daemons it wishes." >}}

The rebindings described allow us to fake spawn Daemons (more specifically, every Daemon found at */Library/LaunchDaemons/*), including our dearest */usr/libexec/cydia/startup*, which activates our tweaks without (luckily!) causing panics/loops that would require a reboot.

Since we're already at a jailbroken state, CoolStar's *jailbreakd* Daemon is already active before these steps --- and thus, it is skipped. The same goes on for *sshd* (OpenSSH).

---
#### Making tweaks appear (Windows' "restart to apply changes")

If we compare Jakes' flow against CoolStar's, we'll see the only diff is the post injecting process to make changes apply: while Jake's only resprings the device, CoolStar's does a wider cleaning by using `ldrestart` as described in his tweet:

{{< tweet 991847869903654913 >}}

That's all, folks. Thank you for reading this far :)

---

Special thanks to:

[Coolstar](https://twitter.com/coolstarorg) and [Electra Team](https://twitter.com/Electra_Team) for an amazing jailbreak tool

[Ian Beer](https://twitter.com/i41nbeer) for amazing iOS kernel research

[Jake James](https://twitter.com/jakeashacks) for provoking my curiosity

[Jay Freeman](https://twitter.com/saurik) for battling jailbreak legality for all these years

[Apple](https://twitter.com/Apple) for the most amazing mobile OS ever
