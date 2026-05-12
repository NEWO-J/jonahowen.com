---
title: "Building a Concurrent Diffing Engine for IDOR/BOLA Triage"
layout: "single"
date: 2026-05-12
showtoc: true
description: "How I built AirGap-Diffy - a local-only HTTP differential testing tool that replays endpoints accross auth sessions to surface access-control breaks without manual comparison"
cover:
  image: "/images/diffy.png"
  alt: "AirGap-Diffy Thumbnail"
---
{{< figure src="/images/diffy.png" alt="Mapview" width="120%" >}}

During my time at Synack Red Team, the scope of real-world penetration tests initially felt overwhelming to me.
Us researchers would often be assigned massive scopes of data to audit that is unlike anything seen in a lab environment such as HackTheBox.

Fellow SRT'ers suggested I pick a vulnerability class, and get good at it - I found myself doing well with Access Control / IDOR vulnerabilites and ended up landing my first critical (CVSS 9.1) bug bounty that way.

Now that I had IDOR down, I needed to automate and reduce triage time. 

My initial intution was to build a system that would use standard hash-diffing on each page to detect differences, this doesn't work due to the dynamic nature of modern website design, anything from a change in clock on the site, to a CSRF token rotating, would result in a false positive.



