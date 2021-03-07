# Steps to user.txt
* Run a Port Scan using NMAP
`nmap -sV -sC -oA nmap/output <IP_ADDRESS_OF_MASCHINE>`

* We can see that Ports **21, 22** and **80** are open

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


* Next I had a quick look at the web page
* Only the _Apache2 Ubuntu Default Page_ was shown
* Having a look at the Source showed that I have to add 'team.thm' to my hosts file


![image](https://user-images.githubusercontent.com/78683952/110202466-24dc1100-7e69-11eb-8fb3-ff424a266bd9.png)


* After putting _team.thm_ into my hosts file the following page appeared

![image](https://user-images.githubusercontent.com/78683952/110202498-59e86380-7e69-11eb-9058-0cab1a29e2eb.png)


* Next I ran **gobuster** and found the hidden directory *scripts*
`gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://team.thm`

![image](https://user-images.githubusercontent.com/78683952/110202590-04f91d00-7e6a-11eb-8267-ee5f922b27c5.png)


* Wthin this directory there is a file called *script.txt*
`gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://team.thm/scripts/ -x .txt`

![image](https://user-images.githubusercontent.com/78683952/110202593-104c4880-7e6a-11eb-9aa7-bc1f84518ea0.png)


* Browsing to http://team.thm/scripts/script.txt displays the following:

![image](https://user-images.githubusercontent.com/78683952/110202629-52758a00-7e6a-11eb-81f0-f54c1c889b14.png)

* the hint on the very bottom made me try to access: _http://team.thm/scripts/script.old_

Voilà :) 

![image](https://user-images.githubusercontent.com/78683952/110202655-7df87480-7e6a-11eb-814f-a3717c2f217a.png)

We now have FTP Credentials and can use them to login to the FTP service 

![image](https://user-images.githubusercontent.com/78683952/110202739-fa8b5300-7e6a-11eb-9ab1-b9aef3554d17.png)

* Next I checked out the New_site.txt:

![image](https://user-images.githubusercontent.com/78683952/110239455-38f64000-7f47-11eb-828b-bf4357747af5.png)


It seems like we have to adjust the entry in our hosts file from **team.thm** to **dev.team.thm**
Furthermore, the two names within the .txt file look like usernames. Let's take a note of that.

* Browsing to **dev.team.thm** shows us the following

![image](https://user-images.githubusercontent.com/78683952/110202886-a3d24900-7e6b-11eb-8d29-7f1f9eb68215.png)


![image](https://user-images.githubusercontent.com/78683952/110202911-b8164600-7e6b-11eb-8236-3e5e6ca11af1.png)

The `page=teamshare.php` looks very suspicious.

Lets try: `page=../../../../etc/passwd `

Nice :)

![image](https://user-images.githubusercontent.com/78683952/110203009-14796580-7e6c-11eb-9871-203f47cfba80.png)

- Being able to exploit this **Directory Traversal** / **LFI Vulnerabilty** we can now try to find the **id_rsa** that was mentioned in the **New_site.txt** 
  - view-source:http://dev.team.thm/script.php?page=php://filter/resource=../../../../etc/ssh/sshd_config

![image](https://user-images.githubusercontent.com/78683952/110203090-6d48fe00-7e6c-11eb-9ca7-81992d6ec662.png)

* We can now connect to the machine with the following command: `ssh -i id_rsa dale@dev.team.thm` and get the **user.txt**

![image](https://user-images.githubusercontent.com/78683952/110239403-f46aa480-7f46-11eb-8383-2015871a8ffb.png)


# Steps to root.txt
## Enumeration

What I almost always do first is `sudo -l`

![image](https://user-images.githubusercontent.com/78683952/110238487-3a713980-7f42-11eb-8caf-478c68087569.png)

That looks interesting. Let's have a closer look **/home/gyles/admin_checks**

![image](https://user-images.githubusercontent.com/78683952/110238529-6c829b80-7f42-11eb-8113-feb52c8605ab.png)

The lines outlined in red look interesting. It seems as if the provided user input will be executed without any further sanitization.

So, I started the script with the following line:
`sudo -u gyles /home/gyles/admin_checks`

and then provided `/bin/bash` as the _date argument_

![image](https://user-images.githubusercontent.com/78683952/110238654-3265c980-7f43-11eb-9d0e-b42a210da0a1.png)

We are now in the context of gyles. 

![image](https://user-images.githubusercontent.com/78683952/110238742-883a7180-7f43-11eb-9e87-d9c335c01a69.png)

Gyles is a member of the admin group. Let's take a note of that.

I continued my search for possible privilege escalation vectors using linpeas **linpeas.sh** (https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS).

The screenshot below shows something interesting:

![image](https://user-images.githubusercontent.com/78683952/110238966-bec4bc00-7f44-11eb-98e4-e6a268b55895.png)

Inside the **/opt/admin_stuff/** directory I found **script.sh**

![image](https://user-images.githubusercontent.com/78683952/110239001-f2074b00-7f44-11eb-954e-ae5f836ed34a.png)

Interesting. So **/usr/local/bin/main_backup.sh** is executed every minute.

Let's find out what it does:

![image](https://user-images.githubusercontent.com/78683952/110239046-372b7d00-7f45-11eb-846c-1880ebeb2722.png)

It basically just copies the content of one directory into another.

Checking out the destination directory showed me that the task is being performed as **root**.

![image](https://user-images.githubusercontent.com/78683952/110239526-a4401200-7f47-11eb-87f2-2c0de2fed090.png)


And the good thing about it is, being a member of the admin group gives us write access to **/usr/local/bin/main_backup.sh**

![image](https://user-images.githubusercontent.com/78683952/110239698-8f17b300-7f48-11eb-8209-17530a0bf160.png)


So all that's left to do is
* starting a netcat listener
* putting a reverse shell into **main_backup.sh**
* wait maximum 1 minute

![image](https://user-images.githubusercontent.com/78683952/110239760-e9187880-7f48-11eb-94a6-bcc272dc680e.png)


* and of course get root.txt :)

![image](https://user-images.githubusercontent.com/78683952/110239296-773f2f80-7f46-11eb-875b-4697e646581d.png)


