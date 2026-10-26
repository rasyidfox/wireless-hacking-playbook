# Hardware & Setup Guide

## Wireless Adapter (WiFi)

### Chipset yang Direkomendasikan

| Chipset | Adapter | Monitor Mode | Injection | WPA3 | Harga | Rating |
|---------|---------|:---:|:---:|:---:|-------|:---:|
| **Atheros AR9271** | TP-Link TL-WN722N v1 | ✅ | ✅ | ❌ | $25 | ⭐⭐⭐⭐⭐ |
| **Atheros AR9462** | Alfa AWUS036ACH | ✅ | ✅ | ✅ | $50 | ⭐⭐⭐⭐⭐ |
| **Realtek RTL8812AU** | Alfa AWUS036ACS | ✅ | ✅ | ✅ | $45 | ⭐⭐⭐⭐ |
| **Realtek RTL8814AU** | Alfa AWUS036AC | ✅ | ✅ | ✅ | $65 | ⭐⭐⭐⭐ |
| **MediaTek MT7612U** | Various | ✅ | ✅ | ✅ | $35 | ⭐⭐⭐ |
| **Intel AX200** | Built-in | ✅ | ⚠️ | ✅ | - | ⭐⭐⭐ |
| **Ralink RT3070** | Alfa AWUS036NH | ✅ | ✅ | ❌ | $30 | ⭐⭐⭐⭐ |

### Best Choice per Use Case

**Budget (Under $30):**
```
TP-Link TL-WN722N v1 (Atheros AR9271)
+ External antenna RP-SMA
```

**Professional ($50-70):**
```
Alfa AWUS036ACH (Atheros AR9462)
+ Dual band (2.4GHz + 5GHz)
+ High gain antenna
```

**Enterprise/Research ($100+):**
```
Alfa AWUS036ACH + Alfa AWUS1900
+ Multiple adapters for multi-band
+ Directional antenna
```

---

## Antenna

| Tipe | Gain | Beamwidth | Use Case | Contoh |
|------|------|-----------|----------|--------|
| **Omni-directional** | 2-5 dBi | 360° | General scanning | Rubber duck |
| **High-gain Omni** | 7-10 dBi | 360° | Long range | Alfa ARS-N10 |
| **Patch** | 8-14 dBi | 60-90° | Targeted area | Panel antenna |
| **Yagi** | 12-20 dBi | 20-40° | Directional | Yagi antenna |
| **Dish** | 20-30 dBi | 5-10° | Very long range | Parabolic |
| **Cantenna** | 10-15 dBi | 30-60° | DIY | Pringles can |

### Pemilihan Antenna
```
Target jauh (100m+)       → Yagi / Dish
Target menengah (30-100m) → Patch / High-gain Omni
Target dekat (<30m)       → Omni / Rubber duck
Area luas (indoor)        → Omni
Target spesifik           → Yagi (arahkan ke target)
```

---

## Portable Setup

### Raspberry Pi PwnBox

**Hardware:**
```
Raspberry Pi 4 (4GB)
+ Alfa AWUS036ACH
+ Alfa AWUS036NH (2nd adapter)
+ 7" touchscreen (opsional)
+ Power bank 20000mAh
+ External antenna Yagi
```

**Software:**
```bash
# Install Kali ARM
sudo apt update && sudo apt upgrade

# Install wireless tools
sudo apt install aircrack-ng reaver bully hashcat \
    hcxdumptool hcxtools bettercap hostapd-wpe

# Setup monitor mode on boot
cat > /etc/network/interfaces.d/wlan1 << EOF
auto wlan1
iface wlan1 inet manual
    pre-up airmon-ng check kill
    pre-up ip link set wlan1 up
EOF
```

**Repository:**
- https://github.com/koutto/pi-pwnbox-rogueap

### Pwnagotchi

```bash
# Auto WiFi handshake capture
# https://pwnagotchi.ai/
sudo bash -c 'cd /tmp && git clone https://github.com/evilsocket/pwnagotchi'
```

---

## Driver Installation

### Atheros (ath9k_htc)
```bash
sudo apt install firmware-atheros
sudo modprobe ath9k_htc
```

### Realtek (rtl8812au)
```bash
# Kali Linux
sudo apt install realtek-rtl88xxau-dkms

# Manual install
git clone https://github.com/aircrack-ng/rtl8812au
cd rtl8812au
make && sudo make install
```

### MediaTek (mt7612u)
```bash
sudo apt install firmware-misc-nonfree
sudo modprobe mt7612u
```

---

## Kali Linux Wireless Setup

### Post-Install
```bash
# Update
sudo apt update && sudo apt upgrade -y

# Wireless tools
sudo apt install -y aircrack-ng reaver bully hashcat \
    hcxdumptool hcxtools bettercap hostapd-wpe \
    wifite eaphammer airgeddon

# Wireshark
sudo apt install -y wireshark

# Kismet
sudo apt install -y kismet

# GATTacker (BLE)
sudo apt install -y nodejs npm
npm install -g gattacker
```

### Monitor Mode Helper
```bash
#!/bin/bash
# monitor.sh - Setup monitor mode
IFACE=$1

sudo airmon-ng check kill
sudo ip link set $IFACE down
sudo iwconfig $IFACE mode monitor
sudo ip link set $IFACE up

echo "[+] $IFACE in monitor mode"
iwconfig $IFACE
```

---

## Hardware Testing Checklist

### Sebelum Pentest
- [ ] Wireless adapter terdeteksi (`lsusb`, `iwconfig`)
- [ ] Monitor mode berfungsi (`airmon-ng start wlan0`)
- [ ] Injection berfungsi (`aireplay-ng --test wlan0mon`)
- [ ] Antenna terpasang dengan benar
- [ ] Baterai cukup (power bank)
- [ ] Firmware up to date
- [ ] Semua tools terinstall
- [ ] Wordlist tersedia
- [ ] Legal authorization dokumen
- [ ] Backup adapter (jika ada)

---

## Referensi

- **Alfa Network:** https://www.alfa.com.tw/
- **Kali Linux Wireless:** https://www.kali.org/docs/wireless/
- **Pi PwnBox:** https://github.com/koutto/pi-pwnbox-rogueap
- **Pwnagotchi:** https://pwnagotchi.ai/
