# Devvortex -htb(Compi)


nmap:
	22, 80 , 8181


SubDirectory enum :
	dev.devvortex.htb ---> http://dev.devvortex.htb/administrator ---> Joomla Admin login page.

in dev.devvortex.htb/README.txt ---> we can see joomla  version no.

v4.2 is vulnerable to unauthenticated info disclouser and also RCE.

google it, and find articles , use the POC. get users.

USing the POC i was able to find a MySQL password  
`P4ntherg0t1n5r3c0n##`

It was also earlier revealed to me using the poc from exploitDB , that there are 2 users `lewis` and `logan paul`. 

Lewis user was in superuser grp.

And the above password was for lewis:
lewis:`P4ntherg0t1n5r3c0n##`

---

Shell as WWW-DATA ENUM::
--

https://vulncheck.com/blog/joomla-for-rce
following this cve i was able to get a webshell in the joomla admin login page.

After getting shell , spwn tty: stablize shell
	python3 -c "import pty;pty.spawn('/bin/bash')"

/etc/passwd revealed logan as a user here.

```
Remember we got mysql creds before, 
	Since now we have a foothold inside the machine ,
		We can now use the mysql creds to further enum.
			`mysql -u lewis -p`
				enter password:
					we also discoverd `sd4fg_users tables`
						now select that and grab user logan's password hash:
`logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12`
&
`lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u`

```

Now use john to crack the hash:
	logan's password : `tequieromucho`

now login as logan in the terminal

----

User Logan ENUM:
--
got userflag.

sudo -l  ---> tells us we can run apport-cli using sudo priv.

googling this points us to a exploit:

---

PRIV ESC:
--
https://github.com/canonical/apport/commit/e5f78cc89f1f5888b6a56b785dddcb0364c48ecb
use the steps and get root;

*Note*:
	If the folder doesnt have any crash reports , then you can  just create an crash via   
```
sleep 13 &  
killall -SIGSEGV sleep
```
and then use the steps in the POC to get root.

pwnd..
