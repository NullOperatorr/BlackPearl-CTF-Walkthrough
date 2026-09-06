# BlackPearl-CTF-Walkthrough
CyberLab-14


## Overview

BlackPearl is a Linux-based machine that demonstrates how DNS misconfigurations, web vulnerabilities, and local privilege escalation can be chained together to gain root access.
(https://tcm-sec.com/)

- Operating System:	Linux
- Difficulty:	Medium
- Goal:	Obtain Root Access

**Enviroment:**

- Kali Machine (Attacker).
- Dev (.ovf) VM.
- Make sure both VMs on the same virtual network (NAT).

  ----


## Reconnaissance

**- Host Discovery:**

We can discover **BlackPerl** via Ping Sweep (nmap) or Arp scan (netdiscover) and the discovered target has IP-Address (192.168.38.128).  

```bash
netdiscover -r 192.168.38.0/24  
nmap -sn 192.168.38.0/24
```

<img width="866" height="687" alt="image" src="https://github.com/user-attachments/assets/17da64f0-fc7b-4be6-a092-483cbb3ddfb5" />


---


## Enumeration

**1- Port Scanning:**

```bash
nmap -Pn -sC -sS -sV -p- -T4 192.168.38.128
```

<img width="923" height="413" alt="image" src="https://github.com/user-attachments/assets/287b178b-39fb-4805-b318-d59e548e2be2" />


| Port | Service | Version / Details |
|------|---------|-------------------|
| 22 | SSH | OpenSSH 7.9p1 |
| 80 | HTTP | nginx 1.14.2 |
| 53 | DNS | NA |


**2- Subdirectory Enumeration:**

- On port (80)



```bash
ffuf -u http://192.168.38.140:80/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
