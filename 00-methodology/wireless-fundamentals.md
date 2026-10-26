# Wireless Fundamentals — 802.11 Deep Dive

## 802.11 Network Architecture

### Basic Service Set (BSS)
- **Infrastructure BSS** — Consists of AP + clients (most common)
- **Independent BSS (IBSS/Ad-Hoc)** — P2P between clients without AP
- **Extended Service Set (ESS)** — Multiple BSS connected via Distribution System (DS)

### Key Components
| Component | Function |
|-----------|----------|
| **AP (Access Point)** | Bridge between wireless and wired network |
| **Station (STA)** | Client connected to AP |
| **BSSID** | MAC Address of the AP (Basic Service Set ID) |
| **SSID** | Network name being broadcast |
| **ESSID** | Same SSID across multiple APs (Extended SSID) |
| **Channel** | Specific frequency (1-13 on 2.4GHz, 36-165 on 5GHz) |
| **Distribution System (DS)** | Wired backbone connecting APs |

### Connection Process
```
1. Probe Request     → Client searching for SSID
2. Probe Response    → AP responds
3. Authentication    → Open / Shared / SAE
4. Association       → Client joins BSS
5. 4-Way Handshake   → Generate PTK/GTK (WPA/WPA2/WPA3)
6. Data Transfer     → Encrypted
```

---

## 802.11 Frame Types

802.11 has **3 main frame types:**

### 1. Management Frames (Type=0)
Manage connection and communication.

| Subtype | Name | Function | Attacked? |
|---------|------|----------|-----------|
| 0 | Association Request | Client → AP: "May I join?" | — |
| 1 | Association Response | AP → Client: "Allowed/denied" | — |
| 2 | Reassociation Request | Moving between APs | — |
| 3 | Reassociation Response | Response | — |
| 4 | Probe Request | Client searches for SSID | ✅ KARMA/MANA |
| 5 | Probe Response | AP responds to probe | ✅ Known Beacons |
| 8 | Beacon | AP broadcasts SSID | ✅ Evil Twin |
| 10 | Disassociation | Disconnect | ✅ Deauth/Disassoc |
| 11 | Authentication | Auth step 1 | ✅ Fake Auth |
| 12 | Deauthentication | Kick client | ✅ Deauth Attack |
| 13 | Action Frame | Management (HT, VHT) | — |

> **Key fact:** Management frames on networks **without PMF (802.11w)** are NOT encrypted! This is why deauth attacks work.

### 2. Control Frames (Type=1)
Assist with data delivery.

| Subtype | Name | Function |
|---------|------|----------|
| 11 | RTS (Request to Send) | Before sending data |
| 12 | CTS (Clear to Send) | RTS response |
| 13 | ACK (Acknowledgment) | Reception confirmation |
| 14 | CTS-to-Self | CTS without RTS |

### 3. Data Frames (Type=2)
Carry payload (IP packet, etc.).

| Subtype | Name | Function |
|---------|------|----------|
| 0 | Data | Regular data |
| 4 | Null Function | Client info: no data sent (power save) |
| 8 | QoS Data | Data with QoS |

---

## Frame Header Structure

