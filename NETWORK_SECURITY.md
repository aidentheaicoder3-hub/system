# Network Security Configuration

## Australia-Only Firewall

All incoming connections are blocked except from Australian IP addresses.

### Current Rules

| Rule | Action |
|------|--------|
| Localhost (lo) | ACCEPT |
| Established/Related connections | ACCEPT |
| Local network (192.168.0.0/24) | ACCEPT |
| Australian IPs (6,665 CIDR ranges) | ACCEPT |
| **Everything else** | **DROP** |

### How It Works

Uses `ipset` with Australian IP ranges from ipdeny.com:

```bash
# View current ipset
sudo ipset list australia | head -50

# View iptables rules
sudo iptables -L INPUT -n -v
```

### Config Files

| File | Purpose |
|------|---------|
| `/etc/ipset-australia.conf` | Australian IP ranges (ipset format) |
| `/etc/iptables.rules` | Firewall rules |
| `/etc/network/if-pre-up.d/iptables` | Auto-load script on boot |
| `/tmp/au.zone` | Raw CIDR list (temporary) |

### Update Australian IP Ranges

```bash
# Download latest Australian IP ranges
curl -s "https://www.ipdeny.com/ipblocks/data/countries/au.zone" -o /tmp/au.zone

# Recreate ipset
sudo ipset destroy australia
sudo ipset create australia hash:net hashsize 8192

# Add all ranges
cat /tmp/au.zone | while read cidr; do
    sudo ipset add australia "$cidr"
done

# Save
sudo ipset save australia > /etc/ipset-australia.conf
```

### Temporarily Allow Another Country

```bash
# Example: Allow US temporarily
curl -s "https://www.ipdeny.com/ipblocks/data/countries/us.zone" -o /tmp/us.zone
sudo ipset create usa hash:net hashsize 8192
cat /tmp/us.zone | while read cidr; do sudo ipset add usa "$cidr"; done
sudo iptables -I INPUT 5 -m set --match-set usa src -j ACCEPT

# Remove later
sudo iptables -D INPUT -m set --match-set usa src -j ACCEPT
sudo ipset destroy usa
```

### Allow Specific IP (Any Country)

```bash
# Allow single IP
sudo iptables -I INPUT 4 -s 1.2.3.4 -j ACCEPT

# Remove later
sudo iptables -D INPUT -s 1.2.3.4 -j ACCEPT
```

### Block Specific IP

```bash
# Block IP (add before ACCEPT rules)
sudo iptables -I INPUT 1 -s 1.2.3.4 -j DROP
```

### Reset to Default (Allow All)

```bash
sudo iptables -F INPUT
sudo iptables -P INPUT ACCEPT
```

---

## Open Ports

| Port | Service | Status |
|------|---------|--------|
| 22222 | SSH | Active |
| 5900 | VNC | Active |

### Permanently Stopped Services

| Port | Service | How to restart |
|------|---------|----------------|
| 11434 | Ollama | `sudo snap start ollama` |
| 9090, 9999 | Python HTTP servers | Manual |
| 4444 | ncat listener | Manual |
| 443 | nc listener | Manual |
| 111 | rpcbind | `sudo systemctl start rpcbind` |

---

## Payload Server (When Needed)

Serve files from /tmp on port 80:

```bash
# Start
cd /tmp && echo 'dd43f8d6354' | sudo -S nohup python3 -m http.server 80 &

# Stop
sudo pkill -f "http.server 80"

# Files available
# - /tmp/ncat.exe (Windows netcat)
```

---

## Remote Access Options

### Option 1: Direct (Australia Only)
- SSH: `ssh -p 22222 bad@YOUR_IP`
- VNC: `vncviewer YOUR_IP:5900`

### Option 2: Tailscale (Through NAT)
```bash
# Install
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Connect via 100.x.x.x addresses
```

### Option 3: Cloudflare Tunnel
```bash
cloudflared tunnel create mytunnel
cloudflared tunnel route dns mytunnel host.yourdomain.com
cloudflared tunnel run mytunnel
```

---

## Quick Commands

```bash
# Check who's connected
ss -tn | grep ESTAB

# Check listening ports
ss -tlnp

# Check firewall rules
sudo iptables -L INPUT -n -v

# Check blocked attempts
sudo iptables -L INPUT -n -v | head -5  # See packet count on DROP

# Whois an IP
whois 1.2.3.4 | grep -E "OrgName|Country|netname"
```

---

## My External IP (Allowed)

- **118.210.134.207** (iiNet/TPG Australia)

---

*Last updated: 2026-01-18*
