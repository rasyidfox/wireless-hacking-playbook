# SDR (Software Defined Radio) Attacks

## SDR Overview

SDR memungkinkan analisis dan serangan pada **semua frekuensi radio**, tidak hanya Wi-Fi.

| Hardware | Harga | Frekuensi | TX | RX | Populer |
|----------|-------|-----------|----|----|---------|
| **RTL-SDR** | $25 | 24MHz-1.7GHz | ❌ | ✅ | Entry-level |
| **HackRF One** | $300 | 1MHz-6GHz | ✅ | ✅ | All-purpose |
| **Yard Stick One** | $120 | Sub-1GHz | ✅ | ✅ | Sub-GHz |
| **Flipper Zero** | $170 | Sub-1GHz | ✅ | ✅ | Multi-tool |
| **BladeRF** | $420 | 300MHz-3.8GHz | ✅ | ✅ | High-end |
| **USRP** | $700+ | Wide range | ✅ | ✅ | Research |
| **Proxmark3** | $250 | 125kHz/13.56MHz | ✅ | ✅ | RFID/NFC |
| **Ubertooth** | $120 | 2.4GHz | ✅ | ✅ | Bluetooth |

---

## RTL-SDR

### Setup
```bash
# Install
sudo apt install rtl-sdr gnuradio

# Test
rtl_test

# Capture signal
rtl_sdr -f 433920000 -s 2048000 -g 40 capture.cfm
```

### Decode Signals
```bash
# 433MHz (keyless entry, IoT)
rtl_433 -f 433920000

# Weather stations
rtl_433 -f 915000000

# Pager (POCSAG)
multimon-ng -t raw -a POCSAG512 capture.raw

# ADS-B (aircraft)
dump1090 --net --interactive
```

---

## HackRF One

### Basic Commands
```bash
# Install
sudo apt install hackrf

# Info
hackrf_info

# TX test (CW)
hackrf_transfer -t signal.bin -f 433920000 -s 2000000 -x 20 -a 1

# RX capture
hackrf_transfer -r capture.bin -f 433920000 -s 2000000 -g 40

# Replay attack
hackrf_transfer -t replay.bin -f 433920000 -s 2000000 -x 20 -a 1
```

### GNURadio
```python
# Flowgraph untuk capture
from gnuradio import gr, blocks, analog, osmosdr

class capture(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self)
        self.src = osmosdr.source(args="hackrf=0")
        self.src.set_center_freq(433920000)
        self.src.set_sample_rate(2000000)
        self.sink = blocks.file_sink(gr.sizeof_gr_complex, "capture.bin")
        self.connect(self.src, self.sink)
```

---

## Common SDR Attacks

### 1. Keyless Entry Replay (Car)
```bash
# Capture remote signal
hackrf_transfer -r car_unlock.bin -f 315000000 -s 2000000 -g 30

# Replay
hackrf_transfer -t car_unlock.bin -f 315000000 -s 2000000 -x 30 -a 1
```
> **Note:** Rolling code (kebanyakan mobil modern) tahan replay.

### 2. RFID Cloning (125kHz)
```bash
# Proxmark3
pm3 --> lf search
pm3 --> lf hid clone -r <hex_data>
pm3 --> lf t55xx write -b <block> -d <data>
```

### 3. NFC Sniffing (13.56MHz)
```bash
# Proxmark3
pm3 --> hf search
pm3 --> hf mf sniff
pm3 --> hf mf dump
pm3 --> hf mf restore
```

### 4. Weather Station / IoT
```bash
# Decode 433MHz sensors
rtl_433 -f 433920000 -M level

# Inject data (if not rolling code)
hackrf_transfer -t weather_data.bin -f 433920000 -s 2000000
```

### 5. Wireless Keyboard Sniffing
```bash
# RTL-SDR capture (2.4GHz)
rtl_sdr -f 2440000000 -s 2000000 keyboard.bin

# Decode with Universal Radio Hacker (URH)
urh keyboard.bin
```

---

## Tools

### Universal Radio Analyzer (URH)
```bash
# Install
pip install urh

# Open GUI
urh

# Auto-detect modulation
urh --auto-detect capture.bin
```

### GNU Radio
```bash
sudo apt install gnuradio
gnuradio-companion  # GUI flowgraph editor
```

### Inspectrum
```bash
# Visual signal analysis
sudo apt install inspectrum
inspectrum capture.bin
```

### rtl_433
```bash
# Generic decoder
sudo apt install rtl-433
rtl_433 -f 433920000 -M level
```

---

## Proxmark3 (RFID/NFC)

### 125kHz (LF)
```bash
# Search
lf search

# Read EM4100
lf em 410x reader

# Clone
lf em 410x clone --id 0102030405

# Write T5577
lf t55xx write -b 0 -d 00148040
```

### 13.56MHz (HF)
```bash
# Search
hf search

# Mifare Classic
hf mf dump
hf mf restore
hf mf cload -f dump.bin

# Desfire
hf 14a reader
hf mfdes info
```

---

## Legal Warning

⚠️ **PERINGATAN HUKUM:**

- ✅ **Legal:** Testing pada device milik sendiri, riset akademik, pen test dengan izin
- ❌ **Illegal:** Membobol kendaraan, akses tanpa izin, intercept komunikasi orang lain
- ❌ **Illegal:** Menyiarkan sinyal tanpa lisensi (FCC/ITU regulations)

---

## Referensi

- **RTL-SDR:** https://www.rtl-sdr.com/
- **HackRF:** https://greatscottgadgets.com/hackrf/
- **Proxmark3:** https://github.com/RfidResearchGroup/proxmark3
- **URH:** https://github.com/jopohl/urh
- **rtl_433:** https://github.com/merbanan/rtl_433
- **GNU Radio:** https://www.gnuradio.org/
