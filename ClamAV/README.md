lab ClamAV

target 192.168.55.42

recon

```bash
sudo nmap -sC -sV clamav -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV clamav -sT -vvv -p- -Pn -T5 --min-rate=5000

PORT    STATE SERVICE     REASON  VERSION
22/tcp  open  ssh         syn-ack OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
25/tcp  open  smtp        syn-ack Sendmail 8.13.4/8.13.4/Debian-3sarge3
80/tcp  open  http        syn-ack Apache httpd 1.3.33 ((Debian GNU/Linux))
| http-methods:
|   Supported Methods: GET HEAD OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Ph33r
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
139/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp open  smux        syn-ack Linux SNMP multiplexer
445/tcp open  netbios-ssn syn-ack Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel
```

Port 80
There’s a webserver (Apache 1.3.33) hosted on the box, and the default page shows the following binary code:

<img width="811" height="26" alt="image" src="https://github.com/user-attachments/assets/1ebf69bd-829a-4326-9215-952a467b9bed" />

I went ahead and ran a searchsploit search for all the versions, and did find one that seemed interesting for the Apache server:

```bash
searchsploit "Apache 1.3.33"
```
<img width="1044" height="144" alt="image" src="https://github.com/user-attachments/assets/f17d9ee6-a114-4418-a86a-42a84b39821a" />
