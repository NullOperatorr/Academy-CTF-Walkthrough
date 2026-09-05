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

**Service & Port Scanning**

```bash
sudo nmap -Pn -sC -sS -sV -p- -T4 192.168.38.138
```

<img width="970" height="863" alt="image" src="https://github.com/user-attachments/assets/a9bfb7f8-31f1-43ce-abe3-a63dcff96cb7" />


Based on the Nmap scan, we found three open ports:

| Port | Service | Version |
|------|---------|---------|
| 21   | FTP     | vsftpd 3.0.3 |
| 22   | SSH     | OpenSSH 7.9p1 |
| 80   | HTTP    | Apache Httpd 2.4.38 |  

We will investigate each service to identify potential vulnerabilities and possible ways to gain initial access.

---

## 2-Enumeration

The plan is to start by enumerating FTP, as it is one of the easiest services to enumerate when it is not configured correctly and may contain valuable information.  
Then, we will move to the web application on port 80, followed by SSH on port 22.

- **(A) FTP Anonymous User allowed:**

<img width="708" height="94" alt="image" src="https://github.com/user-attachments/assets/4dd86d42-99a0-4ecb-97a5-a0c3d9e1236e" />

 An anonymous FTP account allows users to access an FTP server without providing a valid username and password, which can potentially expose sensitive files.  

 **Username: Anonymous**  
 **Password: Anonymous**  

 ```bash
ftp 192.168.38.138
enter Username & Password
ls
get note.txt
exit
cat note.txt
```

<img width="1269" height="530" alt="image" src="https://github.com/user-attachments/assets/3243763a-aacb-4c8d-b169-9cca6c1cf172" />
<img width="1289" height="442" alt="image" src="https://github.com/user-attachments/assets/3fc83256-cf43-4c1c-ac59-bfa99d010849" />  

- Now we have extracted some valuable information:  
1- Grimmie an Administrator uses the same password which is good if we can find.  
2- Authenticated user ID & Password (Hashed) that we will search where to use them later.  

```bash
Username: 10201321
Password: cd73502828457d15655bbd7a63fb0bc8
```

- Now lets try to crack this password.   

```bash
hash-identifier cd73502828457d15655bbd7a63fb0bc8
hashcat -m  0 cd73502828457d15655bbd7a63fb0bc8 /usr/share/wordlists/rockyou.txt
```


<img width="875" height="435" alt="image" src="https://github.com/user-attachments/assets/c767bab9-e34c-4da6-a516-1adff39757ef" />
<img width="1089" height="940" alt="image" src="https://github.com/user-attachments/assets/cc37677d-aebc-4811-be1e-6c81d673b24e" />


```bash
Username: 10201321
Password: student
```

- We concluded that the hashed password is **MD5** and then cracked it with hashcat.
- We need an endpoint to authenticate this user which of course will be on the WebApp on port 80.
- All of the above concluded an FTP information disclosure that must be configured right.

 
 
  **(B) WebApp & Directory Enumeration:**

We will now visit the Academy web application on port 80. Everything appears to be normal. we will move on to directory enumeration, hoping to find valuable directories or files that could help with exploitation.   

```bash
http://192.168.38.138
```

  <img width="1284" height="720" alt="image" src="https://github.com/user-attachments/assets/53d8fe22-71fe-4f03-9959-4f8286527af5" />  

We will move on to directory enumeration, hoping to find valuable directories or files that could help with exploitation. 

```bash
sudo ffuf -u http://192.168.38.138:80/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="1266" height="918" alt="image" src="https://github.com/user-attachments/assets/1c0e656e-2182-4bfe-8d1d-c6c1b2cefc46" /> 

We found two interesting directories: **phpmyadmin** and **academy**.
We will now use the credentials we found earlier and try to log in to the **Academy** web application.


<img width="1278" height="735" alt="image" src="https://github.com/user-attachments/assets/386ca65f-9e16-498d-8a9a-8fce068bb14d" />



