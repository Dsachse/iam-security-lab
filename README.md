# iam-security-lab
# AI robotic hardening

## Overview
I built the Picar-x from a raspberry pi 4 for fun mainly, I wanted to add a ai to it to it and essentially create my own robot that was smart. However because you never know this day in age, for the defensive side of Security I decided to really try to reduce the surface attack vectors. I wanted to see what I could do to help ensure the Pi was safe along with the AI. 

## Architecture
![Architecture Diagram](./diagrams/architecture.png)

The Pi runs headless with locked down remote access; API credentials are isolated from the application codebase, and unused network services/ports are disabled to reduce the attack surface.

## Technologies Used
- Raspberry Pi 4 Model B
- Google Gemini AP
- ufw / firewall rules
- SSH hardening (key-only auth)

## What This Project Demonstrates
- Isolating API credentials from application code on an edge device
- SSH hardening (key-only authentication) to eliminate password-based attack vectors
- Reducing overall attack surface by disabling unused services

## How to Deploy / Reproduce
```bash
## How to Deploy / Reproduce

### 1. Install and configure the firewall (ufw)
```bash
# Install ufw
sudo apt install ufw -y

# Set default policies: block all incoming, allow all outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH before enabling — do this first or you'll lock yourself out
sudo ufw allow ssh

# Enable the firewall and confirm the rules
sudo ufw enable
sudo ufw status verbose
```

### 2. Generate an SSH key pair (on your local machine, not the Pi)
```bash
ssh-keygen -t ed25519
``` 

### 3. Copy the public key to the Pi
```bash
ssh-copy-id pi@<pi-ip-address>
```

### 4. Test key-based login before disabling passwords
```bash
ssh pi@<pi-ip-address>
```
Confirm you can log in with no password prompt before continuing.

### 5. Disable password authentication on the Pi
Edit the SSH config:
```bash
sudo nano /etc/ssh/sshd_config
```
Set the following values:
```
PasswordAuthentication no
PubkeyAuthentication yes
```

### 6. Restart SSH to apply changes
```bash
sudo systemctl restart ssh
```

### 7. (Optional) Additional hardening
```bash
# Auto-ban IPs after repeated failed login attempts
sudo apt install fail2ban -y
```
```

## Key Findings / Results
Before/after state — e.g. "X ports open by default → down to Y after lockdown" or a table/screenshot of ufw status before and after. This section is what makes the project credible — worth filling in with real output.

## What I Learned
[See Part 5 for how to write this section well.]

## What I'd Improve
[Be specific. This section is read closely by hiring managers.]
