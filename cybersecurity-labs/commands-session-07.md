# New Commands: Session 07 (Lab 5, Log Analysis)

Date: Tuesday, November 3, 2026 (ET)
Only commands not recorded in earlier sessions. Type only what is inside each box.

---

## Find where the logs are kept

Check for a separate ufw log file:

```Parrot-OS-Terminal
ls -l /var/log/ufw.log
```

Count today's firewall blocks in the kernel log:

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep -c "UFW BLOCK"
```

## Read and filter the block entries

Show the first 3 block entries:

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep "UFW BLOCK" | head -3
```

Only TCP blocks aimed at the Pi (finds scans):

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep "UFW BLOCK" | grep "DST=192.168.0.XXX" | grep "PROTO=TCP" | head -5
```

Count blocked TCP attempts by source and port:

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep "UFW BLOCK" | grep "DST=192.168.0.XXX" | grep "PROTO=TCP" | grep -oE 'SRC=[^ ]+|DPT=[^ ]+' | paste - - | sort | uniq -c
```

Search one time window:

```Parrot-OS-Terminal
sudo journalctl -k --since "18:20" --until "18:30" | grep "UFW BLOCK" | grep "PROTO=TCP"
```

```Parrot-OS-Terminal
sudo journalctl -k --since "18:20" --until "18:30" | grep -c "UFW BLOCK"
```

Show time, source, destination and port for each entry in a window:

```Parrot-OS-Terminal
sudo journalctl -k --since "18:20" --until "18:30" | grep "UFW BLOCK" | grep -oE '^.{15}|SRC=[^ ]+|DST=[^ ]+|DPT=[^ ]+' | paste - - - -
```

Rank every source/destination pair by number of blocks (noisiest first):

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep "UFW BLOCK" | grep -oE 'SRC=[^ ]+ DST=[^ ]+' | sort | uniq -c | sort -rn
```

Show block entries from one specific address:

```Parrot-OS-Terminal
sudo journalctl -k --since "today" | grep "UFW BLOCK" | grep "SRC=[external address redacted]" | head -2
```

## Check the firewall's logging rate limit

```Parrot-OS-Terminal
sudo grep -n "limit" /etc/ufw/user.rules
```

## Identify a device's network adapters (Windows PC)

Shows every adapter's Physical Address (MAC), IP and DHCPv6 DUID:

```Windows-Command-Prompt
ipconfig /all
```

## Check whether logs survive a restart

List the boots the journal remembers:

```Parrot-OS-Terminal
journalctl --list-boots
```

Check for the permanent-log folder:

```Parrot-OS-Terminal
ls -d /var/log/journal
```

Show every layer of the journal settings, including add-on files:

```Parrot-OS-Terminal
systemd-analyze cat-config systemd/journald.conf | grep -n -i -B3 "storage"
```

## Save logs to a file before shutting down

```Parrot-OS-Terminal
sudo journalctl -k --since "today" > ~/lab-logs-$(date +%F).txt
```

## Shut down safely

```Parrot-OS-Terminal
sudo shutdown now
```
