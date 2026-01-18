# Remote Access - Pi to Home Base

## Goal
Pi in field reports back to home PC (3090 GPU) for:
- Remote control/monitoring
- Offloading heavy compute (hashcat)
- File transfer
- Real-time collaboration

---

## Option 1: Tailscale (Recommended)

Easiest, works through any NAT, encrypted.

```bash
# On Pi
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# On Home PC - install from tailscale.com
tailscale up

# Both get 100.x.x.x addresses
ssh kali@100.x.x.x
```

---

## Option 2: Cloudflare Tunnel

```bash
cloudflared tunnel create sigint-pi
cloudflared tunnel route dns sigint-pi pi.yourdomain.com
cloudflared tunnel run sigint-pi
```

---

## Option 3: Reverse SSH (Needs VPS)

```bash
# On Pi (cron @reboot)
autossh -M 0 -N -R 2222:localhost:22 user@vps-ip

# From home
ssh -p 2222 kali@vps-ip
```

---

## Offloading to 3090

```bash
# Pi captures, sends hash
scp hash.hc22000 user@100.x.x.x:~/cracking/

# Home PC cracks
hashcat -m 22000 -d 1 hash.hc22000 wordlist.txt

# Pi ~10 kH/s vs 3090 ~1,000,000 kH/s = 100,000x faster
```
