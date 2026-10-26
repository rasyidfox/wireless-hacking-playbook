# FragAttacks (2021)

> **CVE-2020-24586, CVE-2020-24587, CVE-2020-24588, CVE-2020-26139... CVE-2020-26147**
> Paper: https://papers.mathyvanhoef.com/fragattacks.pdf
> Tools: https://github.com/vanhoefm/fragattacks

## Overview

FragAttacks (**Frag**mentation + **Aggregation** Attacks) discovered by Mathy Vanhoef (2021). Exploits weaknesses in:
1. **Frame fragmentation** in 802.11 (all standards since 1997)
2. **Frame aggregation** (A-MSDU, A-MPDU — since 802.11n)

**Impact:**
- Leak encrypted data (even WPA3!)
- Perform MITM attacks
- Inject frames into encrypted networks
- **All Wi-Fi devices are vulnerable** (WiFi 4, 5, 6, 6E)

---

## 3 Vulnerability Classes

### Class 1: Exploiting Frame Aggregation Design

#### CVE-2020-26144 — Inject Frame via A-MSDU
- Attacker adds a **fake 802.3 header** inside A-MSDU
- Makes frame appear to come from encrypted network
- **Target:** All Wi-Fi networks

#### CVE-2020-24588 — Aggregation Attack (A-MSDU)
- Exploits **"A-MSDU present"** flag in header that is not validated
- Attacker can wrap normal frames into A-MSDU
- **Target:** All devices

### Class 2: Exploiting Fragmentation Design

#### CVE-2020-24586 — FragCache
- Fragments not cached correctly
- Attacker can reuse fragments from different sessions
- **Impact:** Combined with fragmentation → inject data

#### CVE-2020-24587 — Mixed Key Attack
- Fragments from different sessions can be combined
- **Impact:** Read encrypted data (decryption oracle)

#### CVE-2020-26140/26141 — Fragment Validation
- Fragments not validated correctly
- **Impact:** Decode frames without auth key

### Class 3: Implementation Vulnerabilities

| CVE | Description | Impact |
|-----|-------------|--------|
| CVE-2020-26139 | Forwarding frame without auth | Bypass AP firewall |
| CVE-2020-26140/41 | Fragment validation | Decode data |
| CVE-2020-26143 | Reassembly buffer | Information leak |
| CVE-2020-26145 | Plaintext frame injection | Inject frame into network |
| CVE-2020-26146 | Fragment forwarding | Skip reassembly |
| CVE-2020-26147 | Key reinstallation | Reuse key |

---

## Practical Attacks

### 1. Data Exfiltration (Decryption Oracle)
```
Attacker → Sends crafted fragment → Client
Client → Responds with encrypted data → Attacker
Attacker → Extracts data from fragment
```
> **Impact:** Read data sent by client (even WPA3)

### 2. Frame Injection into Network
```
Attacker → Uses A-MSDU trick → Inject frame
→ Target accepts frame as if from internal network
```
> **Impact:** Exploit servers within the network

### 3. Firewall Bypass
```
Attacker → Sends frame forwarded by AP without validation
→ Access to resources that should be filtered
```

---

## Tools & Usage

### FragAttack Tool

```bash
# Install
git clone https://github.com/vanhoefm/fragattacks
cd fragattacks
sudo python3 setup.py install

# Test all vulnerabilities
python3 fragattack.py --test-all <interface>

# Test specific CVE
python3 fragattack.py --test cve-2020-24588 <interface>

# Test with target SSID
python3 fragattack.py -i <interface> --test inject <SSID>

# Dump information
python3 fragattack.py -i <interface> --info
```

### Attack Modes
```bash
# Aggregation attack
python3 fragattack.py --test aggregation <interface>

# Mixed key attack (decryption oracle)
python3 fragattack.py --test mixedkey <interface>

# Inject frame
python3 fragattack.py --test inject <interface>

# Bypass firewall
python3 fragattack.py --test firewall <interface>
```

---

## Detection

### Wireshark Filters
```
wlan.fc.type_subtype == 0x08  # Beacon frames
wlan.amsdu_present == 1        # A-MSDU frames
wlan.frag == 1                 # Fragmented frames
```

### Scapy Detection
```python
from scapy.all import *

def detect_fragattacks(pkt):
    if pkt.haslayer(Dot11):
        # Check for unusual A-MSDU
        if hasattr(pkt, 'amsdu_present') and pkt.amsdu_present:
            print(f"[!] A-MSDU detected: {pkt.addr2}")
        # Check for fragments
        if pkt.haslayer(Dot11) and pkt.FCfield & 0x003F:
            print(f"[!] Fragmented frame: {pkt.addr2}")
```

---

## Remediation

### For Users/Enterprise
1. **Update firmware** on all Wi-Fi devices (AP, client)
2. **Enable PMF (802.11w)** — WPA3 already requires it
3. **Use WPA3-only** — no fallback to WPA2
4. **Network segmentation** — isolate wireless from critical systems

### For Vendors
1. Validate A-MSDU flag correctly
2. Cache fragments per-session with correct key
3. Validate fragments before forwarding
4. Implement PMF by default

---

## Patch Status (2024)

| Vendor | Status | Notes |
|--------|--------|-------|
| Apple | ✅ Patched | iOS 14.5+, macOS 11.3+ |
| Android | ✅ Patched | Android 11+ |
| Windows | ✅ Patched | 2021 update |
| Linux | ✅ Patched | Kernel 5.12+ |
| Cisco | ✅ Patched | Meraki, IOS-XE |
| Ubiquiti | ✅ Patched | Latest UniFi firmware |
| MikroTik | ⚠️ Partial | RouterOS update |

---

## References

- **Paper:** https://papers.mathyvanhoef.com/fragattacks.pdf
- **GitHub:** https://github.com/vanhoefm/fragattacks
- **Website:** https://www.fragattacks.com/
- **Black Hat Talk:** https://www.youtube.com/watch?v=UYtr3LdR3jU
