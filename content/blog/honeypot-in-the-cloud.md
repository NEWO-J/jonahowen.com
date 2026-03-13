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
I initially tried to run the honeypot on a raspberry pi, using Port Forwarding to expose the SSH honeypot on my router, but this proved difficult as the router would perform NAT on the source IPs, rendering IP geolocation and fraud analysis useless. Because AWS lightsail doesnt use NAT for incoming requests, I figured it was more practical for this project.

**Cost breakdown:**

| Component | Approx. Price |
|---|---|
| Splunk Enterprise Free Trial | FREE |
| Cowrie | FREE |
| AWS Lightsail Ubuntu Instance | $5 | 
| **Total** | **$5** |


### 8:27 AM - Starting the Honeypot

### 8:59 AM - First Contact
at 8:59:17.00 AM CDT, we have first contact! The IP origiates from Mountain View, California, USA.

{{< figure src="/images/FirstProbe.png" alt="First probe" width="120%" >}}

Luckily, this was just Nokia's deepfield internet crawler, so no bad guys yet.

{{< figure src="/images/deepfield.png" alt="deepfield" width="120%" >}}

### 12:31 PM - First Login Attempt (Failure)

At 12:31:02 PM (CDT) A "user" with an IP `134.199.153.119`originating from Alexandria, Australia (DigitalOcean's Australia Data Center) tried to logged in to the honeypot.
The credentials used? - `pi:raspberry`. These credentials were rejected by Cowrie.

{{< figure src="/images/failedlogin.png" alt="Failed login" width="120%" >}}

Looking at the IP fraud score for this one, its starting to look malicious.

{{< figure src="/images/ipfraudscore.png" alt="Fraud score" width="120%" >}}
 
### 12:31 PM - First Successful Login

At 12:31:49 PM (CDT) Just 47 seconds after the first login attempt, the same IP address tried the credentials `root:1qaz2wsx`. Unlike the first attempt, this login was a success.

{{< figure src="/images/firstlogin.png" alt="First login" width="120%" >}}

### 12:31 PM - Recon Begins

Just 1 second after successful login, the bot begins to dig through my host. The command the bot executed was quite long but it tells us a lot.
{{< figure src="/images/reconbegins.png" alt="Recon begins" width="120%" >}}

Heres a breakdown of what this command does:
1. Reset the PATH variable
`export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PATH`
2. Collect OS / Kernel Info
`uname=$(uname -s -v -n -m 2>/dev/null)`
3. Get CPU Architecture
`arch=$(uname -m 2>/dev/null)`
4. Get System Uptime
`uptime=$(cat /proc/uptime 2>/dev/null | cut -d. -f1)`
5. Count CPU Cores
`cpus=$( (nproc 2>/dev/null || /usr/bin/nproc 2>/dev/null || grep -c "^processor" /proc/cpuinfo 2>/dev/null) | head -1)`
6. Detect CPU Model
`cpu_model=$( (...) | awk 'NF{print; exit}' )`
7. Detect GPU Hardware
`gpu_info=$( (lspci | grep -i vga; lspci | grep -i nvidia) | head -n50 )`
8. Capture cat --help (Honeypot Detection)
`cat_help=$( (cat --help 2>&1 | tr '\n' ' ') || cat --help 2>&1)`
9. Capture ls --help (Honeypot Detection)
`ls_help=$( (ls --help 2>&1 | tr '\n' ' ') || ls --help 2>&1)`
10. Read Login History
`last_output=$(last 2>/dev/null | head -n 10)`
11. Print All Collected Data
`echo "UNAME:$uname"; echo "ARCH:$arch"; echo "UPTIME:$uptime"; echo "CPUS:$cpus"; echo "CPU_MODEL:$cpu_model"; echo "GPU:$gpu_info"; echo "CAT_HELP:$cat_help"; echo "LS_HELP:$ls_help"; echo "LAST:$last_output"`
