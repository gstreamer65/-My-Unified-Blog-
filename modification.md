# Modification No 1: Harden play-server & Integrate with Wazuh

**Target Device:** `play-server` (Ubuntu 24.04 LTS)
**Date Started:** 2026-07-22

## Objective
Enroll the `play-server` instance as a third monitored endpoint in Wazuh and apply a multi-layer hardening configuration[cite: 2]. This will allow a comparison of log volume and alert quality between default and hardened Linux hosts[cite: 2].

## Implementation Steps
* Install and configure the Wazuh agent on `play-server` and verify the heartbeat[cite: 2].
* Enable `auditd` and push audit rules focusing on `execve`, network connections, and privilege escalation[cite: 2].
* Deploy Linux Sysmon (`sysmonforlinux`) scoped for lateral movement and persistence[cite: 2].
* Configure `/var/log/auth.log` and `/var/log/syslog` as log sources via `ossec.conf`[cite: 2].
* Apply CIS Benchmark Level 1 hardening (disable unused services, enforce key-based SSH, configure UFW)[cite: 2].

## Changelog Links
* *(Leave this blank for now. You will add links here to your daily changelog notes as you run the commands in the terminal.)*

## Mistakes Encountered
* *(Leave this blank for now. You will link your Mistake notes here when any software throws an error or configurations do not work immediately.)*
