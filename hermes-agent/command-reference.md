# Command Reference: Hermes Agent on Raspberry Pi 4

*Running log of every command used to get the system up and running. Last updated 2026-11-04 01:42 ET.*

## Rules

- Run commands one at a time; wait for the prompt to return before the next.
- Paste in the Pi Terminal: Ctrl+Shift+V. Paste in Windows PowerShell (SSH): right-click.
- Check where you are: 'duppy_coder@hermes-pi' = on the Pi. 'PS C:\...' = on your Windows PC.
- Only type your password at a real password prompt ('[sudo] password for ...' or '...'s password:').
- Never put API keys, passwords, or tokens on the command line, in notes, or in photos.
- Use Tab completion for paths to avoid typos.

## 1. Basic navigation and checks

**Where:** Pi

**Clear the screen (or press Ctrl+L)**

```
clear
```

**Show free disk space**

```
df -h /
```

**Show memory**

```
free -h
```

**Show the Pi's network address**

```
hostname -I
```

**Show settings the Pi booted with**

```
cat /proc/cmdline
```

## 2. System updates

**Where:** Pi

**Refresh the list of available software**

```
sudo apt update
```

**Install all updates**

```
sudo apt full-upgrade -y
```

Note: Can take 10-20 minutes.

**Confirm nothing is left to update**

```
apt list --upgradable
```

Note: No output = fully up to date.

**Restart the Pi**

```
sudo reboot
```

## 3. Display fix (Sharp TV, Wayland)

**Where:** Pi (on the Pi's own Terminal, not SSH)

**List displays and supported modes**

```
wlr-randr
```

Note: Output name for this TV: HDMI-A-2.

**Set the no-crop mode (temporary until reboot)**

```
wlr-randr --output HDMI-A-2 --mode 1360x768
```

**Fallback if the screen goes blank**

```
wlr-randr --output HDMI-A-2 --mode 1280x720@60Hz
```

**Confirm the desktop runs user autostart files**

```
cat /etc/xdg/labwc/autostart
```

Note: Look for lxsession-xdg-autostart.

**Create the autostart folder and file**

```
mkdir -p ~/.config/autostart
nano ~/.config/autostart/display-fix.desktop
```

Note: Never create ~/.config/labwc/autostart (it replaces the system file).

**File contents (4 lines)**

```
[Desktop Entry]
Type=Application
Name=Display Fix
Exec=sh -c "sleep 3 && wlr-randr --output HDMI-A-2 --mode 1360x768"
```

Note: Save: Ctrl+O, Enter, Ctrl+X.

**Check the file**

```
cat ~/.config/autostart/display-fix.desktop
```

Note: Known gap: resets after screen sleep. kanshi fix pending.

## 4. Backing up a system file

**Where:** Pi

**Back up the boot command line before editing**

```
sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
```

**Confirm the backup exists**

```
ls /boot/firmware/cmdline.txt.bak
```

## 5. Password incident recovery

**Where:** Pi

**Clear shell history and change the password**

```
clear
history -c && history -w
passwd
```

Note: Use if a password is ever typed at a normal prompt.

## 6. Install Hermes Agent (safe method)

**Where:** Pi

**Download the installer without running it**

```
curl -fsSL https://hermes-agent.nousresearch.com/install.sh -o install.sh
ls -l install.sh
```

**Read it (q to quit)**

```
less install.sh
```

**Check no download-redirect variable is set (should print blank)**

```
echo $HERMES_REPO_URL
```

**List every web address the script contacts**

```
grep -oE 'https?://[^" )]+' install.sh | sort -u
```

Note: Expect only github.com/NousResearch, github.com/astral-sh/uv, *.nousresearch.com.

**Run the installer, reload the shell, health check**

```
bash install.sh
source ~/.bashrc
hermes doctor
```

## 7. SSH from Windows PC to the Pi

**Where:** Pi, then PC

**On the Pi: confirm SSH is running**

```
systemctl is-active ssh
```

Note: Should say active.

**On the Pi: get the host key fingerprint**

```
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Note: Expected: SHA256 starting AGVlkj, ending ZyWNwg.

**On the PC (PowerShell): connect**

```
ssh duppy_coder@<Pi address>
```

Note: Type 'yes' only if the fingerprint matches. Type the password promptly.

## 8. Install Docker Engine (official repo)

**Where:** Pi

**Remove conflicting packages**

```
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

**Add Docker's signing key**

```
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

**Verify the key fingerprint**

```
gpg --show-keys /etc/apt/keyrings/docker.asc
```

Note: Must be 9DC858229FC7DD38854AE2D88D81803C0EBFCD88.

**Add Docker's repository (paste all lines at once)**

```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
```

Note: Expect Suites: trixie, Architectures: arm64.

**Install and test**

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

**Allow your user to run Docker without sudo**

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

Note: Risk: docker group = root-level privileges.

## 9. Hermes management

**Where:** Pi

**Health check**

```
hermes doctor
```

**Change model/provider (re-enter API key)**

```
hermes setup model
```

Note: Paste the key once; nothing else in the field.

**Change terminal backend**

```
hermes setup terminal
```

**Configure messaging (Telegram later)**

```
hermes setup gateway
```

**Configure tool providers**

```
hermes setup tools
```

**Turn tools on/off**

```
hermes tools
```

**Telemetry setting**

```
hermes setup telemetry
```

**View or edit config**

```
hermes config
hermes config edit
```

Note: Secrets live in ~/.hermes/.env. Never photograph it.

**Start chatting**

```
hermes
```

## 10. Hermes gateway service checks

**Where:** Pi

**Should it start at boot? / Is it running?**

```
systemctl --user is-enabled hermes-gateway
systemctl --user is-active hermes-gateway
```

Note: Current target: disabled / inactive.

## 11. GitHub (Windows PC)

**Where:** PC

**Check Git and identity**

```
git --version
git config --global --list
```

**Sign in and clone**

```
git credential-manager github login
git clone https://github.com/duppy-c/cybersecurity-labs.git
```

**Push notes**

```
cd "$env:USERPROFILE\Documents\cybersecurity-labs"
git status
git add session-notes/<file>
git commit -m "<message>"
git push
```

## 12. Shut down or restart the Pi

**Where:** Pi

**Shut down safely (wait until the green light stops blinking before unplugging)**

```
sudo shutdown now
```

**Restart**

```
sudo reboot
```

## 13. Update Hermes (check for security fixes)

**Where:** Pi

**Update Hermes to the latest version**

```
hermes update
```

Note: Run regularly; the npm vulnerabilities found 2026-10-22 are fixed upstream, not locally. Do not run `hermes doctor --fix` until its changes are reviewed.

## 14. iSH on the iPhone (SSH to the Pi, home Wi-Fi)

**Where:** iSH app on iPhone

**Install the SSH client**

```
apk update
```

```
apk add openssh-client
```

**Connect to the Pi (only type yes if the fingerprint starts AGVlkj and ends ZyWNwg)**

```
ssh duppy_coder@<Pi address>
```

**Start Hermes**

```
hermes
```

## 15. Windows network troubleshooting

**Where:** PC (PowerShell)

**Show every adapter, its address, MAC, and gateway**

```
ipconfig /all
```

**Can the PC reach the router at all?**

```
ping <router address>
```

**Is the router's settings page reachable? (TcpTestSucceeded: True/False)**

```
Test-NetConnection <router address> -Port 443
```

```
Test-NetConnection <router address> -Port 80
```

**Forget a saved Wi-Fi network (replace NAME)**

```
netsh wlan delete profile name="NAME"
```

**Make a detailed Wi-Fi report (opens as wlan-report-latest.html)**

```
netsh wlan show wlanreport
```

Note: If Wi-Fi won't connect after a router change: Device Manager > Network adapters > adapter > Uninstall Device (keep the driver), then restart.

## 16. Router (TP-Link Archer AX50)

**Where:** Browser on a home device

**Open the router page (use the number if tplinkwifi.net shows TP-Link's help page)**

Type in the browser address bar: https://<router address> (or http://<router address>)

Note: Back up settings (System Tools > Backup & Restore) before firmware upgrades or big changes. Never restrict admin access to listed devices without a tested way back in.

## 17. Pi Wi-Fi network management (NetworkManager)

**Where:** Pi (on its own screen; switching networks drops SSH)

**Which Wi-Fi network is the Pi on?**

```
nmcli -t -f ACTIVE,SSID device wifi | grep '^yes'
```

**List nearby networks**

```
nmcli device wifi list
```

**List saved connections (press q to exit)**

```
nmcli connection show
```

**Forget a saved network (replace NAME)**

```
nmcli connection delete "NAME"
```

**Show the Pi's addresses, and its Wi-Fi MAC**

```
hostname -I
```

```
cat /sys/class/net/wlan0/address
```

Note: The Pi's address <Pi address> is reserved on the router (MAC entered with dashes, e.g. DC-A6-32-...).

## 18. Display: kanshi (survives screen sleep and reboots)

**Where:** Pi (own Terminal)

**View the kanshi settings**

```
cat ~/.config/kanshi/config
```

**Reload kanshi after changes**

```
killall -s SIGHUP kanshi
```

**Change the port name in the autostart file**

```
sed -i 's/HDMI-A-2/HDMI-A-1/' ~/.config/autostart/display-fix.desktop
```

Note: The screen name depends on the HDMI port used (HDMI-A-1 or HDMI-A-2). Check the first line of `wlr-randr`.

## 19. Hermes updates and checks

**Where:** Pi

**Update Hermes, then run the health check**

```
hermes update
```

```
hermes doctor
```

**After EVERY update: check the gateway (updates can start it). Target: disabled / inactive**

```
echo "BOOT: $(systemctl --user is-enabled hermes-gateway)  RUNNING: $(systemctl --user is-active hermes-gateway)"
```

**If RUNNING says active, stop it**

```
systemctl --user stop hermes-gateway
```

**Identify a process by its PID (replace 6660)**

```
ps -p 6660 -o pid,etime,cmd
```

**View which Hermes tools are on (look only)**

```
hermes tools
```

**Chat with Hermes; leave the chat**

```
hermes
```

```
/exit
```

## 20. Attack surface check (open ports)

**Where:** Pi

**List listening TCP ports (target: only 22, sshd)**

```
sudo ss -tlnp
```

**List UDP ports**

```
sudo ss -ulnp
```

**Turn off rpcbind (port 111, only needed for NFS) and lock it off**

```
sudo systemctl disable --now rpcbind.service rpcbind.socket rpcbind.target
```

```
sudo systemctl mask rpcbind.service rpcbind.socket
```

**Confirm it's off (target: disabled / inactive)**

```
echo "SERVICE: $(systemctl is-enabled rpcbind.service) / $(systemctl is-active rpcbind.service)   SOCKET: $(systemctl is-enabled rpcbind.socket) / $(systemctl is-active rpcbind.socket)"
```

Note: To undo someday (only if NFS is needed): `sudo systemctl unmask rpcbind.service rpcbind.socket`, then `sudo systemctl enable --now rpcbind.socket`.

## 21. Password reset with physical access (init=/bin/sh)

**Where:** PC (edit card), then the Pi's keyboard

Note: Append ` init=/bin/sh` to the single line in `cmdline.txt` on the card's boot partition (back it up first). Boot to the `#` prompt, run the commands below, then remove `init=/bin/sh` on the PC and boot normally.

```
mount -o remount,rw /
```

```
passwd pi
```

```
sync
```

## 22. tmux (keep Hermes running when the phone disconnects)

**Where:** Pi (run after SSH-ing in; NOT at the iPhone's localhost prompt)

**Install (once)**

```
sudo apt install tmux
```

**Start a session named hermes, then run Hermes inside it**

```
tmux new -s hermes
```

```
hermes
```

**Reattach after a disconnect**

```
tmux attach -t hermes
```

**List running sessions**

```
tmux ls
```

Note: Detach on purpose with Ctrl+B, then D (lowercase b). In iSH, the ^ toolbar key is Ctrl.

## 23. SSH keys (key-only login)

**Where:** iSH (localhost:~# prompt) for key creation and copying; Pi for server settings

**Create a key on the iPhone (press return at the file prompt; set a passphrase)**

```
ssh-keygen -t ed25519 -C "iphone-ish"
```

**Copy the public key to the Pi**

```
cat ~/.ssh/id_ed25519.pub | ssh duppy_coder@<Pi address> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

**On the Pi: list allowed keys by fingerprint**

```
ssh-keygen -lf ~/.ssh/authorized_keys
```

**On the Pi: check effective SSH settings**

```
sudo sshd -T | grep -Ei 'passwordauthentication|pubkeyauthentication'
```

**On the Pi: turn off password login (00 file beats 50-cloud-init.conf)**

```
printf 'PasswordAuthentication no\nKbdInteractiveAuthentication no\n' | sudo tee /etc/ssh/sshd_config.d/00-keys-only.conf
```

```
sudo sshd -t
```

```
sudo systemctl reload ssh
```

**Prove passwords are refused (expect "Permission denied (publickey)")**

```
ssh -o PubkeyAuthentication=no duppy_coder@<Pi address>
```

Note: Undo from the Pi's own keyboard: `sudo rm /etc/ssh/sshd_config.d/00-keys-only.conf` then `sudo systemctl reload ssh`. Never share or photograph `id_ed25519` (no .pub).

## 24. WireGuard (Pi-only tunnel: Pi <Pi tunnel address>, iPhone <iPhone tunnel address>)

**Where:** Pi

**Install and check**

```
sudo apt update && sudo apt install wireguard
```

```
wg --version
```

**Make the Pi's keys (root-only files)**

```
sudo sh -c 'umask 077; wg genkey | tee /etc/wireguard/wg-private.key | wg pubkey > /etc/wireguard/wg-public.key'
```

**Show the Pi's PUBLIC key (safe to share)**

```
sudo cat /etc/wireguard/wg-public.key
```

**Write wg0.conf without displaying the private key (iPhone public key saved in ~/iphone-wg.pub)**

```
sudo sh -c 'umask 077; printf "[Interface]\nAddress = <Pi tunnel address>/24\nListenPort = 51820\nPrivateKey = %s\n\n[Peer]\nPublicKey = %s\nAllowedIPs = <iPhone tunnel address>/32\n" "$(cat /etc/wireguard/wg-private.key)" "$(cat /home/duppy_coder/iphone-wg.pub)" > /etc/wireguard/wg0.conf'
```

**Check the config without the private key**

```
sudo grep -v PrivateKey /etc/wireguard/wg0.conf
```

**Start now and at every boot; check status (look for "latest handshake")**

```
sudo systemctl enable --now wg-quick@wg0
```

```
sudo wg show
```

**SSH through the tunnel (from iSH, tunnel switched on in the WireGuard app)**

```
ssh duppy_coder@<Pi tunnel address>
```

**Confirm where logins come from**

```
who
```

Note: iPhone app peer = Pi public key (<key redacted>), Endpoint <Pi address>:51820 (home test), Allowed IPs <Pi tunnel address>/32, keepalive 25. Each side's Peer holds the OTHER side's public key. Never show or photograph private keys.

## 25. tmux naming

**Create one named session (-s) / reconnect (-t) / clear all**

```
tmux new -s hermes
```

```
tmux attach -t hermes
```

```
tmux kill-server
```

Note: `tmux new -t hermes` (with -t) creates grouped sessions named hermes-0, hermes-1, ... instead of one session.

## 26. Remote access from anywhere (WireGuard, verified 2026-11-05)

**Daily use (iPhone):** turn the WireGuard tunnel on, then in iSH:

```
ssh duppy_coder@<Pi tunnel address>
```

```
tmux attach -t hermes
```

**Check the tunnel from the Pi (endpoint = where the phone connected from)**

```
sudo wg show
```

- Endpoint = home public IP or 192.168.0.x: phone is on home Wi-Fi.
- Endpoint = any other address: phone connected from outside (cellular).

Note: the phone's Endpoint must be the router's **IP Address** field (Advanced > Network > Internet) plus `:51820`, **not** the Default Gateway. Connection type is Dynamic IP, so recheck this field if the tunnel stops connecting. Use the app's Latest handshake / Data received, not the status-bar VPN badge (iPhone 17 hides it).

**Which address to use**

| Phone location | Tunnel | Command |
|---|---|---|
| Home Wi-Fi | Off | `ssh duppy_coder@<Pi address>` |
| Anywhere | On | `ssh duppy_coder@<Pi tunnel address>` |

## 27. iSH welcome message (on the iPhone, localhost:~# prompt)

```
echo "YOUR MESSAGE" > /etc/motd
```

```
cat /etc/motd
```

Note: `>` replaces the message; `>>` adds a line.