```
┌─────────────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ Frame Control│ Duration  │  Addr1   │  Addr2   │  Addr3   │ Seq Ctrl │
│   (2 bytes)  │ (2 bytes) │ (6 bytes)│ (6 bytes)│ (6 bytes)│ (2 bytes)│
├─────────────┴───────────┴──────────┴──────────┴──────────┴──────────┤
│                        Frame Body (0-2304 bytes)                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Address Fields depend on To DS / From DS:**
| To DS | From DS | Addr1 | Addr2 | Addr3 | Addr4 |
|-------|---------|-------|-------|-------|-------|
| 0 | 0 | DA | SA | BSSID | — |
| 0 | 1 | DA | BSSID | SA | — |
| 1 | 0 | BSSID | SA | DA | — |
| 1 | 1 | RA | TA | DA | SA |

> DA=Destination, SA=Source, BSSID=AP MAC, RA=Receiver, TA=Transmitter

---

## 802.11 Authentication State Machine

### Open Authentication
```
Client → Authentication Request (Seq=1) → AP
Client ← Authentication Response (Seq=2) ← AP
Client → Association Request → AP
Client ← Association Response ← AP
```
> Weakest, no verification.

### Shared Key Authentication (WEP)
```
Client → Authentication Request (Seq=1) → AP
Client ← Challenge Text ← AP
Client → Challenge + Encrypted Response → AP
Client ← Authentication Response (Seq=4) ← AP
```
> Not more secure than Open (actually weaker because it exposes challenge).

### SAE (WPA3)
```
Client → Commit (password + element) → AP
Client ← Commit (password + element) ← AP
Client → Confirm (proof) → AP
Client ← Confirm (proof) ← AP
```
> Dragonfly handshake — resistant to offline dictionary attacks.

---

## 4-Way Handshake (WPA/WPA2/WPA3-PSK)

```
┌──────────┐                    ┌──────────┐
│  Client   │                    │    AP    │
└────┬─────┘                    └────┬─────┘
     │                               │
     │    ┌─────────────────────┐    │
     │◄───┤ 1. ANonce           │◄───┤
     │    └─────────────────────┘    │
     │                               │
     ├─────────────────────►│       │
     │    ┌─────────────────┐│       │
     │    │ 2. SNonce + MIC  ├┘       │
     │    └─────────────────┘        │
     │                               │
     │◄─────────────────────┤       │
     │    ┌─────────────────┐│       │
     │    │ 3. GTK + MIC    ├┘       │
     │    └─────────────────┘        │
     │                               │
     │    ┌─────────────────┐        │
     │    │ 4. ACK + MIC     │───────►│
     │    └─────────────────┘        │
     │                               │
```

**Key Derivation:**
```
PMK = PBKDF2(Passphrase, SSID, 4096, 256)
PTK = PRF-512(PMK, "Pairwise key expansion", ANonce + SNonce + AA + SPA)
GTK = Derived from PMK (broadcast/multicast key)
```

**Capturing 4-Way Handshake = Can crack PMK (passphrase)**

---

## Key Parameter Definitions

### IV (Initialization Vector)
- 24-bit (WEP) — Very short, easy collision
- 48-bit (WPA/TKIP) — Larger, replay counter

### Nonce
- ANonce (AP Nonce) — Random from AP
- SNonce (STA Nonce) — Random from client
- Used to derive unique PTK per session

### PMF (Protected Management Frames / 802.11w)
- **Encrypts management frames** (deauth, disassoc)
- **Prevents deauth attacks**
- **Required in WPA3**, optional in WPA2
- Status in beacon frame: `RSN Capabilities > MFPC > MFPR`

---

## Channel & Bandwidth

### 2.4 GHz
| Channel | Frequency (MHz) | Overlap |
|---------|-----------------|---------|
| 1 | 2412 | — |
| 6 | 2437 | ✅ Most common |
| 11 | 2462 | — |

> 3 non-overlapping channels: **1, 6, 11**

### 5 GHz
| Band | Channels | Use |
|------|----------|-----|
| U-NII-1 | 36, 40, 44, 48 | Indoor |
| U-NII-2 | 52-64 | DFS (Dynamic Frequency Selection) |
| U-NII-2e | 100-144 | DFS, outdoor |
| U-NII-3 | 149-165 | Outdoor |

### 6 GHz (Wi-Fi 6E)
- Channel 1-233 (5955-7115 MHz)
- **WPA3-only** (no legacy support)
- Channel width up to 160 MHz

---

## Encryption Comparison

| Feature | WEP | WPA (TKIP) | WPA2 (CCMP) | WPA3 (SAE) |
|---------|-----|------------|-------------|------------|
| Key Size | 40/104-bit | 128-bit | 128/256-bit | 128/256-bit |
| Integrity | CRC-32 | Michael MIC | CCM (CBC-MAC) | CCM/GCMP |
| Key Mgmt | Static | PSK (PBKDF2) | PSK (PBKDF2) | SAE (Dragonfly) |
| IV Size | 24-bit | 48-bit | 48-bit | 48-bit |
| PMF | ❌ | ❌ | Optional | ✅ Required |
| Forward Secrecy | ❌ | ❌ | ❌ | ✅ |
| Offline Crack | ✅ Instant | ✅ Dictionary | ✅ Dictionary | ❌ Difficult |

---

## References

- IEEE 802.11-2016 Standard
- CWNA (Certified Wireless Network Administrator)
- 802.11 Frame Types: https://www.wireshark.org/docs/dfref/w/wlan.html
- PMF/802.11w: IEEE 802.11w-2009
