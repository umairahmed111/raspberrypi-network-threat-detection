# Raspberry Pi–Based Network Threat Detection & Attack Observation System

## End Goal
(One paragraph — fixed forever)
Design and implement a Raspberry Pi–based network monitoring system that passively captures network traffic, identifies suspicious connection behavior, logs potential intrusion attempts, and analyzes attacker patterns using real-world data collected from an exposed system.
---

## Day 0 – Environment Setup
**What I did:**
- Installed Raspberry Pi OS Lite
- Configured SSH
- Identified network interface

**What broke / confused me:**
- SSH connection initially failed

**What I learned:**
- Headless setup requires SSH + network readiness

**Open questions:**
- Why did SSH fail initially?

## PHASE 1 — NETWORK VISIBILITY

**What I did:**

 I ran few basic commands:
 - $ hostname
MaXx1212
 - $ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      inet 127.0.0.1/8 scope host lo
   --- Hidden ---
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   --- Hidden ---
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
   --- Hidden ---
 - $ ip r
default via --- Hidden ---

- $ sudo tcpdump -i eth0
--- Hidden ---

**What broke / confused me:**
- saw no SYN traffic until I generated it.

  
**What I learned:**
- SYN packets only appear when TCP connections are initiated
- Port scanning is visible and pattern-based
- Nmap generates SYN packets to test port availability

  
**Open questions:**
- What the gateway does?
- I don’t yet understand how nmap decides which ports to scan?
- How can I distinguish scanning from legitimate multi-connection behavior?

