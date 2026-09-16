# CR-18 / Module 5 / N5a — Lateral Movement & Network Pivoting

A KYPO cyber range training scenario (CTF-style) covering lateral movement and SSH-based network pivoting on a flat, poorly segmented network.

## Overview

Trainees start with access to an attacker workstation on a flat internal LAN. Starting from a single foothold, they must enumerate the subnet, compromise a poorly-secured workstation, harvest reused credentials/key material, and pivot through that workstation to reach an internal service that is only reachable from within the LAN.

The scenario is built around a single narrative thread: a weak SSH login on one workstation, a reused private key, and a flat network topology are enough for an attacker to move from a single compromised desktop to a "protected" internal service — illustrating the real-world risk of credential/key reuse and insufficient network segmentation.

## Prerequisites

- Comfort using a Linux shell and an SSH client
- Familiarity with dictionary/brute-force attacks (e.g. using hydra)
- Basic understanding of TCP port forwarding and tunnelling
- Completion of CR-18 Modules 2–4 (enumeration, protocol- and service-level attacks), or equivalent hands-on experience

## Learning Outcomes

- Demonstrate how a single compromised workstation enables internal reconnaissance
- Pivot through a compromised host to bypass network segmentation controls
- Exploit LAN-only services that are not externally accessible
- Understand the risks associated with flat network architectures
- Relate pivoting techniques to real-world lateral movement and post-exploitation tactics

## Scenario Structure

The training is delivered as a sequence of levels:

| # | Title | Type |
|---|-------|------|
| 0 | Introduction | Info |
| 1 | Get Access | Access (console login) |
| 2 | Background — Pivoting and SSH Port Forwarding | Info |
| 3 | Establish a Foothold | Training |
| 4 | Demonstrate Internal Impact | Training |
| 5 | Module Completed | Info |

Estimated total duration: ~58 minutes.

An in-scenario briefing introduces the concepts of lateral movement vs. pivoting and SSH local port forwarding (`ssh -L`) before the hands-on levels begin, so trainees have the background needed to complete the exercise without prior pivoting experience.

## Topology

The environment consists of the following machines on an isolated network:

- **Attacker workstation** (`pentestvm`) — pre-equipped with reconnaissance and exploitation tooling; this is the trainee's entry point into the exercise.
- **Victim workstation** (`uservm`) — a standard user machine on the internal LAN, hidden from the topology view until discovered through enumeration.
- **Internal server** (`servervm`) — a backend host on the same LAN, also hidden until discovered, reachable only via pivoting from the victim workstation.
- **Router** — connects the internal LAN to the sandbox's external/management network.

All hosts sit on a single `/24` internal network, reflecting the "flat network" premise of the exercise. Only the attacker workstation is directly reachable by the trainee; the other hosts must be discovered and reached as the exercise progresses.

## Skills Practiced

- Host discovery and service enumeration on an internal subnet
- Dictionary/brute-force attacks against SSH
- Post-exploitation enumeration (SSH keys, shell history, credential artifacts)
- Credential and key reuse across hosts
- SSH local port forwarding to pivot through a compromised host
- Brute-forcing a web application login reached only through a pivot

## MITRE ATT&CK Coverage

Techniques referenced across the scenario include:

- Valid Accounts (T1078)
- Network Service Discovery (T1046)
- System Network Configuration Discovery (T1016)
- Brute Force (T1110)
- Remote System Discovery (T1018)
- Unsecured Credentials (T1552)
- Remote Services (T1021)
- Proxy (T1090)
- Protocol Tunneling (T1572)

## Notes

This repository contains the scenario definition (topology and training content) for deployment on a KYPO-based cyber range. Solutions, hints, and flag values are intentionally excluded from this README.
