# iam-security-lab
# Raspberry pi hardening lab

## Overview
I built the Picar-x from a raspberry pi 4 for fun mainly, I wanted to add a ai to it and essentially create my own robot that was semi smart. However because you never know this day in age, for the defensive side of Security, I decided to really try to reduce the surface attack vectors. I wanted to see what I could do to help ensure the Pi was safe along with the AI.

## Architecture
![Architecture Diagram](./diagrams/architecture.png)

The Pi is accessed entirely over the network via keybased SSH no direct physical login exists. API credentials live outside the codebase in a permission-locked file, and the firewall denies all inbound traffic except SSH, keeping the exposed attack surface to a single, hardened entry point.

## Technologies Used
- Raspberry Pi 4 Model B
- Google Gemini API
- ufw / firewall rules
- SSH hardening (key-only auth)

## What This Project Demonstrates
- Isolating API credentials from application code on an edge device
- SSH hardening (key-only authentication) to eliminate password-based attack vectors
- Reducing overall attack surface by disabling unused services

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

### 2. Generate an SSH key pair (on your local machine — PowerShell on Windows, Terminal on Mac/Linux — not the Pi)
```bash
ssh-keygen -t ed25519 -C "your-label-here"
```

### 3. Copy the public key to the Pi
```bash
ssh-copy-id <username>@<pi-ip-address>
```

**Note for Windows users:** `ssh-copy-id` is a Linux/Mac only tool and isn't available in PowerShell — you'll get `'ssh-copy-id' is not recognized as an internal or external command`. Use this instead:

```bash
type C:\Users\<you>\.ssh\id_ed25519.pub | ssh sachse02@<pi-ip-address> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Both prompt for the Pi's account password one last time — that's expected, since you need an existing way in to install the very first key.

### 4. Test key-based login before disabling passwords
```bash
ssh <username>@<pi-ip-address>
```
Confirm you can log in with no password prompt before continuing.

### 5. Disable password authentication on the Pi
Edit the SSH config:
```bash
sudo nano /etc/ssh/sshd_config
```

Set the following values:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
MaxAuthTries 3
```
​
### 6. Restart SSH to apply changes
```bash
sudo systemctl restart ssh
```

### 7. (Optional) Additional hardening
```bash
# Auto-ban IPs after repeated failed login attempts
sudo apt install fail2ban -y
```

### 8. Lock down file permissions on project code

By default, files and folders may be loose with permissions. So always make sure to restrict and review anything you create or share.

```bash
# Directories: owner gets full access, group can read/enter but not write, everyone else gets nothing
chmod -R 750 ~/path/to/your/project/

# Files: owner can read/write, group can read-only, everyone else gets nothing
find ~/path/to/your/project/ -type f -exec chmod 640 {} \;
```

Verify the result:
```bash
ls -la ~/path/to/your/project/
```

### 9. Enable automatic security updates

Keeps the system patched against known vulnerabilities without manual intervention.

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Verify it's actually enabled:
```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Expected output:
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";


### 10. Keep secrets out of application code

Hardcoding API keys directly in scripts is a common way credentials end up leaked — especially if that code is ever pushed to a public repo. Store secrets in a separate, permission-locked file instead.

```bash
sudo mkdir -p /etc/myapp
sudo nano /etc/myapp/env
```

Add secrets in `KEY=value` format, one per line:
API_KEY=your_actual_key_here


Lock the file down so only root can read it:
```bash
sudo chmod 600 /etc/myapp/env
sudo chown root:root /etc/myapp/env
```

If the app runs as a systemd service, load the file via `EnvironmentFile=` in the unit file:
[Service]
EnvironmentFile=/etc/myapp/env


The application code itself never contains a single secret — it's safe to make the repo public.

## Key Findings / Results

**Before hardening:**
- No firewall running at all, every port was reachable by default
- SSH accepted password authentication vulnerable to brute force guessing
- Root login was permitted directly over SSH
- No automated protection against repeated failed login attempts
- No limit on authentication attempts per connection

**After hardening — verified with `sudo ufw status verbose`:**

​```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
​```

Only SSH (port 22) is reachable — every other port is denied by default, both IPv4 and IPv6. Combined with:
- Key-only SSH authentication (`PasswordAuthentication no`)
- Root login disabled (`PermitRootLogin no`)
- Login attempts capped at 3 per connection (`MaxAuthTries 3`)
- fail2ban actively monitoring and auto-banning repeated failed attempts

...the attack surface went from "every port reachable, password-guessable, unlimited attempts" to "one port reachable, key-only, capped attempts, automatically banning abuse."

## What I Learned
Originally, I went into this project just trying to build a fun, silly AI robot. But since I'm also studying cybersecurity, I realized I should make sure the Pi itself is actually secure, not just working. For example, there was no firewall running at all originally, and I hadn't limited failed login attempts even after I thought the project was "done."

I set up asymmetric key-based SSH access and disabled passwords entirely, specifically to close off brute force attacks. That decision forced me to actually research the algorithm choice rather than just picking any random one, that's how I learned Ed25519 is a more modern, safer choice and faster than older RSA keys or depending on situation ECDSA as well (which can be vulnerable if its per signature random number generation is ever weak or predictable.)

The biggest thing I learned, though, was around protecting API keys especially relevant given how much AI tooling relies on them today. I moved my API keys out of the Python script entirely and into a separate, permission locked file the system loads at runtime. That way, if I ever share or publish the actual code, the real key is never exposed in it. However prior to this project I would not have thought of even worrying about how the API keys are stored, I definitely had and still have alot to learn in this area.

## What I'd Improve
I think I would probably do more research for hardening in the beginning of the project. Because as I stated earlier, I actually had to go back and add things like the max attempts to ssh when a password has failed. I think in the future as well I will keep continuing  hardening this device because things change constantly. One thing I would like to mention is I will probably be (and you should too) rotating/generating new keys every so often to ensure that the keys haven't been stolen or accessible. Although this project overall has low probability of it actually causing harm if something did happen, I still wanted to act like this project was really worth protecting and what I can do. Another thing I would like to improve further, most of these worries about api keys and having to regulate the firewalls inward is because I have it connecting to a LLM through the cloud. The most "secure" way would to be to get either better hardware and/or look for a really small AI model to run completely local. 
