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


## Day 2 

 Resuming after a one-day gap. Re-reading logs to re-anchor context

**Today's objective:-**
Identify suspicious TCP connection behavior by counting SYN packets over time.

Questions:- 

When does “many SYN packets” become suspicious?
- I think that when you get multiple SYN request/packets in a very short span of time(correct me if I am wrong).

Which test did trigger alerts?
- The tests that triggered alerts were nmap, http and  after generated multiple attempt ssh also.

Which test should not have triggered alerts?
- Http should not have triggered alert as it was only surfing the web.

**What I learned:**
- SSH did not trigger alerts because it creates a single long-lived TCP session, not SYN bursts.

**Phase 1 Locked — Summary**
- Implemented a stateless SYN threshold detector
- Observed alert flapping due to lack of suppression
- Legitimate bursty traffic (HTTP) triggered alerts more easily than SSH
- Threshold-based detection cannot infer intent
- Conclusion: volume-only detection is insufficient without state/context

**THE CORE PRINCIPLE I JUST LEARNED:**

- States should be few. Transitions should be strict.


## Phase 2: State Design (No Implementation)

**Objective:**
Design a stateful detection model to address limitations of stateless SYN thresholding.

**States Defined:**

- NORMAL: No recent suspicious behavior
  
- WATCH: IP has triggered suspicious behavior but intent is uncertain
  
- BLOCK: IP has shown repeated suspicious behavior consistent with abuse
  
- State Transition Logic:
  
  - NORMAL → WATCH: One SYN burst detected
    
  - WATCH → BLOCK: Three SYN bursts within 60 seconds
    
  - WATCH → NORMAL: Quiet for 60 seconds
    
  - BLOCK → WATCH: Quiet for 5–10 minutes

- Design Principles Learned:
  
  - Alerts should trigger on state transitions, not events
    
  - Suspicion should decay faster than confirmed malicious behavior
    
  - Thresholds are triggers, not verdicts
    
  - Legitimate traffic can appear more aggressive than attacks

- Known Weaknesses (Accepted):
  
  - Low-and-slow scans can evade burst thresholds
    
  - Shared IPs (NAT/proxies) can cause false positives
    
  - Protocol context is not yet considered

 **Status:**
Phase 2 design completed. Implementation deferred to Day 2.
