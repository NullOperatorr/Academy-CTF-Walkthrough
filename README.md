# Academy-CTF-Walkthrough
CyberLab-12


## #Overview

Academy is a vulnerable machine from TCM Security (https://tcm-sec.com/). The objective is to gain root access and capture the flag.  
It involves a chain of vulnerabilities and misconfigurations across different services, which eventually leads to full system compromise.  

https://drive.google.com/file/d/1u4628J7AwEzFCS3gWZbJgv-lhGzwmrvf/view?source=post_page-----e891243c61a8-----------------------------------------
 
## #Methodology

We will walkthrough each of these following steps one by one:

Reconnaissance → Enumeration → Credential Discovery → File Upload (RCE) → Local Enumeration → Credential Reuse → CronJobs Enumeration → Root Access



