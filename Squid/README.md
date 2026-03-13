lab Squid

target 192.168.56.189

```bash
sudo nmap -sC -sV squid -sT -vvv -p- -Pn -T5 --min-rate=5000
```

```bash
Nmap scan report for squid (192.168.56.189)
Host is up, received user-set (0.00040s latency).
Scanned at 2026-03-13 09:30:45 UTC for 120s
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON  VERSION
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
3128/tcp  open  http-proxy    syn-ack Squid http proxy 4.14
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.14
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
```


SMB

Right away I notice that 445 - SMB seems to be open which is on the top of my priorities for Windows machines. I go ahead and investigate it using various tools.


