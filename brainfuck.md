
[OS] Linux
[Web-Technology] nginx
[Hostname] Brainfuck

[IP] : 10.10.10.17

[USERS] :: admin, administrator, orestis (orestis@brainfuck.htb)


https://sup3rs3cr3t.brainfuck.htb/ --> enumerate
https://www.brainfuck.htb ---> not reachable
htts://brainfuck.htb ---> wordpress 4.7.3 ---> wpsscan


Nmap: 
```

PORT STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE SASL(PLAIN) PIPELINING TOP USER CAPA RESP-CODES UIDL
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more LOGIN-REFERRALS ID AUTH=PLAINA0001 listed SASL-IR LITERAL+ ENABLE have post-login capabilities Pre-login OK IDLE IMAP4rev1
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
| tls-alpn: 
|_  http/1.1
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_http-server-header: nginx/1.10.0 (Ubuntu)
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


There are 3 domains :
https://sup3rs3cr3t.brainfuck.htb/ --> enumerate
https://www.brainfuck.htb ---> not reachable
htts://brainfuck.htb ---> wordpress 4.7.3 ---> wpsscan


Wps scan of `https://brainfuck.htb` found a plugin that is out of date .

----
Vulnerable WordPress plugin
--
Searchsploit the plugin gave us exploits --> priv esc and SQL injection

Reading the priv esc exploit , i knew that this is it. It can allow me to login as anyone without knowing the password because of incorrect usage of wp_set_auth_cookie(). lol

Follow the exploit steps and do it. 
Save it as .html and run the xploit and dont forget to also set the email value with the one we found earlier.

After doing all the steps , we get a dashboard, bingo!

---

Dashboard ENUM:
--
Usually , when I have access to wordpress dashboard , I would look for evil uploads by going to `appearance-->editor` but here we cant edit that file.

after further enum, i found a plugin. `Easy WP SMTP`. Going to its settings , we can see a password field . If we inspect it , we can read the password stored in plaintext xD.

We also had a smtp port open too as well as POP3.


SMTP mail server creds:::: `orestis/kHGuERB29DNiNE`

Googling how to connect to POP3  helped. https://unix.stackexchange.com/questions/201818/checking-pop-mail-account-using-terminal

Using telnet to connect 
```
telnet 10.10.10.17 110
Trying 10.10.10.17...
Connected to 10.10.10.17.
Escape character is '^]'.
+OK Dovecot ready.
USER orestis
+OK
PASS kHGuERB29DNiNE
+OK Logged in.
```

Again used google to find pop3 commands for interaction.
```
- USER: Your user name for the mail server
- PASS: Your password
- QUIT: Ends the session and saves changes
- STAT: Returns the number of messages and their total size
- LIST: Returns a list of messages and their sizes
- RETR message#: Retrieves a selected message
- DELE message#: Deletes a selected message
- NOOP: Keeps the connection open
- RSET: Resets the mailbox and undeletes deleted messages
- TOP msg n: Shows the first n lines of message number msg
- CAPA: Gets capabilities
```

Found 2 messages .
One of the messages contained this :) :
```
From: root@brainfuck.htb (root)

Hi there, your credentials for our "secret" forum are below :)

username: orestis
password: kIEnnfEKJ#9UmdO
```


---
SECRET forum ENUM:
--

As the messages says it is the creds for secret forum, I already found the secret forum.
Also when i tried to login with SSH, it was password login denied. 
It only accepts login through rsa_keys. 

After gaining access to the secret forum, we found exactly this, the private ***RSAkey***.
But upon close inspection , there is a link to a file , to the key, but the url is encrypted using some sort of cipher.  Its Vigenère cipher .

Vigenere cipher needs a passphrase to decrypt.

TO find the passphrase , we can do a python script or use a online tool , i went for the latter.
https://rumkin.com/tools/cipher/vigenere/
The user `orestis` has a signature that he puts in every one of his messages 
`"Orestis - Hacking for fun and profit"`
Now if we look at this same encrypted version of this text , we can find the same number of characters, only the letters are different.

So use these 2 items on that website to get the passphrase needed to decipher the texts and eventually decrypt the url. Remove empty white spaces and the dash -.


The passphrase is `fuckmybrain`  

Using this key, we can finally get the decoded url to get the RSA Private key for ssh login.
```
There you go you stupid fuck, I hope you remember your key password because I dont :)

https://brainfuck.htb/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa
```

----

Path 2 SSH login ENUM:
--
Dont forget to give the key 600 chmod perm.

We cant login this easy. lol , we dont have the ssh id_rsa passphrase xD RIP 

One of the encrypted message from the forum said to **brute force** the key.

Time for johnTheRipper

First conver the id_rsa to id_rsa.hash file with ssh2john

`ssh2john id_rsa > id_rsa.hash`

Then crack it with john

`john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash`

The passkey = `3poulakia!`

Its time for Ssh login ! finally!

---

SSH ENUM (orestis):
--
Upon logon, it says i have a mail. 
Looking at the mail directory in var/spool, we can see 2 users
`nobody & orestis`.

Cant cat nobody and mail in orestis , we alreay read earlier.

---

PRIV ESC :
--
type `id` or `groups` and we can see , user `orestis` is in the `lxd` group.

Google lxd priv esc will show us the way.
https://www.hackingarticles.in/lxd-privilege-escalation/

follow this , download the git repo and build the image .
Then transfer the zip file to the htb machine 

Litterlly follow each step from that article and got root flag.

PWNed!
