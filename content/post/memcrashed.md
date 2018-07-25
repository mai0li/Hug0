+++
date = 2018-07-19T09:00:40-03:00
title = "Memcrashed"
draft = false
cover = "https://cdn-images-1.medium.com/max/1600/1*rW7Okvh0F97n7VWCPOnJQw.png"
categories = ["Reports"]
tags = ["general-security", "Memcached", "Cybersecurity", "Infosec", "DDoS", "IoT"]
weight = 1
+++

Defenseless memcached servers still face the web as they were ever meant to.

<!--more-->

DDoS amplification methods are pursued by the bad guys for a long time. Hitting the weakest point of the hosting chain is a way. Leveraging a service widely used in many web-facing servers in order to generate traffic is another.

![](https://cdn-images-1.medium.com/max/800/1*rW7Okvh0F97n7VWCPOnJQw.png)

Source: [0Kee Team](https://0kee.360.cn/) @ [POC2017](http://powerofcommunity.net/2017.htm)

#### From [memcached's Wiki](https://github.com/memcached/memcached/wiki/ConfiguringServer#networking):

> By default memcached listens on TCP and UDP ports, both 11211. `-l` allows you to bind to specific interfaces or IP addresses. Memcached does not spend much, if any, effort in ensuring its defensibility from random internet connections. So you must not expose memcached directly to the internet, or otherwise any untrusted users. Using SASL authentication here helps, but should not be totally trusted.

By spoofing an IP address in a UDP connection, an attacker can efficiently carry a DDoS attack just by stacking the memcached "stats" command:

![](https://cdn-images-1.medium.com/max/800/1*dvSJ8Ed_Uycc0CIWnaMyiA.png)

keep the stats\nstats\nstats\nstats going...

#### So how's the real world doing?

[Pathetic Patching Leaves Over 70,000 Memcached Servers Still Up For Grabs ≈ Packet Storm\
Information Security Services, News, Files, Tools, Exploits, Advisories and Whitepaperspacketstormsecurity.com](https://packetstormsecurity.com/news/28039/Pathetic-Patching-Leaves-Over-70-000-Memcached-Servers-Still-Up-For-Grabs.html)

Bad.

#### So how's this tiny South American country doing?

![](https://cdn-images-1.medium.com/max/800/1*265QkRiGK0MQ0Zf5u26vNw.png)

obligatory Shodan report.

Take another look at the top versions graph.

More than half of the servers (which shouldn't be exposed to the web) run versions outdated and prone to [RCE vulnerabilities reported by Talos in 2016](http://blog.talosintelligence.com/2016/10/memcached-vulnerabilities.html). Which can result in crafted requests to efficiently maximize the amplification factor of the DDoS attack as described in the 0kee report.

> Update memcached.
