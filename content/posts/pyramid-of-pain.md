---
layout: post
title: "Pyramid of Pain"
slug: "pop"
date: "2023-09-08"
comments: false
categories:
  - cybersecurity
tags:
  - Pyramid of Pain
  - Indicators of attack
---

A pyramid which represents how much pain we can cause to an adversary by detecting indicators of their attacks.

![pyaramid](/images/pyaramidofpain/pyramid-of-pain.png)

img: [SOCRadar](https://socradar.io/re-examining-the-pyramid-of-pain-to-use-cyber-threat-intelligence-more-effectively/)

### The Pyramid

- **Hash Values**: Detecting of hash values as a indicator is fairly easy to evade for an adversary by simply changing just a bit from the file. So hash values are considered trivial and is ranked the lowest and widest in the pyramid.
- **IP Address**: It is easy for an adversary to use a lot of different IP addresses and since it doesn't take time to spun up a new IP address, blocking of ip addresses are easy for an adversary to evade.
- **Domain Names**: Domain names are ranked higher than IP address because changing of DNS servers takes a few minutes to hours to propagate globally. They are simple for an adversary to evade but we are on our way to cause some pain.
- **Network/Host Artifacts**:  Observables that are caused by an adversary on either the network or hosts. These observables can include information that reveals an adversary like user agents used by an adversary in a network traffic, packets with malformed headers and misconfigured packets. On the host artifacts, indicators of an adversarial attack could be changing of registry values, services started and stopped, dropping of files and executables in startup folder etc. On this layer of the pyramid, we are starting to cause pain to the adversary and it is becoming a nuisance to them.
- **Tools**: In this layer, we identify the software used by the adversaries which includes tools like backdoor, RATs and other tools that an adversary can use post exploitation. Identification of these tools becomes a challenge to an adversary since they have to write new tools for their exploitation.
- **TTP (Tactics, Techniques and Procedures):** At the apex is the TTP, it is concerned with the behavior of an adversary. Responding at this level is the most effective and is the toughest for the adversary since they have to learn new behavior i.e. new techniques and tactics.

> For further reading of the [Pyramid of Pain](https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html) 