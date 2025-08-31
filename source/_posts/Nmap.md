---
title: Nmap
date: 2025-03-19 19:24:31
tags: [Port]
categories: [Tools]
description: Note for Nmap Usage
comments: false
cover: images/Nmap.png
---

> I'm going to steal a house...
>
> Where is the house?
>
> What is inside the house?
>
> Where is the window?
>
> Why it deserves me to steal?

### **NMAP = Network Mapper**

To collect information before implementing attacks.

For: Host Discovery, Port Scanning, Service Version Detection, OS Detection

#### Port State

- Open: Accepting packets (TCP/UDP)
- Closed: NOT listening; no service
  - Still useful for determine live hosts: When Nmap sends probe packets to a closed port, most operating systems will respond with an RST (reset) packet. This indicates that while the port is closed, the host itself is active. This differs from filtered or blocked ports, which typically don't return any response. Therefore, closed ports can help confirm whether a host is online.
  - Still useful for OS fingerprinting: Different operating systems respond to closed ports in distinctive ways, including variations in RST packet format, TTL values, window size, IP ID sequences, and other characteristics. Nmap uses these differences to perform operating system fingerprinting. In fact, closed ports often provide more information for OS detection than open ports because their responses are more consistent and predictable.
- Filtered: The port cannot be reached so its state cannot be determined
  - Most likely blocked by a firewall
- Unfiltered: The port can be reached but Nmap cannot determine its state

#### Scan

- -sS: SYN Scan
- -sT: Connect Scan
- -sU: UDP Scan

Reviewing output from Nmap is time consuming, which has large bunch of data. We need Parses Nmap .gnmap output files

Q: Why there is so many lowercase and uppercase in params?

A: UNIX is Case Sensitive
