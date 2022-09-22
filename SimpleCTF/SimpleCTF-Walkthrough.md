### How many services are running under port 1000?

-   To find how many services are running we can do that by scanning the machine with `nmap`.
-   Syntax: `nmap -T4 -sC -sV -p- -o nmap_all_port <machine_ip>`
    -   `-o` tell `nmap` to write the scanning result to a file which in my case I named it _**nmap_all_port**_
    -   `-p-` indicate scanning all ports
-   `nmap` scan can take some times so it is beneficial to add `-o` flag to write the result to a file of your own specific name so we can check the result later and as often as we want.
-   By default, `nmap` would only scan up to 1000 ports. If all you need is the result of up to 1000 ports, there is no need to add the `-p-` flag.
-   Here is the result

```bash
cat nmap_all_port                                               
# Nmap 7.92 scan initiated Tue Sep 20 01:12:14 2022 as: nmap -T4 -sC -sV -p- -o nmap_all_port 10.10.146.70
Nmap scan report for 10.10.146.70 (10.10.146.70)
Host is up (0.24s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.79.200
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
# Nmap done at Tue Sep 20 01:21:39 2022 -- 1 IP address (1 host up) scanned in 565.02 seconds
```

-   From the result, we can see that there are 2 services under 1000 ports.

### What is running on the higher port?

-   The higher ports refer to any ports number that is higher than 1000.
-   According to the `nmap` result above, the service on the higher port is `ssh`.

### What's the CVE you're using against the application?

-   Now we try to access the <machine_ip> from a web browser. We see apache so we know that it is a web page so we can do directory busting or directory brute forcing.
    
-   To do directory brute forcing, scan the machine with any directory busting tool such as`gobuster`, `ffuf`, `dirb`, etc.
    
-   In my case I used `gobuster`
    
-   Syntax: `gobuster dir -u <machine_ip> -w /usr/share/wordlists/amass/subdomains-top1mil-5000.txt -o gobuster`
    
    -   `-u` flag means target uri
    -   `-w` flag means word list. I chose that one.
    -   `-o` flag write the result on a file. I use this because directory busting can take a long time depends on the target.
    -   I also like to come back to read the result later if I cannot remember the result when I am trying to hack. This would make it easy to just `clear` the terminal and the result will not be gone.
    -   Here is the result
    ![[gobuster.png]]
  
-   After getting the directory result, navigate to that url on a web browser. It said that it is _**CMS version 2.2.8**_. We, then used this information to google its vulnerability and found its CVE.
    
-   `Answer: CVE-2019-9053`
    

### To what kind of vulnerability is the application vulnerable?

-   It is stated on the exploitDB website that this CVE is SQL-injection.
-   SQL-injection short form is SQLi.
-   `Answer: SQLi`

### What's the password?

-   Try `ftp` with anonymous access and it led to a dead end.
-   Download the exploit code from exploitDB or github.
-   Mine is `46635.py`.
-   Syntax: `python3 [46635.py](<http://46635.py/>) -u [<http://10.10.69.120/simple/>](<http://10.10.69.120/simple/>) --crack -w /usr/share/seclists/Passwords/Common-Credentials/best110.txt`
-   When I was hacking this room, I ran into an issue which can be resolved by changing `print something` to `print(something)` which is the syntax change between Python 2 and Python 3.
-   Now I can run it with python 3.
-   While running it, I thought there was an issue with the code that made it froze. It was not an issue. It was just the code trying different passwords from the word list. So, note to self that a seems to be zero progress might not always means the code is hanging. I wouldn’t have known that and would be frustrated had I not went to a restroom and came back to see the progress.

![[get_credential.png]]

-   Now we got a username and a password.

### Where can you login with the details obtained?

-   Try `ssh` and it worked.
-   Connect `ssh` with the credentials from earlier
-   Side note that even though the machine time is expired, ssh session remained so I can continue to explore.

![[connect_ssh.png]]

### What's the user flag?

-   Get user flag in `/home/<username>` directory

### Is there any other user in the home directory? What's its name?

-   Navigate to `/home` directory and `ls`

### What can you leverage to spawn a privileged shell?

-   Try `sudo -l` and see that the current user have sudo privilege on `/usr/bin/vim`
-   Tip I got from Discord when I had just started out was if you find something from `sudo -l` _**GTFOBins**_ is your friend.
-   Escalate the privilege using stuff from _**GTFOBins**_

![[root.png]]

### What's the root flag?

-   Now that we are root, we can get the root flag in `/root` directory.

Writing Date: Sep 20, 2022 17:00 UTC+7

Author: GexterTHM