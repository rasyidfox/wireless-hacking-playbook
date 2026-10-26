# Man-in-the-Middle Attack

> Once you are in the network, you can test if it is vulnerable to Man-in-the-Middle attacks.

## Steps

1. **ARP Spoofing** attack using Ettercap
2. **Sniff** the traffic using Wireshark or TCPDump
3. **Analyze** the traffic using PCredz (Linux) or Network Miner (Windows & Linux)

---

## ARP Spoofing with Ettercap

```bash
ettercap -T -q -i wlan0 -M ARP /<IP_target>//
```

## Sniffing

### Wireshark
```bash
wireshark -i wlan0
```

### TCPDump
```bash
tcpdump -i wlan0 -w capture.pcap
```

## Traffic Analysis

### PCredz (Linux)
Extracts credit card numbers, emails, HTTP authentication, etc. from network traffic.

### Network Miner (Windows & Linux)
Network forensic analysis tool for extracting files, images, credentials from PCAP files.
