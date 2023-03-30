---
# title: "HackTheBox"
# permalink: /life/pages/
excerpt: "Mitre Att&Ck security framework"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: hacking
categories:
  - software_engineering
  - hacking
---

* Table of contents (do not remove this line)
{:toc}

The MITRE ATTACK Framework is a curated knowledge base that tracks cyber adversary tactics and techniques

TTPs (Tactics, Techniques, and Procedures):

1. Tactics: Adversarial tactics are specific technical objectives that an adversary intends to achieve
1. A technique describes one specific way an adversary may try to achieve an objective.
1. Procedures: A procedure is a series of actions that an adversary may take to achieve a technique.

[Crowdstrike definition](https://www.crowdstrike.com/cybersecurity-101/mitre-attack-framework/) - start here

[Mitre Att&Ck official security framework](https://attack.mitre.org/)

[Enterprise Tactics](https://attack.mitre.org/tactics/enterprise/)


## CrowdStrike Falcon Adaptation

Our Objective layer: Groups related tactics, making them easier to learn and remember.

-    Gain access -- Initial Access, Credential Access, Privilege Escalation
-    Keep access -- Persistence, Defense Evasion
-    Explore -- Discovery, Lateral Movement
-    Contact controlled systems -- Command and Control
-    Follow through (basically, steal and break things) -- Collection, Exfiltration, Execution, Impact
-    Network-based effects -- Network Effects, Remote Service Effects
