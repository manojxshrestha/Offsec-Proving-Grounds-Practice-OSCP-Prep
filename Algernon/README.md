lab Algernon

target 192.168.53.65

```bash
sudo nmap -sC -sV algernon -sT -vvv -p- -Pn -T5 --min-rate=5000
```

```bash

PORT      STATE    SERVICE       REASON      VERSION
21/tcp    open     ftp           syn-ack     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  09:31PM       <DIR>          ImapRetrieval
| 02-21-25  01:39AM       <DIR>          Logs
| 04-29-20  09:31PM       <DIR>          PopRetrieval
|_02-21-25  01:39AM       <DIR>          Spool
| ftp-syst:
|_  SYST: Windows_NT
80/tcp    open     http          syn-ack     Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp   open     msrpc         syn-ack     Microsoft Windows RPC
139/tcp   open     netbios-ssn   syn-ack     Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds? syn-ack
5040/tcp  open     unknown       syn-ack
9998/tcp  open     http          syn-ack     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 9D7294CAAB5C2DF4CD916F53653714D5
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: Fri, 21 Feb 2025 09:43:48 GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
|_http-server-header: Microsoft-IIS/10.0
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /interface/root
17001/tcp open     remoting      syn-ack     MS .NET Remoting services
49664/tcp open     msrpc         syn-ack     Microsoft Windows RPC
49665/tcp open     msrpc         syn-ack     Microsoft Windows RPC
49666/tcp open     msrpc         syn-ack     Microsoft Windows RPC
49667/tcp open     msrpc         syn-ack     Microsoft Windows RPC
49668/tcp open     msrpc         syn-ack     Microsoft Windows RPC
49669/tcp open     msrpc         syn-ack     Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
|_clock-skew: 0s
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 55999/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 7682/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 62932/udp): CLEAN (Timeout)
|   Check 4 (port 16225/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time:
|   date: 2025-02-21T09:43:50
|_  start_date: N/A
```

The port scanning results reveal several interesting open ports on the Windows machine: 21, 80, 9998, and 170001.

    Port 21: FTP
    Ports 80 & 9998: HTTP
    Port 170001: MS .NET Remoting Services

Initially, I attempted to access FTP (port 21) and logged in using an anonymous user. The login was successful, but there was no useful information available on the FTP server.
<img width="882" height="396" alt="image" src="https://github.com/user-attachments/assets/eab36ccd-7584-487b-bb55-d19cdadf375d" />

Next, I tested port 80 (HTTP). After accessing it, the web server only displayed the default IIS page.
<img width="1445" height="775" alt="image" src="https://github.com/user-attachments/assets/7111cca5-16f0-43fe-8475-c3820b319a06" />

Then, I tried accessing port 9998 (HTTP) via http://192.168.53.65:9998/. Upon access, the website redirected me to http://192.168.53.65:9998/interface/root#/login.

<img width="1442" height="800" alt="image" src="https://github.com/user-attachments/assets/78eaf54b-ccc3-4963-9f20-c076ab1a46d9" />

Since no email or password was known for login, and the SmarterMail version was also unknown, I inspected the webpage elements and found that the SmarterMail version in use was 100.0.6919 (build 6919).

<img width="1109" height="331" alt="image" src="https://github.com/user-attachments/assets/92f2eb33-b675-4520-b424-889b4e89f3a4" />

<img width="429" height="24" alt="image" src="https://github.com/user-attachments/assets/4680ac43-378c-4ac5-a4ac-8b5f5b100bce" />

After searching online for an exploit targeting SmarterMail 6919, I found a relevant entry on ExploitDB. https://www.exploit-db.com/exploits/49216 According to the information, SmarterMail versions before build 6985 expose a .NET remoting endpoint, which is vulnerable to a .NET deserialization attack. This aligns with port 170001 (MS .NET Remoting Services) being open.

Initial Access:

I downloaded the public exploit from github instead https://github.com/devzspy/CVE-2019-7214 and modified the HOST and LHOST parameters. The LPORT parameter was optional, but I left it at its default value, port 4444, for the reverse shell listener.

<img width="348" height="100" alt="image" src="https://github.com/user-attachments/assets/ac72e9b3-fa40-4a3d-ade6-06b359f8a02c" />

Foothold

<img width="1235" height="165" alt="image" src="https://github.com/user-attachments/assets/85916718-9bac-4f92-a615-b1db37f1eb03" />

<img width="716" height="81" alt="image" src="https://github.com/user-attachments/assets/0a83d5d4-8507-4862-bcc4-a8e500df8d8a" />


This machine only have proof.txt and after submitted in portal the progress will 100% done.
