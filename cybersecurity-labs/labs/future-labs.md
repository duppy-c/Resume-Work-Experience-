# Future Labs

## Lab 3: System-Wide Encrypted DNS

> **Status: COMPLETED in [Session 05](../sessions/2026-11-03_session-05.md).** Original draft kept below for reference.
>
> ~~PLANNED (draft).~~ These steps are an outline. At lab time, every command will be checked against the current Parrot/Debian documentation before running it. This lab changes how the whole Pi finds websites, so we'll go step by step and keep a rollback ready.

### Why this lab exists

In Lab 2, Firefox's encrypted DNS cut its leaks from **27 → 0**, but the positive control (`getent hosts example.org`) still leaked in plain text. **Every program outside Firefox** (apt updates, terminal tools, Hermes Agent) still asks for website addresses in the clear on port 53.

### Goal

Encrypt DNS for the **entire Pi**, then prove it with the same method as Lab 2: a before capture, an after capture and a positive control.

### Planned approach (draft)

Use **systemd-resolved** with **DNS-over-TLS (DoT)**. DoT sends DNS on **port 853** inside TLS encryption instead of plain text on port 53.

1. **Baseline:** confirm which DNS system the Pi uses right now. Is NetworkManager managing it? Is systemd-resolved installed? Save a copy of `/etc/resolv.conf`.
2. **Before capture:** run the Lab 2 tcpdump on port 53 while running `getent hosts example.org` and `sudo apt update`. Count the leaks.
3. **Install and configure (draft):**
   - Install the `systemd-resolved` package.
   - Edit `/etc/systemd/resolved.conf`, under `[Resolve]`:
     - `DNS=1.1.1.1#cloudflare-dns.com`
     - `DNSOverTLS=yes`
     - `Domains=~.` (sends **all** lookups to the encrypted server)
   - Restart the service, and check NetworkManager is handing DNS to resolved.
4. **Verify the configuration:** `resolvectl status` should show DNS-over-TLS turned on.
5. **After capture:**
   - Port 53 on `wlan0` should be **silent** during `getent` and `apt update`.
   - Port 853 should show the encrypted traffic, proving lookups still happen, just encrypted.
6. **Positive control:** confirm tcpdump still catches a forced plain lookup sent straight to the router.
7. **Report:** compare before and after, list anything that still leaks, and note what's still visible (IPs, SNI).

### Rollback plan

- Keep the saved `/etc/resolv.conf`.
- If browsing breaks: restore the original settings, remove or disable `systemd-resolved`, and restart NetworkManager.
- Test that internet access works again before ending the session.

### Concepts you'll learn

- DNS-over-TLS (port 853) vs. DNS-over-HTTPS (port 443)
- How Linux decides which DNS server to ask (resolv.conf, the stub resolver, NetworkManager)
- Why changing one variable at a time matters, and why rollback plans matter

---

## Other Planned Work

- **Network isolation:** when the second ISP line arrives, move the lab onto its own network. Keep IoT devices (Ring, Roku, Google Cast) on a separate network from the lab.
- **Hermes Agent card:** test-boot it on the Pi 4, then start the agent setup.
- **Router audit follow-up:** confirm with screenshots that Remote Management, WPS, UPnP and DMZ are all off.

---

## Lab 4: Firewall with ufw

> **Status: COMPLETED in [Session 06](../sessions/2026-11-03_session-06.md).**

## Lab 6: Disable LLMNR (planned)

**Why:** in Session 06, `ss -tlpn` showed systemd-resolved listening for LLMNR on **0.0.0.0:5355 and [::]:5355** (all interfaces). LLMNR poisoning is a common attack technique on local networks. ufw currently blocks it, but a second layer is better (defense in depth).

**Draft plan (verify against the docs at lab time):**
1. Baseline: `resolvectl status` (+LLMNR) and `sudo ss -tulpn | grep 5355`.
2. Add `LLMNR=no` to a resolved drop-in (for example, `/etc/systemd/resolved.conf.d/no-llmnr.conf`), then restart resolved.
3. Check whether NetworkManager also needs the setting (`connection.llmnr`).
4. Verify: `+LLMNR` changes to `-LLMNR`, and port 5355 is gone from `ss`.
5. Reboot and confirm persistence (`uptime -s`).
6. Rollback: delete the drop-in and restart resolved.

## Lab 5: Log Analysis (planned)

Read `journalctl`, sudo/auth events and **ufw BLOCK** entries (logging is on, low), and find your own Lab 4 scans in the logs.
