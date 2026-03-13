---
title: "Honeypot In The Cloud"
layout: "single"
date: 2025-03-10
showtoc: false
description: "Post coming soon."
cover:
  image: "/images/honeymap.png"
  alt: "Honeypot In The Cloud"
---
For 24 hours, I ran a honeypot entirely in the cloud.
It was hosted using Cowrie inside a AWS LightSail instance with a public IP and exposed SSH service, all logs got forwarded to my Splunk server for analysis.

`9:02:00.000 AM` - Starting the honeypot

`12:35:56.000 PM` - First Login
At 12:35 PM (CDT) A user with an IP originating from Alexandria, Australia (DigitalOcean's Australia Data Center) successfully logged in to the honeypot.
The credentials used? - `root:Admin2026!`
{{< figure src="/images/firstlogin.png" alt="Firstlogin" width="50%" >}}
The IP itself had some reports of fraud, but not a ton, indicating it may be a relatively new probe.
{{< figure src="/images/virusscore.png" alt="Firstlogin" width="50%" >}}
