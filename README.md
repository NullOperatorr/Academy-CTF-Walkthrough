# Academy-CTF-Walkthrough
CyberLab-12


## #Overview

Academy is a vulnerable machine from TCM Security (https://tcm-sec.com/). The objective is to gain root access and capture the flag.  
It involves a chain of vulnerabilities and misconfigurations across different services, which eventually leads to full system compromise.  

**Machine:**  
https://drive.google.com/file/d/1u4628J7AwEzFCS3gWZbJgv-lhGzwmrvf/view?source=post_page-----e891243c61a8-----------------------------------------

**Enviroment:**  

1- Kali Linux (Attacker).  
2- Academy (.ova) VM.
3- Make sure that both on the same Virtual network (NAT).  

---

## #Methodology

We will walkthrough each of these following steps one by one:

Reconnaissance → Enumeration → Credential Discovery → File Upload (RCE)   → Local Enumeration → Credential Reuse → CronJobs Enumeration → Root Access


---

## 1-Information Gathering:

**Host Discovery**

```bash
ip a
sudo netdiscover -r 192.168.38.0/24
sudo nmap -sn 192.168.38.0/24
ping 192.168.38.138
```
<img width="1036" height="859" alt="image" src="https://github.com/user-attachments/assets/d04aa7df-7af9-4f4a-96a1-4e3d0708a6fb" />

We can identify Academy IP-Address by netdiscover (Arp Scan) or by Nmap (Ping Sweep), Eventually the IP (192.168.38.138).
  
