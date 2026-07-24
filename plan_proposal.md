# Sprint 11 Project Plan Proposal

**NAME:** Daud Mehmud[cite: 2]
**DATE SUBMITTED:** 2026-07-03[cite: 2]

## Modifications

### Modification #1
**Chosen Modification:** Harden play-server (Ubuntu 24.04 LTS) and integrate it as a new Wazuh-monitored endpoint[cite: 2].

**Description:** The baseline environment already monitors observer and ad01[cite: 2]. My modification will enroll play-server, the fresh Ubuntu 24.04 LTS instance, as a third monitored endpoint in Wazuh, and then apply a multi-layer hardening configuration to it[cite: 2]. This lets me compare log volume and alert quality between a default (unhardened) and a hardened Linux host, mirroring a real-world scenario where a SOC analyst stands up a new server and has to immediately decide how to tune its logging posture[cite: 2].

**Specific tasks inside this modification:**
* Install and configure the Wazuh agent on play-server and verify heartbeat in the Wazuh dashboard[cite: 2].
* Enable auditd on play-server and push audit rules aligned with MalwareArchaeology's Linux Logging Cheat Sheet (focus: execve, network connections, privilege escalation attempts)[cite: 2].
* Deploy Linux Sysmon (sysmonforlinux) on play-server using a custom configuration adapted from Olaf Hartong's sysmon-modular project, scoped to a threat model around lateral movement and persistence[cite: 2].
* Configure /var/log/auth.log and /var/log/syslog as additional Wazuh log sources using localfiles directives in ossec.conf[cite: 2].
* Apply CIS Benchmark Level 1 hardening (disable unused services, restrict SSH to key-based auth, configure UFW firewall) so I can observe how hardening actions appear in the logs themselves[cite: 2].

**Why This Modification?** Linux servers are the backbone of most enterprise infrastructure, yet they are frequently under-monitored[cite: 2]. I want to understand firsthand what a "before vs. after" hardening story looks like from the SIEM's perspective[cite: 2]. I'm also genuinely curious whether the auditd + Sysmon combination on Linux produces redundant alerts or complementary ones, that seems like exactly the kind of thing a new SOC analyst would want to know[cite: 2].

**References to use while implementing:**
* Malware Archaeology - Windows & Linux Logging Cheat Sheets: audit rule baselines to implement on play-server with auditd[cite: 2].
* Olaf Hartong - sysmon-modular (GitHub): modular Sysmon config generator to build a custom Linux Sysmon config[cite: 2].
* Wazuh Docs - Wazuh Agent Enrollment Guide: step-by-step agent install and registration against the Wazuh manager[cite: 2].
* CIS Benchmarks - CIS Ubuntu Linux 24.04 LTS Benchmark: hardening checklist for Level 1 controls on play-server[cite: 2].
* Wazuh Docs - Monitoring log files with Wazuh: how to add localfiles entries (auth.log, syslog) to ossec.conf[cite: 2].

## Experiments

### Experiment #1
**T1021.004 Remote Services: SSH (Lateral Movement via SSH Brute-Force Simulation)**

**Description:** Using Blue-Team Workstation (the Kali box), I will run a simulated SSH brute-force attack against play-server using Hydra or Medusa[cite: 2]. I will also use the ART T1021.004 atomic to test whether Wazuh's out-of-the-box active response rules flag the repeated authentication failures and whether the source IP gets added to the block list[cite: 2]. After the test, I'll manually inspect /var/log/auth.log and the Wazuh MITRE ATT&CK module to confirm detection[cite: 2].

**Why I picked this:** SSH brute-force is one of the most common attack vectors against Linux servers in the wild[cite: 2]. I want to see exactly what the log artifact looks like at different failure thresholds, does Wazuh alert at 5 failures? 10? And does it matter if the attacker uses slow vs. fast attempts? This feels like a foundational skill for any SOC analyst who monitors Linux infrastructure[cite: 2].

**MITRE ATT&CK reference:** T1021.004 - Remote Services: SSH[cite: 2]

