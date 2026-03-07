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

<img width="427" height="545" alt="image" src="https://github.com/user-attachments/assets/063eb65f-b0a6-45ff-bdb7-d751a373093e" />

now lets use the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — Which turned out not.

```bash
impacket-GetNPUsers nagoya-industries.com/ -usersfile users.txt
```
<img width="974" height="604" alt="image" src="https://github.com/user-attachments/assets/d11c8143-0e8a-4ae3-8114-0383a726fac0" />


I used the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — WHich turned out not.

Also confirmed that the naming format is the correct one using kerbrute :
