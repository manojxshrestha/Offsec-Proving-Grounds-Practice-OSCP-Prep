

add host first sudo nano /etc/hosts

<img width="803" height="232" alt="image" src="https://github.com/user-attachments/assets/cbc1f15e-05b7-423a-ab60-d183bf3f6910" />

Starting Nmap with usual flags and a full port scan.

```bash
sudo nmap -sC -sV hokkaido -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV hokkaido -sT -vvvv -p- -Pn -T5 --min-rate=5000

PORT      STATE    SERVICE        REASON      VERSION
53/tcp    open     domain         syn-ack     Simple DNS Plus
80/tcp    open     http           syn-ack     Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open     kerberos-sec   syn-ack     Microsoft Windows Kerberos (server time: 2025-05-16 14:02:02Z)
135/tcp   open     msrpc          syn-ack     Microsoft Windows RPC
139/tcp   open     netbios-ssn    syn-ack     Microsoft Windows netbios-ssn
389/tcp   open     ldap           syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
445/tcp   open     microsoft-ds?  syn-ack
464/tcp   open     kpasswd5?      syn-ack
593/tcp   open     ncacn_http     syn-ack     Microsoft Windows RPC over HTTP 1.0
636/tcp   open     ssl/ldap       syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
1433/tcp  open     ms-sql-s       syn-ack     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   192.168.128.40:1433:
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ms-sql-info:
|   192.168.128.40:1433:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open     ldap           syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
3269/tcp  open     ssl/ldap       syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
3389/tcp  open     ms-wbt-server  syn-ack     Microsoft Terminal Services
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Issuer: commonName=dc.hokkaido-aerospace.com
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-05-15T13:51:51
| Not valid after:  2025-11-14T13:51:51
| MD5:   37b4:2769:019c:9068:c300:46e5:9ca9:3e0e
| SHA-1: 25cf:2462:c0b0:737c:55cf:04b2:695c:9b14:96ee:d53d
| rdp-ntlm-info:
|   Target_Name: HAERO
|   NetBIOS_Domain_Name: HAERO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hokkaido-aerospace.com
|   DNS_Computer_Name: dc.hokkaido-aerospace.com
|   DNS_Tree_Name: hokkaido-aerospace.com
|   Product_Version: 10.0.20348
|_  System_Time: 2025-05-16T14:02:51+00:00
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
5985/tcp  open     http           syn-ack     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8530/tcp  open     http           syn-ack     Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: 403 - Forbidden: Access is denied.
8531/tcp  open     unknown        syn-ack
9389/tcp  open     mc-nmf         syn-ack     .NET Message Framing
47001/tcp open     http           syn-ack     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49684/tcp open     ncacn_http     syn-ack     Microsoft Windows RPC over HTTP 1.0
58538/tcp open     ms-sql-s       syn-ack     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   192.168.128.40:58538:
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
| ms-sql-info:
|   192.168.128.40:58538:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 58538
```

