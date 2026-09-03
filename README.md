# iam-security-lab
# [AI robotic hardening]

## Overview
[Overview

A security hardening case study on a a Raspberry Pi–based robotics and AI companion build (SunFounder PiCar-X). This project documents securing API credentials and reducing the device's network attack surface on a resource constrained edge device moving from a working but exposed setup to a hardened one.]

## Architecture
![Architecture Diagram](./diagrams/architecture.png)

[The Pi runs headless with locked down remote access; API credentials are isolated from the application codebase, and unused network services/ports are disabled to reduce the attack surface.]

## Technologies Used
- [Raspberry Pi 4 Model B]
- [Google Gemini AP]
- [ufw / firewall rules]
- [SSH hardening (key-only auth)]

## What This Project Demonstrates
- [Isolating API credentials from application code on an edge device]
- [SSH hardening (key-only authentication) to eliminate password-based attack vectors]
- [Reducing overall attack surface by disabling unused services]

## How to Deploy / Reproduce
```bash
[# [Add your actual commands here, e.g.:]
# ufw enable
# ufw allow <port>
# ufw deny <port>
# sudo systemctl disable <unused-service>]
```

## Key Findings / Results
[[Before/after state — e.g. "X ports open by default → down to Y after lockdown" or a table/screenshot of ufw status before and after. This section is what makes the project credible — worth filling in with real output.]]

## What I Learned
[See Part 5 for how to write this section well.]

## What I'd Improve
[Be specific. This section is read closely by hiring managers.]
