# Wireless Technology and Frequency Spectrum Overview

## Frequency Bands and Technologies

### RFID (Radio Frequency Identification)
- LF (Low Frequency): 120-150 kHz
- HF (High Frequency): 13.56 MHz
- UHF (Ultra-High Frequency): 433 MHz

### Keyless Entry Systems
- North America: 315 MHz
- Europe and Asia: 433.92 MHz

### Cellular Frequencies (US)
- 698-894 MHz: LTE Band 12, 13, 17
- 1710-1755 MHz: AWS (Advanced Wireless Services)
- 1850-1910 MHz: PCS (Personal Communications Service)
- 2110-2155 MHz: AWS

### Global Positioning System (GPS)
- L1: 1575.42 MHz
- L2: 1227.60 MHz
- L Band: Frequency Range: 1-2 GHz

### 802.15.4 (ZigBee)
- Europe: 868 MHz
- US and Australia: 915 MHz

### 802.15.1 (Bluetooth)
- Frequency Range: 2.4-2.483.5 GHz

### Wi-Fi Standards
- 802.11b/g: 2.4 GHz
- 802.11a: 5.0 GHz
- 802.11n: 2.4/5.0 GHz

### Satellite and Microwave Communication
- C Band: 4-8 GHz
- Ku Band: 12-18 GHz
- K Band: 18-26.5 GHz
- Ka Band: 26.5-40 GHz

---

## FCC ID Lookup
To identify devices and their frequency allocation

**Tools:** https://apps.fcc.gov/oetcf/eas/reports/GenericSearch.cfm

## Frequency Database
Access comprehensive database for frequency information

**Tools:** https://www.radioreference.com/db/sid/10974

---

## Linux Networking Commands

### View Wireless Interfaces
```bash
iwconfig
```

### Monitor Mode Activation
```bash
airmon-ng start $IFACE
```

### Packet Capture
```bash
airodump-ng $IFACE
```

### Bluetooth Scanning
```bash
hcitool scan
```

### ZigBee Network Analysis
```bash
sudo zbstumbler /dev/ttyUSB0
```
