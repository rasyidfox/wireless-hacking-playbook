# Wordlist Generation & Password Cracking

## Wordlist Generation

### Crunch
```bash
# Generate wordlist dengan panjang 6-8 karakter
crunch 6 8 abcdefghijklmnopqrstuvwxyz -o wordlist.txt

# Dengan angka
crunch 8 8 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt

# Dengan karakter khusus
crunch 8 8 "abcdefghijklmnopqrstuvwxyz0123456789!@#$" -o wordlist.txt

# Dari pattern (birthdate)
crunch 11 11 -t 1990@@@@@@% -o wordlist.txt
# @ = lowercase, % = numbers, ^ = uppercase

# Dengan rules
crunch 8 8 -f /usr/share/crunch/charset.lst mixalpha -o wordlist.txt
```

### CeWL (Custom Wordlist by Crawling)
```bash
# Crawl website untuk wordlist
cewl https://company.com -w wordlist.txt -d 3

# Dengan minimum length
cewl https://company.com -w wordlist.txt -m 6

# Dengan metadata (PDF, DOCX)
cewl https://company.com -w wordlist.txt --meta

# Generate dari profile
cewl https://company.com/team -w team_wordlist.txt
```

### Mentalist (GUI)
```bash
# Visual wordlist generator
# https://github.com/sc0tfree/mentalist
python3 mentalist.py
```

### Username Generator
```bash
# CeWL untuk username
cewl https://company.com -w usernames.txt -d 2

# Manual generation
for first in john jane bob alice; do
    for last in doe smith jones; do
        echo "${first}.${last}"
        echo "${first}${last}"
        echo "${first[0]}${last}"
    done
done > usernames.txt
```

---

## Hashcat Rules

### Built-in Rules
```bash
# Best rules
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Combinator
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -r /usr/share/hashcat/rules/combinator.rule

# D3ad0ne rule (comprehensive)
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -r /usr/share/hashcat/rules/d3ad0ne.rule

# Dive rule (very aggressive)
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -r /usr/share/hashcat/rules/dive.rule
```

### Custom Rules
```bash
# custom.rule
# Tambah tahun di akhir
$2$0$2$0
$2$0$2$1
$2$0$2$2

# Leet speak
sa@
se3
si1
so0
ss$

# Capitalize first letter
c
```

**Apply custom rules:**
```bash
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -r custom.rule
```

---

## Attack Modes

### Dictionary Attack
```bash
hashcat -m 16800 capture.16800 -a 0 wordlist.txt
```

### Brute Force (Mask)
```bash
# 8 karakter lowercase
hashcat -m 16800 capture.16800 -a 3 ?l?l?l?l?l?l?l?l

# Dengan angka
hashcat -m 16800 capture.16800 -a 3 ?u?l?l?l?l?l?d?d?d

# Pattern (birth year + name)
hashcat -m 16800 capture.16800 -a 3 19?d?d?l?l?l?l
```

### Combinator Attack
```bash
hashcat -m 16800 capture.16800 -a 1 wordlist1.txt wordlist2.txt
```

### Hybrid Attack
```bash
# Dictionary + mask
hashcat -m 16800 capture.16800 -a 6 wordlist.txt ?d?d?d?d

# Mask + dictionary
hashcat -m 16800 capture.16800 -a 7 ?d?d?d?d wordlist.txt
```

### Rule-based Attack
```bash
hashcat -m 16800 capture.16800 -a 0 wordlist.txt -r rules.rule
```

---

## GPU Optimization

### NVIDIA
```bash
# Install
sudo apt install nvidia-driver nvidia-cuda-toolkit

# Status
nvidia-smi

# Hashcat benchmark
hashcat -b -m 16800

# Optimized kernel
hashcat -m 16800 capture.16800 -a 0 wordlist.txt -O
```

### AMD
```bash
# Install
sudo apt install rocm-opencl-dev

# Hashcat
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    --opencl-device-types 1,2
```

---

## Wordlist Resources

| Wordlist | Size | Use Case |
|----------|------|----------|
| **rockyou.txt** | 14M | General (most common) |
| **SecLists** | Multi | Security testing |
| **Probable-Wordlists** | Multi | Smart generation |
| **WPA-Wordlists** | 100M+ | WiFi specific |

### Download
```bash
# Kali default wordlists
ls /usr/share/wordlists/

# rockyou
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# SecLists
sudo apt install seclists

# Probable-Wordlists
git clone https://github.com/berzerk0/Probable-Wordlists
```

---

## WiFi-Specific Wordlists

### Generate dari Company Info
```bash
#!/bin/bash
# generate_company_wordlist.sh
COMPANY="CorpName"
YEAR=$(date +%Y)

cat << EOF > company_wordlist.txt
${COMPANY}${YEAR}
${COMPANY}!
${COMPANY}@${YEAR}
${COMPANY}123
${COMPANY}_wifi
wifi${COMPANY}
${COMPANY}wifi
${COMPANY}.com
password
Password1
Welcome1
${COMPANY}2024
${COMPANY}2025
EOF
```

### Common WiFi Passwords
```bash
# Pattern-based
cat << EOF > wifi_passwords.txt
12345678
password
123456789
qwerty123
wifi12345
password1
welcome1
admin123
letmein1
1234567890
EOF
```

---

## Optimasi Cracking

### Performance Tips
```bash
# Workload tuning
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    -w 3  # High workload

# Segment wordlist
split -l 1000000 big_wordlist.txt chunk_
for chunk in chunk_*; do
    hashcat -m 16800 capture.16800 -a 0 $chunk
done

# Session management
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    --session mysession

# Resume
hashcat --session mysession --restore
```

### Time Estimation
```bash
# Benchmark
hashcat -b -m 16800

# Estimate
hashcat -m 16800 capture.16800 -a 3 ?l?l?l?l?l?l?l?l \
    --keyspace

# Show progress
hashcat -m 16800 capture.16800 -a 0 wordlist.txt \
    --status --status-timer=10
```

---

## Referensi

- **Hashcat:** https://hashcat.net/hashcat/
- **SecLists:** https://github.com/danielmiessler/SecLists
- **Probable-Wordlists:** https://github.com/berzerk0/Probable-Wordlists
- **CeWL:** https://digininja.github.io/CeWL/
- **Crunch:** https://sourceforge.net/projects/crunch-wordlist/
