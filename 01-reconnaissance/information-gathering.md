# Information Gathering & Service Enumeration

## Listing Wireless Access Points Within Range

### Determine Used Driver

```bash
iw list
```

If it says "nl80211 not found" we are using **ieee80211** drivers. Else we are using **mac80211** and the "iw list" output will print wireless card information.

### Display SSID and their Channel (mac80211 driver)

```bash
iw dev wlan0 scan | egrep "DS Parameter set|SSID"
```

### Display SSID and their Channel (ieee80211 driver)

```bash
iwlist wlan0 scanning | egrep "ESSID|Channel"
```

### Display Wireless Card MAC Address

```bash
macchanger -s mon0
```

### Increase TX Power

> **Note:** TX power depends on your country.

```bash
iw reg set B0
iwconfig wlan0 <txpower>|NmW|NdBm|off|auto>
```

**Example:**
```bash
iw reg set B0 && iwconfig wlan0 txpower 30
```

### Set Monitor Mode

```bash
airmon-ng check kill
ifconfig $IFACE down
iwconfig $IFACE mode monitor
ifconfig $IFACE up
```

**Alternative:**
```bash
airmon-ng stop ath0
airmon-ng start wifi0
iwconfig ath0 channel 6
```

### Delete Interface

```bash
iw dev $IFACE interface del
```

**Verify:**
```bash
iwconfig $IFACE
```

### Restart Network Manager

```bash
systemctl restart NetworkManager.service
```

### List Networks

```bash
airodump-ng $IFACE -c $CHANNEL -e $ESSID
```

### Monitor 2.4 GHz (w/ Channel Hopping)

```bash
airodump-ng mon0
```

### Monitor 5 GHz (w/ Channel Hopping)

```bash
airodump-ng --band a mon0
```

### Monitor Specific Channel

```bash
airodump-ng -c <channel> mon0
```

### Monitor Clients on AP

```bash
airodump-ng -c <channel> --bssid <MAC_AP> -w <capture_file> mon0
```

### Packet Capture

```bash
airodump-ng wlan0mon
```

### Find Hidden SSID

```bash
airodump-ng -c <channel> --bssid <MAC_AP> mon0
aireplay-ng --deauth 20 -a <MAC_AP> mon0
```

### Get Hidden SSID with Clients

```bash
aireplay-ng --deauth $NUMBER_DEAUTH_PACKETS -a $AP_MAC -c $CLIENT_MAC $IFACE
```

### Get Hidden SSID without Clients

**Dictionary Attack:**
```bash
mdk3 $IFACE p -t $AP_MAC -f $DICTIONARY_PATH
```

**Bruteforce Attack:**
```bash
mdk3 $IFACE p -t $AP_MAC -c $AP_CHANNEL -b $CHARACTER_SET
```

### All-in-one w/ Web UI

#### Viewing Network Details
- Press `i` to see the details of a selected network.

#### Sorting the Network List
- Press `s` to sort the list of detected networks.

#### Displaying Current Users
- Press `c` to display the users connected to the current network.

```bash
kismet -c mon0
```

### Generate Graph of Probe Requests of Nearby Devices

> Interesting to rebuild devices' PNLs & spawn rogue AP in smart way.

```bash
airodump-ng -r targenet.pcap -w TARGETNET
airgraph-ng -i TARGETNET-01.csv -g CPG -o targetnet-pnl.png
```

**Aircrack-ng Handbook all in one:** https://github.com/lucthienphong1120/Aircrack-ng-handbook

---

## GeoWiFi: WiFi Geolocation Data Search Tool

GeoWiFi is a powerful tool designed to search WiFi geolocation data by BSSID and SSID using various public databases.

### Configuration

GeoWiFi uses a configuration file located at `gw_utils/config.yaml` to store API keys and other settings.

```yaml
wigle_auth: "your_wigle_encoded_key"
google_api: "your_google_api_key"
combain_api: "your_combain_api_key"
no-ssl-verify: false
```

### Search by BSSID

```bash
python3 geowifi.py -s bssid 00:11:22:33:44:55
```

### Search by SSID

```bash
python3 geowifi.py -s ssid "NetworkName"
```

### Output in JSON Format

```bash
python3 geowifi.py -s bssid 00:11:22:33:44:55 -o json
```

### Output in Map Format

```bash
python3 geowifi.py -s ssid "NetworkName" -o map
```
