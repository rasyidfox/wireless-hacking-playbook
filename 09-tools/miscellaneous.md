# Miscellaneous

## WPA/WPA2 No Clients Showing

In WPA/WPA2 if there is no clients connected, try to broadcast deauthenticate. Do it a few times and if there is clients connected they should pop on your airodump-ng capture.

```bash
# Deauthentication broadcast attack
aireplay-ng -0 1 -a <BSSID> <INTERFACE>
```

---

## WiFi Jamming

**Links:**
- https://github.com/bhikandeshmukh/Wi-Fi-Jammer
- https://github.com/zflemingg1/WiFi-Jammer
- https://github.com/DanMcInerney/wifijammer

---

## Automation Tools

- **WEF** (WiFi Exploitation Framework)
- **Wifiphisher**
- **Kismet**

---

## MACStealer

- **Paper:** https://papers.mathyvanhoef.com/usenix2023-wifi.pdf
- **Repository:** https://github.com/vanhoefm/macstealer
- **Wifi Framing:** https://github.com/domienchepers/wifi-framing

## ICMP Redirect

> Man-in-the-Middle Attack without Rogue AP: When WPAs Meet ICMP Redirects.

---

## Bluetooth and Software Defined Radio Attacks

## RFID, Smart Cards, and NFC Hacking

---

## Tools

- **GeoWiFi** — WiFi Geolocation Data Search
- **PiDense** — Raspberry Pi wireless auditing
- **Wifite** — Automated wireless attack tool
