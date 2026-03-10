lab  Hepet

target 192.168.59.140

```bash
sudo nmap -sC -sV hepet -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash

PORT      STATE SERVICE        REASON  VERSION
25/tcp    open  smtp           syn-ack Mercury/32 smtpd (Mail server account Maiser)
| smtp-commands: localhost Hello hepet; ESMTPs are:, TIME, SIZE 0, HELP
|_ Recognized SMTP commands are: HELO EHLO MAIL RCPT DATA RSET AUTH NOOP QUIT HELP VRFY SOML Mail server account is 'Maiser'.
79/tcp    open  finger         syn-ack Mercury/32 fingerd
| finger: Login: Admin         Name: Mail System Administrator\x0D
| \x0D
|_[No profile information]\x0D
105/tcp   open  ph-addressbook syn-ack Mercury/32 PH addressbook server
106/tcp   open  pop3pw         syn-ack Mercury/32 poppass service
110/tcp   open  pop3           syn-ack Mercury/32 pop3d
|_pop3-capabilities: EXPIRE(NEVER) APOP UIDL TOP USER
135/tcp   open  msrpc          syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn    syn-ack Microsoft Windows netbios-ssn
143/tcp   open  imap           syn-ack Mercury/32 imapd 4.62
|_imap-capabilities: CAPABILITY X-MERCURY-1A0001 OK complete AUTH=PLAIN IMAP4rev1
443/tcp   open  ssl/http       syn-ack Apache httpd 2.4.46 (OpenSSL/1.1.1g PHP/7.3.23)
|_ssl-date: TLS randomness does not represent time
|_http-title: Time Travel Company Page
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
| http-methods:
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
| tls-alpn:
|_  http/1.1
445/tcp   open  microsoft-ds?  syn-ack
2224/tcp  open  http           syn-ack Mercury/32 httpd
|_http-title: Mercury HTTP Services
| http-methods:
|_  Supported Methods: GET HEAD
5040/tcp  open  unknown        syn-ack
7680/tcp  open  pando-pub?     syn-ack
8000/tcp  open  http           syn-ack Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
| http-methods:
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-title: Time Travel Company Page
11100/tcp open  vnc            syn-ack VNC (protocol 3.8)
| vnc-info:
|   Protocol version: 3.8
|   Security types:
|_    Unknown security type (40)
20001/tcp open  ftp            syn-ack FileZilla ftpd 0.9.41 beta
|_ftp-bounce: bounce working!
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -r--r--r-- 1 ftp ftp            312 Oct 20  2020 .babelrc
| -r--r--r-- 1 ftp ftp            147 Oct 20  2020 .editorconfig
| -r--r--r-- 1 ftp ftp             23 Oct 20  2020 .eslintignore
| -r--r--r-- 1 ftp ftp            779 Oct 20  2020 .eslintrc.js
| -r--r--r-- 1 ftp ftp            167 Oct 20  2020 .gitignore
| -r--r--r-- 1 ftp ftp            228 Oct 20  2020 .postcssrc.js
| -r--r--r-- 1 ftp ftp            346 Oct 20  2020 .tern-project
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 build
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 config
| -r--r--r-- 1 ftp ftp           1376 Oct 20  2020 index.html
| -r--r--r-- 1 ftp ftp         425010 Oct 20  2020 package-lock.json
| -r--r--r-- 1 ftp ftp           2454 Oct 20  2020 package.json
| -r--r--r-- 1 ftp ftp           1100 Oct 20  2020 README.md
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 src
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 static
|_-r--r--r-- 1 ftp ftp            127 Oct 20  2020 _redirects
| ftp-syst:
|_  SYST: UNIX emulated by FileZilla
33006/tcp open  mysql          syn-ack MariaDB 10.3.24 or later (unauthorized)
49664/tcp open  msrpc          syn-ack Microsoft Windows RPC
49665/tcp open  msrpc          syn-ack Microsoft Windows RPC
49666/tcp open  msrpc          syn-ack Microsoft Windows RPC
49667/tcp open  msrpc          syn-ack Microsoft Windows RPC
49668/tcp open  msrpc          syn-ack Microsoft Windows RPC
49669/tcp open  msrpc          syn-ack Microsoft Windows RPC
Service Info: Hosts: localhost, www.example.com; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Metric sh*t ton of ports, let’s start at the top.

25/TCP - SMTP
<img width="963" height="28" alt="image" src="https://github.com/user-attachments/assets/cd448449-2121-4d77-aabf-2ab2bb8e310e" />

```bash
smtp-user-enum -t hepet -U /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```
<img width="1051" height="666" alt="image" src="https://github.com/user-attachments/assets/9f83ae61-8b20-4448-9066-f4832de3e491" />

This gave me list of existing SMTP usernames.

79/TCP - Finger
<img width="698" height="25" alt="image" src="https://github.com/user-attachments/assets/cb1a3fbe-c6ce-46b4-b258-8df78057a77f" />

<img width="611" height="74" alt="image" src="https://github.com/user-attachments/assets/11d1234f-965d-4c05-a38f-1aa99089cafb" />


This is the same info nmap gave us. After checking hacktricks  https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-finger.html I didn’t find anything useful.


443/TCP - HTTPS

