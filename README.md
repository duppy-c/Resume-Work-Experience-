# Devin English — Home Lab & Build Portfolio

Hands-on work from my home lab: cybersecurity labs on a Raspberry Pi, a self-hosted AI agent on a second Pi, and Kandies, a music streaming app I built and launched. Everything here is real work with real results — session notes, commands, and evidence photos included.

## What is in this repo

### 1. Cybersecurity Home Labs — Parrot OS on Raspberry Pi 4
[`cybersecurity-labs/`](cybersecurity-labs/)

A lab series on a Raspberry Pi 4 running Parrot OS, working through network reconnaissance, DNS privacy, firewalls, and log analysis on my own home network.

| Lab | Result |
|---|---|
| Lab 1 — Home Network Reconnaissance | Identified all 8 devices on the network; 0 threats found |
| Lab 2 — DNS Privacy (Firefox) | Plain-text DNS leaks cut from **27 to 0** with DNS-over-HTTPS |
| Lab 3 — System-wide DNS-over-TLS | Leaks cut from **5 to 0**; encrypted DNS survived reboot |
| Lab 4 — Firewall with ufw | Firewall proven with nmap + curl; caught a leftover test server still listening after its rule was removed |
| Lab 5 — Log Analysis | **171 firewall block entries** analyzed; reconstructed a test "attack" from the logs; found that Pi logs are volatile by design |

Also in here: full command references, a window-layout project in Lua (devilspie2), and OverTheWire Bandit work. Session notes are in [`cybersecurity-labs/sessions/`](cybersecurity-labs/sessions/), evidence photos in [`cybersecurity-labs/evidence/`](cybersecurity-labs/evidence/).

**Skills:** Linux system administration, networking (DNS, DHCP, LLMNR), nmap, tcpdump, Wireshark, ufw/nftables, systemd, log analysis, Bash, Lua

### 2. Hermes — Self-Hosted AI Agent on Raspberry Pi
[`hermes-agent/`](hermes-agent/)

A self-hosted AI agent ("Hermes") running on a Raspberry Pi 4: Docker sandboxing, an egress firewall, key-only SSH, and remote access over a self-hosted WireGuard tunnel — no cloud gateway, no messaging-platform dependency.

Highlights:

- SSH hardened to **key-only authentication** (password login off, verified)
- Remote access verified **over cellular** through a Pi-only WireGuard tunnel (no IP forwarding — least privilege)
- Router re-hardened after a factory reset (WPS, UPnP, PPTP, remote management off)
- Privacy reviews of every third-party service before use — one provider failed and was dropped
- Full plan, session log, and command reference included

**Skills:** Raspberry Pi / Linux, Docker, WireGuard, SSH hardening, network segmentation, threat modeling, privacy review

### 3. Kandies — Music Streaming App
[`kandies/`](kandies/)

A free music streaming app for Ohio artists ([kandiesapp.com](https://kandiesapp.com)), built as a single-file app on Netlify with a Supabase backend and Cloudflare R2 audio. I run it as Kandies LLC (formed November 2026, Columbus, Ohio).

Highlights:

- Shipped Android release: signed app bundle, built with a GitHub Actions release pipeline and a protected upload key
- Hardened the Supabase backend: row-level security plus trigger and column-level controls, verified with access-control testing
- Built in-app account deletion and a privacy policy from a real data inventory
- Live at kandiesapp.com with artist profiles, an Ohio First spotlight, and listener/artist accounts

**Skills:** JavaScript, HTML/CSS, Supabase (PostgreSQL, Row Level Security), Netlify, Capacitor/Android, GitHub Actions, security hardening, privacy compliance

## How this repo is organized

```text
.
├── cybersecurity-labs/   Parrot OS lab series (sessions, commands, evidence photos)
├── hermes-agent/         Self-hosted AI agent project (plan, sessions, command reference)
└── kandies/              Kandies app build notes and history
```

## A note on privacy

This is a public portfolio, so home-network identifiers (MAC addresses, local IP addresses, email addresses) are redacted throughout the notes and blurred in the evidence photos. Originals stay private.

## Contact

- GitHub: [github.com/duppy-c](https://github.com/duppy-c)
- Kandies: [kandiesapp.com](https://kandiesapp.com)
