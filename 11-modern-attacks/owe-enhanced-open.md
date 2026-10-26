# OWE (Opportunistic Wireless Encryption)

> **RFC 8110** — Enhanced Open
> Standard: Wi-Fi Alliance Enhanced Open (2018)

## Overview

OWE (**Opportunistic Wireless Encryption**) is an upgrade to **Open** networks (no password) that adds **encryption** without requiring password configuration.

**Problems it solves:**
- Open network = **plaintext**, anyone can sniff
- Enterprise (802.1X) = Too complex for guest/public
- WPA2-PSK = Requires password distribution

**OWE:** **Automatic** encryption without password, like HTTPS for Wi-Fi.

---

## Comparison

| Feature | Open | OWE | WPA2-PSK | WPA3-SAE |
|---------|------|-----|----------|----------|
| Password | ❌ | ❌ | ✅ | ✅ |
| Encryption | ❌ | ✅ | ✅ | ✅ |
| Forward Secrecy | ❌ | ✅ | ❌ | ✅ |
| Complexity | Low | Low | Medium | High |
| Deployment | All | Wi-Fi 6+ | All | Wi-Fi 6+ |

---

## How It Works

### OWE Handshake
```
Client → Probe Request → AP
Client ← Probe Response (RSN + OWE) ← AP
Client → Association Request (DH public key) → AP
Client ← Association Response (DH public key) ← AP
Client ←──────── Derive PMK ────────→ AP
Client ←────── 4-Way Handshake ──────→ AP
```

### Diffie-Hellman Key Exchange
```
Client: g^a mod p (public key)
AP:     g^b mod p (public key)

Shared Secret = g^(ab) mod p
PMK = HKDF(Shared Secret, ...)
```

> **Forward Secrecy:** Even if password/AP is compromised, past sessions remain secure.

---

## Deployment

### AP Configuration (hostapd)

```conf
# OWE (Enhanced Open)
interface=wlan1
driver=nl80211
ssid=GuestNetwork
hw_mode=g
channel=6

# OWE configuration
ieee80211w=2
wpa=2
wpa_key_mgmt=OWE
rsn_pairwise=CCMP

# Transition mode (backward compatibility)
owe_transition_ssid="GuestNetwork-Open"
owe_transition_bssid=00:11:22:33:44:55
```

### Dual SSID (Transition Mode)
```
1. "GuestNetwork-Open"  (Open SSID - legacy)
2. "GuestNetwork"       (OWE SSID - new)

Clients that support OWE → choose OWE
Legacy clients → fall back to Open
```

### wpa_supplicant (Client)

```conf
network={
    ssid="GuestNetwork"
    key_mgmt=OWE
    # or for transition mode
    # scan_ssid=1
}
```

---

## Tools & Testing

### Test OWE Connection

```bash
# Scan for OWE networks
iw dev wlan0 scan | grep -A5 "GuestNetwork"

# Check RSN capabilities
iw dev wlan0 scan | grep -B2 -A5 "RSN:"

# Connect
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/owe.conf
```

### Scapy - Parse OWE Beacon

```python
from scapy.all import *

def check_owe(pkt):
    if pkt.haslayer(Dot11Beacon):
        ssid = pkt.info.decode()
        rsn = pkt.getlayer(RSN_Element)
        if rsn:
            akms = rsn.akm_suites
            for akm in akms:
                if akm.suite == 18:  # OWE
                    print(f"[+] OWE Network: {ssid}")
```

---

## Attacks Against OWE

### 1. Downgrade Attack
- Attacker broadcasts **Open SSID** with same name
- Clients that don't support OWE will connect
- **Mitigation:** Remove transition SSID once all clients support OWE

### 2. Evil Twin OWE
- Attacker creates fake OWE AP
- Client is still encrypted to attacker
- **Mitigation:** Certificate validation, 802.1X for enterprise

### 3. DH Manipulation
- Attacker tries to manipulate Diffie-Hellman parameters
- **Mitigation:** Validate group, use P-256 or Curve25519

---

## Limitations

1. **No authentication** — Cannot verify AP identity
2. **Does not prevent Evil Twin** — Attacker can create fake OWE AP
3. **No authorization** — Everyone can join

> **OWE ≠ Enterprise replacement** — Only an upgrade from Open network

---

## Use Cases

| Scenario | OWE | WPA3 | Enterprise |
|----------|-----|------|------------|
| Public WiFi (airport, cafe) | ✅ | ❌ | ❌ |
| Guest network | ✅ | ❌ | ⚠️ |
| Home WiFi | ❌ | ✅ | ❌ |
| Corporate | ❌ | ❌ | ✅ |
| IoT (simple) | ✅ | ❌ | ❌ |

---

## References

- **RFC 8110:** https://tools.ietf.org/html/rfc8110
- **Wi-Fi Alliance Enhanced Open:** https://www.wi-fi.org/discover-wi-fi/enhanced-open
- **OWE in hostapd:** https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf
