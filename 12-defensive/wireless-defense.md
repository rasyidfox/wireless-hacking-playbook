# Wireless Defense & Hardening

## Wireless IDS/IPS (WIDS/WIPS)

### Kismet (Passive IDS)

```bash
# Monitor mode + logging
kismet -c wlan0mon -n --no-line-wrap

# Alert configuration
# /usr/local/etc/kismet_alerts.conf
alert = DEAUTHFLOOD 3
alert = ASSOCIATION 3
alert = FLOOD 5
```

### Detect Evil Twin

```bash
#!/bin/bash
# detect_evil_twin.sh
# Compare SSID to BSSID mapping

KNOWN_AP="00:11:22:33:44:55"
SSID="CorporateWiFi"

while true; do
    CURRENT=$(iw dev wlan0 scan | grep -B5 "$SSID" | grep "BSS" | awk '{print $2}')
    if [ "$CURRENT" != "$KNOWN_AP" ]; then
        echo "[!] Evil Twin detected! BSSID: $CURRENT (expected: $KNOWN_AP)"
        # Send alert
        echo "Evil Twin: $SSID at $CURRENT" | mail -s "WIRELESS ALERT" admin@company.com
    fi
    sleep 30
done
```

### Wireshark Detection Filters

```
# Rogue AP detection
wlan.ssid == "CorporateWiFi" && wlan.bssid != 00:11:22:33:44:55

# Deauth flood
wlan.fc.type_subtype == 0x0c && wlan.fc.retry == 1

# Beacon flood (many APs)
wlan.fc.type_subtype == 0x08

# KARMA detection (probe response to any SSID)
wlan.fc.type_subtype == 0x05 && wlan.ssid != ""
```

### Zeek (Bro) Wireless

```zeek
# wireless.zeek
event ieee802_11_beacon(c: connection, info: IEEE802_11::BeaconInfo)
{
    if (info$ssid == "CorporateWiFi" && info$bssid !in known_aps)
    {
        NOTICE([$msg=fmt("Rogue AP detected: %s at %s", info$ssid, info$bssid),
                $sub=info$bssid]);
    }
}
```

---

## Network Hardening

### 1. Strongest Encryption
```bash
# WPA3-Enterprise (SAE) with PMF
# hostapd.conf
wpa=2
wpa_key_mgmt=SAE
rsn_pairwise=CCMP
ieee80211w=2  # PMF Required
wpa_group_rekey=3600
```

### 2. PMF (Protected Management Frames / 802.11w)
```conf
# hostapd.conf
ieee80211w=1  # Optional (capable)
ieee80211w=2  # Required (recommended)
```

**Benefits:**
- ✅ Prevents deauth attacks
- ✅ Prevents disassociation attacks
- ✅ Required for WPA3

### 3. 802.1X / Enterprise
```bash
# RADIUS + EAP-TLS (certificate-based)
eap_user_file=/etc/hostapd/eap_user
ca_cert=/etc/hostapd/ca.pem
server_cert=/etc/hostapd/server.pem
private_key=/etc/hostapd/server.key
private_key_passwd=password
```

### 4. Network Segmentation
```
[WLAN] → VLAN 10 (Guest) → Internet
[WLAN] → VLAN 20 (Corporate) → Internal Network
[WLAN] → VLAN 30 (IoT) → Isolated

Firewall Rules:
- VLAN 10 → Internet only
- VLAN 20 → Internal resources
- VLAN 30 → No inter-VLAN routing
```

### 5. MAC Address Filtering (Defense in Depth)
```bash
# hostapd.conf
macaddr_acl=1
accept_mac_file=/etc/hostapd/accept_mac
```
> **Note:** Can be bypassed with spoofing, but adds friction.

### 6. Disable WPS
```conf
# hostapd.conf
wps_state=0  # Disable WPS
```

### 7. Rate Limiting & Detection
```bash
# iptables rate limiting
iptables -A FORWARD -m mac --mac-source 00:11:22:33:44:55 \
    -m limit --limit 10/minute --limit-burst 5 -j ACCEPT

# Detect deauth flood
iw event -t | grep -i "deauth\|disassoc"
```

---

## Wireless Security Checklist

### AP Configuration
- [ ] WPA3-Enterprise (or WPA2-Enterprise minimum)
- [ ] PMF (802.11w) = Required
- [ ] WPS = Disabled
- [ ] WPS PIN = Disabled
- [ ] UPnP = Disabled
- [ ] Remote management = Disabled
- [ ] Strong admin password
- [ ] Firmware up to date
- [ ] Guest network = isolated VLAN
- [ ] MAC filtering (defense in depth)
- [ ] Disable unused features (WDS, WMM if not needed)

### Client Configuration
- [ ] WPA3-SAE support
- [ ] PMF enabled
- [ ] Certificate validation (Enterprise)
- [ ] Auto-connect = Manual approval
- [ ] VPN always-on
- [ ] Firewall enabled
- [ ] OS up to date

### Monitoring
- [ ] WIPS/WIDS deployed
- [ ] Rogue AP detection
- [ ] Deauth/disassoc alerting
- [ ] Anomaly detection
- [ ] Log retention ≥ 90 days
- [ ] Regular wireless audits

---

## Responding to Wireless Attacks

### Deauth Attack Detected
```bash
# 1. Identify attacker
airodump-ng --band abg -w attacker_capture mon0

# 2. Locate (RSSI-based)
python3 locate_attacker.py --bssid <attacker_bssid>

# 3. Block
iptables -A INPUT -m mac --mac-source <attacker_mac> -j DROP

# 4. Document
echo "$(date): Deauth attack from <MAC>" >> /var/log/wireless_incidents.log
```

### Evil Twin Detected
```bash
# 1. Confirm rogue AP
iw dev wlan0 scan | grep -A5 "CorporateWiFi"

# 2. Isolate affected clients
# (deauth from rogue, force reconnect to legitimate AP)
aireplay-ng --deauth 5 -a <rogue_bssid> -c <client_mac> mon0

# 3. Block rogue at switch port (if wired infrastructure allows)
# 4. Notify users
```

---

## Security Assessment Tools

### Wireless Pentest Suite
| Tool | Purpose | Defensive Use |
|------|---------|---------------|
| Kismet | IDS, monitoring | Rogue AP detection |
| Wireshark | Packet analysis | Anomaly detection |
| Nmap | Network discovery | Asset inventory |
| Wifite | Attack automation | Test your own defenses |
| Airgeddon | Multi-attack | Security validation |
| Bettercap | MITM framework | Network monitoring |

### Vulnerability Scanning
```bash
# Scan for misconfigured APs
nmap -sV -p 80,443,8080,8443 <AP_IP>

# Check for open management
nikto -h http://<AP_IP>

# WPS check
wash -i mon0
```

---

## Compliance & Standards

### NIST SP 800-97
- Guidelines for Securing Wireless LANs
- https://csrc.nist.gov/publications/detail/sp/800-97/final

### NIST SP 800-153
- Guidelines for Securing Wireless Local Area Networks (WLANs)

### PCI DSS (Payment Card Industry)
- Requirement 2.2.7: Wireless encryption
- Requirement 11.1: Wireless access point detection

### ISO 27001
- A.13.1.1: Network controls
- A.13.1.2: Security of network services

---

## References

- **NIST 800-97:** https://csrc.nist.gov/publications/detail/sp/800-97/final
- **Wi-Fi Alliance WPA3:** https://www.wi-fi.org/discover-wi-fi/security
- **OWASP Wireless:** https://owasp.org/www-community/attacks/Brute_force_attack
- **Kismet Documentation:** https://www.kismetwireless.net/docs/
