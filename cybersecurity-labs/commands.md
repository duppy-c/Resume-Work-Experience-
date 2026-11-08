# Command Reference

These are all the commands used in Sessions 01–06, grouped by task. The label above each box says where to type it.
**Type only what is inside the box.**

---

## System Maintenance

Update the package list, then install all updates:

```Parrot-OS-Terminal
sudo apt update && sudo apt full-upgrade -y
```

Install Nmap (if it's missing):

```Parrot-OS-Terminal
sudo apt install nmap -y
```

Set the time zone:

```Parrot-OS-Terminal
sudo timedatectl set-timezone America/New_York
```

Check the time and time zone:

```Parrot-OS-Terminal
timedatectl
```

## Password Reset (offline, from Windows plus the Pi)

1. Put the SD card in the Windows PC. Open the **boot** drive and add this to the **end** of the one-line `cmdline.txt` (space first, no new line):

```Notepad-cmdline.txt
 init=/bin/sh
```

2. Boot the Pi. At the `#` prompt, run:

```Parrot-OS-Root-Shell
mount -o remount,rw /
passwd pi
sync
```

3. Put the card back in the PC, **remove** `init=/bin/sh` from `cmdline.txt`, and boot normally.

## Exposure Audit (what is listening on the Pi)

List listening ports and the program that owns each one:

```Parrot-OS-Terminal
sudo ss -tulpn
```

List TCP listeners only:

```Parrot-OS-Terminal
sudo ss -tlpn
```

Show the full command line of a process (replace 2297 with the PID):

```Parrot-OS-Terminal
ps -p 2297 -o args=
```

Optional: turn off wsdd network discovery:

```Parrot-OS-Terminal
gsettings set org.gnome.system.wsdd display-mode 'disabled'
```

## Lab 1: Network Reconnaissance

Show the Pi's interfaces and IP addresses:

```Parrot-OS-Terminal
ip -br addr
```

Find every live host on the LAN (uses ARP) and save the results:

```Parrot-OS-Terminal
sudo nmap -sn 192.168.0.0/24 -oN lab1-discovery.txt
```

Service and version scan of specific hosts (top 100 ports):

```Parrot-OS-Terminal
sudo nmap -sV --top-ports 100 192.168.0.XXX 192.168.0.XXX -oN lab1-unknowns.txt
```

Audit the router:

```Parrot-OS-Terminal
sudo nmap -sV 192.168.0.XXX -oN lab1-router.txt
```

Pull only the IP and MAC lines out of a saved scan:

```Parrot-OS-Terminal
grep -E "report for|MAC" lab1-discovery.txt
```

Find a MAC prefix, with 2 lines of context above it:

```Parrot-OS-Terminal
grep -i -B2 "00:B4:63" lab1-discovery.txt
```

Passive capture filtered by one device's MAC (stop with Ctrl + C):

```Parrot-OS-Terminal
sudo tcpdump -i wlan0 -n -l ether host 00:b4:63:xx:xx:xx
```

Quick check of whether one host is up:

```Parrot-OS-Terminal
sudo nmap -sn 192.168.0.XXX
```

**OUI lookup (manual):** go to maclookup.app and search the first 3 bytes, for example `00:B4:63`.

## Display (LXQt on the Pi 4)

Show the desktop name:

```Parrot-OS-Terminal
echo $XDG_CURRENT_DESKTOP
```

Show the screen name and available resolutions:

```Parrot-OS-Terminal
xrandr --query | head -5
```

Check the Pi display settings:

```Parrot-OS-Terminal
grep -sE "overscan|dtoverlay" /boot/config.txt /boot/firmware/config.txt
```

Test a resolution (temporary until reboot):

```Parrot-OS-Terminal
xrandr --output HDMI-1 --mode 1360x768
```

Undo the test:

```Parrot-OS-Terminal
xrandr --output HDMI-1 --mode 1280x720
```

Make the resolution permanent with an autostart file:

```Parrot-OS-Terminal
mkdir -p ~/.config/autostart
```

```Parrot-OS-Terminal
printf '[Desktop Entry]\nType=Application\nName=TV Resolution\nExec=sh -c "sleep 3; xrandr --output HDMI-1 --mode 1360x768"\n' > ~/.config/autostart/tv-resolution.desktop
```

```Parrot-OS-Terminal
cat ~/.config/autostart/tv-resolution.desktop
```

Remove the autostart file:

```Parrot-OS-Terminal
rm ~/.config/autostart/tv-resolution.desktop
```

12-hour clock custom format (LXQt World Clock settings):

```Clock-Format-Box
h:mm AP
```

## Lab 2: DNS Privacy

Capture plain-text DNS and save it (stop with Ctrl + C):

```Parrot-OS-Terminal
sudo tcpdump -i wlan0 -n -l port 53 | tee lab2-dns-before.txt
```

List the unique names that leaked:

```Parrot-OS-Terminal
grep -oE '[A-Z]+\? [^ ]+' lab2-dns-before.txt | awk '{print $2}' | sort -u
```

Count them:

```Parrot-OS-Terminal
grep -oE '[A-Z]+\? [^ ]+' lab2-dns-before.txt | awk '{print $2}' | sort -u | wc -l
```

After test (same commands, new file name):

```Parrot-OS-Terminal
sudo tcpdump -i wlan0 -n -l port 53 | tee lab2-dns-after.txt
```

Positive control. Make a plain DNS lookup outside Firefox from a second terminal tab:

```Parrot-OS-Terminal
getent hosts example.org
```

Firefox pages used:

```Firefox-Address-Bar
about:networking#dns
```

```Firefox-Address-Bar
about:policies
```

## Firefox Policy Investigation

Find the policy files:

```Parrot-OS-Terminal
find /etc /usr/lib -name policies.json 2>/dev/null
```

Find which package installed them:

```Parrot-OS-Terminal
dpkg -S /etc/firefox-esr/policies/policies.json /etc/firefox/policies/policies.json
```

Count the Parrot references in each file:

```Parrot-OS-Terminal
grep -c parrot /etc/firefox-esr/policies/policies.json /etc/firefox/policies/policies.json
```

## Window Layout (Sessions 03–04)

Identify a window's class (click the window after pressing Enter):

```Parrot-OS-Terminal
xprop WM_CLASS
```

Measure a window, showing only the useful lines (click the window):

```Parrot-OS-Terminal
xwininfo | grep -E "Absolute|Relative|Width|Height"
```

Find which window manager is running:

```Parrot-OS-Terminal
ps -e | grep -E "openbox|xfwm4|kwin"
```

Check whether a package is available or installed:

```Parrot-OS-Terminal
apt-cache policy devilspie2
```

Install devilspie2 and check the version:

```Parrot-OS-Terminal
sudo apt install devilspie2 -y
```

```Parrot-OS-Terminal
devilspie2 --version
```

Create the scripts folder:

```Parrot-OS-Terminal
mkdir -p ~/.config/devilspie2
```

Edit the script:

```Parrot-OS-Terminal
nano ~/.config/devilspie2/windows.lua
```

Run with debug output (stop with Ctrl + C):

```Parrot-OS-Terminal
devilspie2 --debug
```

Create the autostart entry:

```Parrot-OS-Terminal
mkdir -p ~/.config/autostart
```

```Parrot-OS-Terminal
nano ~/.config/autostart/devilspie2.desktop
```

Confirm it's running after a reboot:

```Parrot-OS-Terminal
pgrep -a devilspie2
```

Audit autostart entries (security check):

```Parrot-OS-Terminal
ls -la ~/.config/autostart
```

Check nano key remaps (for the Ctrl + O issue):

```Parrot-OS-Terminal
grep -in "bind" /etc/nanorc ~/.nanorc ~/.config/nano/nanorc 2>/dev/null
```

**nano save routes:** Ctrl + X → Y → Enter (reliable), Ctrl + S, or F3. Ctrl + C cancels any prompt.

## iSH on iPhone (Alpine Linux)

Install the SSH client:

```iSH-Terminal
apk add openssh-client
```

Connect to OverTheWire Bandit (replace 0 with the level number):

```iSH-Terminal
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

Bandit commands so far:

```Bandit-SSH-Session
ls
```

```Bandit-SSH-Session
cat readme
```

```Bandit-SSH-Session
cat ./-
```

```Bandit-SSH-Session
cat "spaces in this filename"
```

## Lab 3: System-Wide Encrypted DNS (Session 05)

Baseline checks:

```Parrot-OS-Terminal
systemctl status systemd-resolved --no-pager
```

```Parrot-OS-Terminal
ls -l /etc/resolv.conf
```

```Parrot-OS-Terminal
cat /etc/resolv.conf
```

```Parrot-OS-Terminal
nmcli connection show --active
```

Back up the DNS file:

```Parrot-OS-Terminal
sudo cp -L /etc/resolv.conf ~/resolv.conf.backup
```

Before/after capture (stop with Ctrl + C):

```Parrot-OS-Terminal
sudo tcpdump -i wlan0 -n -l port 53 | tee lab3-dns-before.txt
```

```Parrot-OS-Terminal
sudo tcpdump -i wlan0 -n -l port 53 or port 853 | tee lab3-dns-after.txt
```

Generate test traffic:

```Parrot-OS-Terminal
getent hosts example.org
```

```Parrot-OS-Terminal
sudo apt update
```

Install and enable systemd-resolved:

```Parrot-OS-Terminal
sudo apt install systemd-resolved -y
```

```Parrot-OS-Terminal
sudo systemctl enable --now systemd-resolved
```

```Parrot-OS-Terminal
sudo systemctl restart NetworkManager
```

Create the DNS-over-TLS drop-in:

```Parrot-OS-Terminal
sudo mkdir -p /etc/systemd/resolved.conf.d
```

```Parrot-OS-Terminal
sudo nano /etc/systemd/resolved.conf.d/dns-over-tls.conf
```

```Resolved-Config-in-nano
[Resolve]
DNS=1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com
DNSOverTLS=yes
Domains=~.
```

```Parrot-OS-Terminal
sudo systemctl restart systemd-resolved
```

Stop the router from supplying DNS:

```Parrot-OS-Terminal
sudo nmcli connection modify NoService ipv4.ignore-auto-dns yes ipv6.ignore-auto-dns yes
```

```Parrot-OS-Terminal
sudo nmcli connection up NoService
```

Verify:

```Parrot-OS-Terminal
resolvectl status
```

```Parrot-OS-Terminal
resolvectl status wlan0
```

```Parrot-OS-Terminal
sudo resolvectl flush-caches
```

```Parrot-OS-Terminal
resolvectl query wikipedia.org
```

Positive control (a deliberate plain-text query to the router):

```Parrot-OS-Terminal
dig @192.168.0.XXX control-test.example.org
```

Analyze the capture:

```Parrot-OS-Terminal
grep -oE '[A-Z]+\? [^ ]+' lab3-dns-after.txt | awk '{print $2}' | sort -u
```

```Parrot-OS-Terminal
grep -c '\.853' lab3-dns-after.txt
```

```Parrot-OS-Terminal
grep -oE '1\.(1\.1\.1|0\.0\.1)\.853' lab3-dns-after.txt | sort | uniq -c
```

Prove persistence (shows the last boot time):

```Parrot-OS-Terminal
uptime -s
```

Rollback:

```Parrot-OS-Terminal
sudo rm /etc/systemd/resolved.conf.d/dns-over-tls.conf && sudo systemctl restart systemd-resolved
```

```Parrot-OS-Terminal
sudo nmcli connection modify NoService ipv4.ignore-auto-dns no ipv6.ignore-auto-dns no && sudo nmcli connection up NoService
```

## Lab 4: Firewall with ufw (Session 06)

Check the Pi's IP and the ufw state:

```Parrot-OS-Terminal
ip -br addr show wlan0
```

```Parrot-OS-Terminal
apt-cache policy ufw
```

```Parrot-OS-Terminal
sudo ufw status verbose
```

Start a test web server (in its own tab; stop with Ctrl + C):

```Parrot-OS-Terminal
python3 -m http.server 8000
```

Scan from iSH (install the tools first):

```iSH-Terminal
apk add nmap curl
```

```iSH-Terminal
nmap -sT -Pn -p 22,80,8000 192.168.0.XXX
```

The reliable closed-vs-filtered test is curl: **refused** = closed, **timed out** = filtered.

```iSH-Terminal
curl -v --max-time 5 http://192.168.0.XXX:80
```

```iSH-Terminal
curl --max-time 5 http://192.168.0.XXX:8000
```

Turn the firewall off/on (for testing only):

```Parrot-OS-Terminal
sudo ufw disable
```

```Parrot-OS-Terminal
sudo ufw enable
```

List the raw kernel firewall rules (nftables):

```Parrot-OS-Terminal
sudo nft list ruleset
```

Least-privilege allow rule, and how to remove it:

```Parrot-OS-Terminal
sudo ufw allow from 192.168.0.0/24 to any port 8000 proto tcp
```

```Parrot-OS-Terminal
sudo ufw status numbered
```

```Parrot-OS-Terminal
sudo ufw delete allow from 192.168.0.0/24 to any port 8000 proto tcp
```

Check the service layer, and stop a process by its PID:

```Parrot-OS-Terminal
sudo ss -tlpn
```

```Parrot-OS-Terminal
kill 2669
```
