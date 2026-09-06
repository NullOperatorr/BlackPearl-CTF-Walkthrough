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

```bash
http://192.168.38.128:80
```

<img width="1291" height="506" alt="image" src="https://github.com/user-attachments/assets/53d82a46-37e0-4dcd-a5a1-111716cf6401" />
<img width="915" height="505" alt="image" src="https://github.com/user-attachments/assets/18837aa9-cb47-4c24-ba54-efbe22e7eb37" />


- We found an Email which maybe useful in the future. (alek@blackpearl.tcm)
  

```bash
ffuf -u http://192.168.38.128:80/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="1175" height="918" alt="image" src="https://github.com/user-attachments/assets/f08635d2-5599-4866-820e-9a515d1bee2c" />
<img width="1259" height="573" alt="image" src="https://github.com/user-attachments/assets/88ab2d65-9cc7-4b92-bfcc-147a27807536" />

- The subdirectory did not reveal anything useful; it was just a trap, so we will proceed with DNS enumeration.

  **3- DNS Enumeration:**

  ```bash
  dnsrecon -r 127.0.0.0/24 -n 192.168.38.128 -d blank
  ```

<img width="1002" height="329" alt="image" src="https://github.com/user-attachments/assets/5961f5a3-a54b-4d11-922e-2a5667fc239a" />  

- We have now confirmed the domain name, which we discovered twice earlier. To ensure proper DNS resolution, we will map the domain name to the machine’s IP address in `/etc/hosts`.

  ```bash
  sudo mousepad /etc/hosts
  192.168.38.128     blackperl.tcm
  ```

  <img width="794" height="635" alt="image" src="https://github.com/user-attachments/assets/13d1fe97-8e5b-4cee-8c1f-9497c77a43dc" />


