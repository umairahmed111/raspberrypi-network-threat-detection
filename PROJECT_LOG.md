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
- I think that when you get multiple SYN request/packets in a very short span of time(I know I am wrong here but still keeping it and other log statements to remember where I went wrong).
  
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


## Phase 2: Day 1 - State Design (No Implementation) 

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


## Day 3

**Today's objective:-** 
Implement a state machine that changes state only when rules are met, and alerts only when state changes.

**Phase 2 — Day 2: Stateful IDS Implementation**
- The transitions bahaved as expected
- Created a very compact reasoning based code for our system(Many more changes to come + optimizations as well)
- Cooldowns were not observable due to event-driven design
- State transitions only occurred when packets arrived
- Identified need for periodic state evaluation
- BLOCK currently represents a confidence state, not enforcement


**Phase 2 locked:**
- Implemented a stateful IDS with time-aware cooldowns and asymmetric escalation.
- System correctly transitions states without packet dependency.
- Identified limitation: lack of protocol context prevents distinguishing benign bursty traffic from malicious behavior.
- Optimization and contextual awareness deferred to Phase 3.

## Day 4

**Phase 3 — Day 1: Context design & integration planning**
- Designed context rules and classified them into structural, behavioral, and semantic signals. 
- Defined strict constraints: context must never override the state machine and may only influence thresholds, cooldowns, suspicion weight, and logging.
- Created a clear integration plan where context exists as a separate layer producing numeric influence rather than decisions.
- Identified that Phase 3’s purpose is confidence modulation, not classification or enforcement. Accepted protocol blindness as intentional and documented context-related weaknesses and attacker evasion tradeoffs.
- In Phase 3, we are focusing on one word that is 'Context'. But what exactly the Context is?
- Context are the information we are needed to learn a pattern and based on that influence thresholds, cooldowns, suspicion weight, and logging (It does not override the state or give decisions).

**The One-Sentence Compression of Phase 3(Just to remember):** 
- Phase 3 is about adding context that adjusts confidence without ever taking control.
 

## Day 5

**Phase 3 — Day 2: Implement structural context (ports + handshake ratio)**
- Integrated structural context into the existing stateful IDS without modifying the core state machine.
- Added port concentration tracking to detect multi-port access patterns indicative of scanning behavior.
- Added handshake completion tracking to differentiate failed connection attempts from legitimate traffic.
- Modified burst detection logic to use weighted SYN counts (effective_syns) instead of raw SYN thresholds.
- Verified that legitimate SSH activity and repeated login attempts do not escalate due to successful handshakes.
- Verified that SYN floods and port scans escalate faster due to higher context weight.
- Observed that cooldown behavior remains static and independent of attack severity.
- Identified design limitation: context currently influences state entry but not state exit or forgiveness.
- Concluded that additional temporal memory is required for severity-aware cooldowns.


## Day 6

**Phase 4 — Day 1: Reputation design & influence mapping**
 - Defined the purpose of reputation as long-term distrust memory, distinct from detection and state transitions.
 - Established that reputation accumulates only on confirmed events, not on raw packets.
 - Designed severity-based reputation growth with repetition amplification to reflect attacker intent over time.
 - Defined strict decay principles: continuous, severity-dependent, asymptotic decay with no instant forgiveness.
 - Mapped how reputation influences the state machine indirectly by modifying escalation sensitivity, cooldown duration, de-escalation    resistance, and re-entry friction.
 - Explicitly prohibited reputation from directly setting states or blocking traffic.
 - Concluded that Phase 4 implementation should focus on memory and decay only, without modifying existing Phase 2 or Phase 3 files.

## Day 7

Resuming after a three-days gap. Re-reading logs to re-anchor where I stopped(Dont take this much long breaks because after that there is a lot to retake and it consumes too much time). 

**Phase 4 — Day 2: Implement Time-Weighted Reputation Memory**
- Implemented standalone reputation memory with severity-aware decay.
- Reputation decays continuously and asymptotically.
- High-severity behavior persists longer than mild behavior.
- No state machine or detection logic modified.


## Day 8

**Phase 4 — Day 3: Reputation wiring, observability, and controlled testing**
- Created a Phase 4–specific detector file to wire reputation memory into the state machine without modifying Phase 3 logic.
- Integrated reputation memory into burst detection by dynamically adjusting burst thresholds based on historical reputation.

