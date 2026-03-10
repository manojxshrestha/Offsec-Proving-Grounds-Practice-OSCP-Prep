lab  Hepet

target 192.168.52.140

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

<img width="931" height="21" alt="image" src="https://github.com/user-attachments/assets/1ffb0dac-f904-418f-bc53-203659d5c8a1" />

<img width="2478" height="901" alt="image" src="https://github.com/user-attachments/assets/99883e06-85ba-4632-8ca7-8f8a75b04dcd" />

We get some names from the website, we could try and brute force later with them (some of these have also been found on [[#25/TCP - SMTP]]).

```bash
ela
charlotte
magnus
agnes
jonas
martha
```

Furthermore we also notice that jonas has SicMundusCreatusEst as his title on the website, could be a password?

note: Is that a Dark reference?

```bash
python3 dirsearch.py -u http://192.168.52.140:443/ -e* --output-file dirsearch-443.txt -r
```
<img width="1052" height="416" alt="image" src="https://github.com/user-attachments/assets/42026a04-ecde-4152-880f-f4393c063f7a" />

This didn’t really yield anything valuable


2224/TCP - HTTP
<img width="993" height="373" alt="image" src="https://github.com/user-attachments/assets/44ccbcdb-8399-428d-92fa-80431f60d6c1" />

<img width="1007" height="784" alt="image" src="https://github.com/user-attachments/assets/a93d86ba-f513-4350-8eeb-49bd9e59c310" />

<img width="669" height="26" alt="image" src="https://github.com/user-attachments/assets/e28529f1-6a10-4ad7-8e66-c61ec7d20404" />

I went ahead and started looking for exploits regarding this version running:

<img width="1164" height="419" alt="image" src="https://github.com/user-attachments/assets/6d1899f4-ea18-4f60-a118-f19e5fb2ddf0" />




E-Mail Related Port Pentest

After building a custom username and password wordlist, I proceeded with SMTP enumeration on port 25 using the tool smtp-user-enum.
```bash
smtp-user-enum -M VRFY -U users.txt -t 192.168.52.140
```
<img width="1017" height="583" alt="image" src="https://github.com/user-attachments/assets/3bc3ac45-2815-4151-b42f-0c2efe0882b5" />


The enumeration results revealed 5 valid SMTP users: agnes, magnus, charlotte, martha, jonas . These were noted as valid email accounts on the mail server.

Previously, I had identified potential valid credentials: jonas:SicMundusCreatusEst .

Using these credentials, I attempted to access the IMAP service running on port 143. Upon accessing the mailbox, I found:

    1 folder: INBOX
    5 email messages inside
