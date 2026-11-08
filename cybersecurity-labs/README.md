# Cybersecurity Labs: Parrot OS / Hermes Agent AI Project

This is a home lab on a Raspberry Pi 4 running Parrot OS, building toward local AI models and AI agents (Hermes).

## Sessions

| Date | Session | Topics |
|---|---|---|
| 2026-10-29 | [Session 01](sessions/2026-10-29_session-01.md) | Pi 4 boot, password reset, router + Pi hardening, **Lab 1: Network Recon (8/8 devices identified)** |
| 2026-10-29 | [Session 02](sessions/2026-10-29_session-02.md) | Time zone / clock / display fixes, Firefox policy investigation, **Lab 2: DNS Privacy (27 → 0 leaks)** |
| 2026-10-30 | [Session 03](sessions/2026-10-30_session-03.md) | iSH: OverTheWire Bandit 0→2; window measurements; window manager identified as xfwm4 |
| 2026-11-01 / 11-03 | [Session 04](sessions/2026-11-03_session-04.md) | **First Lua script:** devilspie2 window layout + autostart, verified after reboot |
| 2026-11-03 | [Session 05](sessions/2026-11-03_session-05.md) | **Lab 3: System-wide DNS-over-TLS (5 → 0 leaks, survives reboot)** |
| 2026-11-03 | [Session 06](sessions/2026-11-03_session-06.md) | **Lab 4: ufw firewall proven with nmap + curl; iSH nmap anomaly solved; leftover server found and killed** |
| 2026-11-03 | [Session 07](sessions/2026-11-03_session-07.md) | **Lab 5: Log Analysis (171 UFW BLOCK entries; logs are volatile by design)** |

## Reference

- [Command reference](commands.md): every command used, grouped by task
- [Future labs](labs/future-labs.md): planned labs (Lab 3 completed in Session 05)

## Folder layout

```text
cybersecurity-labs/
├── README.md
├── commands.md
├── sessions/      session notes with timestamped logs
├── labs/          planned labs
└── evidence/      photos, named EXX_date_timeET_description.jpg
```

## Before publishing publicly

- [x] Blur the email address in `evidence/session-02/E28_*` (done: verification box blurred, email unreadable)
- [ ] Consider redacting MAC addresses (text and photos) — DECISION: hide them. Text done (full MACs → OUI + XX:XX:XX, full LAN addresses → 192.168.0.XXX); photos still to blur
- [ ] Never commit Bandit level passwords (OverTheWire asks players not to publish them)
- [ ] Never commit passwords. Credentials are kept on a separate printed list only
- [x] Review `evidence/session-06/E89_*` (home-folder file names) — checked: names are generic Parrot dotfiles only (.bash_history, .bashrc, .BurpSuite/, .config/, .msf4/, .Xauthority, etc.), no file contents, no personal names; LAN IP in the curl line blurred and verified unreadable