**References to use while conducting:**
* Red Canary / ART - T1021.004 Atomic Tests (GitHub): scripted SSH lateral movement tests to run[cite: 2].
* MITRE ATT&CK-T1021.004 Technique Page: official TTP description, detection advice, and sub-technique context[cite: 2].
* Wazuh Docs - Active Response: Blocking IPs: how Wazuh automatically blocks IPs on repeated failed SSH login[cite: 2].
* Kali Linux Docs - Hydra / Medusa usage: reference for configuring brute-force speed and wordlist parameters[cite: 2].

### Experiment #2
**T1053.003 Scheduled Task/Job: Cron (Persistence via Malicious Cron Job)**

**Description:** I will simulate an attacker achieving persistence on play-server by creating a malicious cron job that periodically executes a reverse-shell payload (or a benign stand-in like a curl to a test server)[cite: 2]. I'll use the ART T1053.003 atomic and observe whether auditd, Linux Sysmon, and Wazuh each independently detect the cron job creation, and then monitor the periodic execution events[cite: 2]. I will also test whether Wazuh's FIM (File Integrity Monitoring) module catches the modification to /etc/crontab or the cron.d directory[cite: 2].

**Why I picked this:** Cron-based persistence is one of the oldest tricks in the Linux attacker playbook, yet many organizations still fail to monitor cron changes[cite: 2]. I'm genuinely curious whether three different logging mechanisms (auditd, Sysmon, Wazuh FIM) each see a different aspect of the same attack, or whether they all catch the same thing[cite: 2]. If they overlap, which one is more useful for an analyst?[cite: 2]

**MITRE ATT&CK reference:** T1053.003 - Scheduled Task/Job: Cron[cite: 2]

**References to use while implementing:**
* Red Canary / ART-T1053.003 Atomic Tests (GitHub): scripted cron persistence atomics to run on play-server[cite: 2].
* MITRE ATT&CK-T1053.003 Technique Page: technique description, real-world examples, and detection guidance[cite: 2].
* Wazuh Docs - File Integrity Monitoring (FIM): configuring FIM rules to alert on /etc/cron* directory changes[cite: 2].
* CERT.JP - Tools Analysis Result Sheet: cross-references persistence TTPs with expected log artifacts[cite: 2].

### Experiment #3
**T1136.001 Create Account: Local Account (Unauthorized User Creation)**

**Description:** Using ART T1136.001, I will simulate an attacker creating a new local user account on play-server (and optionally adding it to the sudo group as an escalation step)[cite: 2]. I'll observe whether Wazuh detects the account creation event via auditd logs, whether the Sysmon Process Creation event captures the useradd or adduser command with full arguments, and whether the Wazuh MITRE ATT&CK dashboard correctly flags T1136[cite: 2]. I'll also test T1548.003 (sudo escalation) as a bonus sub-experiment, chaining it to the account creation to simulate what a real attacker might do next[cite: 2].

**Why I picked this:** Account creation is one of the most obvious signs of an active compromise, yet it can easily be missed if nobody is watching for it[cite: 2]. I want to confirm that my deployment will catch this[cite: 2]. And I'm curious about the chain: does creating a user and then giving it sudo rights generate two separate alerts, or does Wazuh correlate them into a single narrative? That correlation question is what excites me most about SIEM[cite: 2].

**MITRE ATT&CK reference:** T1136.001 Create Account: Local Account; bonus: T1548.003 - Sudo and Sudo Caching[cite: 2]

**References to use while implementing:**
* Red Canary / ART-T1136.001 Atomic Tests (GitHub): atomic test to create a local Linux user via shell commands[cite: 2].
* MITRE ATT&CK-T1136.001 & T1548.003 Pages: technique context, detection data sources, and mitigation info[cite: 2].
* Malware Archaeology - Linux Logging Cheat Sheet: expected auditd events for useradd, usermod, passwd, sudoers edits[cite: 2].
* Wazuh Docs - Wazuh MITRE ATT&CK module: how to read and use the ATT&CK dashboard to validate TTP detection[cite: 2].
