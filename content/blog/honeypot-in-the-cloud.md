---
title: "$5 Honeypot In The Cloud"
layout: "single"
date: 2025-03-10
showtoc: true
description: "Running a Cowrie honeypot hosted in AWS Lightsail with Splunk Connector + Geomap for 24 hours"
cover:
  image: "/images/honeymap.png"
  alt: "Honeypot In The Cloud"
---
It is suprising how much traffic can hit a honeypot in just 24 hours..

I've been wanting to get more blue team experience under my belt, so I thought "why not learn with the real deal?" and I set up a honeypot using AWS lightsail connected to Splunk.
I initially tried to run the honeypot on a Raspberry Pi, using port forwarding to expose the SSH honeypot on my router, but this proved difficult as the router would perform NAT on the source IPs, rendering IP geolocation and fraud analysis useless. Because AWS lightsail doesnt use NAT for incoming requests, I figured it was more practical for this project.

**Cost breakdown:**

| Component | Approx. Price |
|---|---|
| Splunk Enterprise Free Trial | FREE |
| Cowrie | FREE |
| Docker Ubuntu Instance | FREE |
| AWS Lightsail Ubuntu Instance | $5 | 
| **Total** | **$5** |

### 10:15 AM - Open for Business
At 10:15:25 AM CDT, I started up Cowrie, the honeypot is officially exposed to the world wide web.

### 10:27 AM - First Contact
at 10:27:21 AM CDT, we have first contact! No login attempt yet, this was likley just a port scan / probe. 
{{< figure src="/images/firstcontact.png" alt="First contact width="120%" >}}
Using Geolocation, we can see the IP originates from Signapore.
{{< figure src="/images/15minutesin.png" alt="Signapore" width="120%" >}}

### 10:28 AM - First Login
at 10:28: a successful login from the same IP
Credentials used: [root/1234567890]

### 10:29 AM - Malware Assessment Stage
A common pattern in modern malware dropper bots is a sort of "pre-screening test"
In this stage, the bot will run several commands with the goal of answering the question:

*“Is this machine a real, useful target where my malware will successfully run?"*

Heres a breakdown of what the command does:

1. Reset the PATH variable `export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PATH`
2. Collect OS / Kernel Info `uname=$(uname -s -v -n -m 2>/dev/null)`
3. Get CPU Architecture `arch=$(uname -m 2>/dev/null)`
4. Get System Uptime `uptime=$(cat /proc/uptime 2>/dev/null | cut -d. -f1)`
5. Count CPU Cores `cpus=$( (nproc 2>/dev/null || /usr/bin/nproc 2>/dev/null || grep -c "^processor" /proc/cpuinfo 2>/dev/null) | head -1)`
6. Detect CPU Model `cpu_model=$( (...) | awk 'NF{print; exit}' )`
7. Detect GPU Hardware `gpu_info=$( (lspci | grep -i vga; lspci | grep -i nvidia) | head -n50 )`
8. Capture cat –help (Honeypot Detection) `cat_help=$( (cat --help 2>&1 | tr '\n' ' ') || cat --help 2>&1)`
9. Capture ls –help (Honeypot Detection) `ls_help=$( (ls --help 2>&1 | tr '\n' ' ') || ls --help 2>&1)`
10. Read Login History `last_output=$(last 2>/dev/null | head -n 10)`
11. Print All Collected Data `echo "UNAME:$uname"; echo "ARCH:$arch"; echo "UPTIME:$uptime"; echo "CPUS:$cpus"; echo "CPU_MODEL:$cpu_model"; echo "GPU:$gpu_info"; echo "CAT_HELP:$cat_help"; echo "LS_HELP:$ls_help"; echo "LAST:$last_output"`

### 10:30 AM - XMRIG Cryptominer Gets Installed
My honeypot passed the test! the bot proceeded to the next stage and cryptominer malware was installed on the honeypot.


It took several iterations of my honeypot design to ensure it passed these kinds of tests, I used Cowrie in "Proxy" mode with a custom docker instance, this is much more convincing than the default "Shell" mode you will see with most Cowrie honeypots.


### 10:30 AM - C2 Server Spotted
After failing to execute the Cryptominer malware, the bot tried again, using an HTTP request instead of SFTP to download the malware,
this revealed the IP address of the C2 server hosting the malware.

{{< figure src="/images/failedlogin.png" alt="Failed login" width="120%" >}}

