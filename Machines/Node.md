## Node

#### Scan to discover open ports
```
nmap -sS -P0 -n -p- -T5 -oA nmap-tcp-allports-fast 10.10.10.58
```

#### Enumerate the discovered ports
```
nmap -sV -n -sC -p22,3000 -oA nmap-tcp-foundports 10.10.10.58
```

Gobuster/dirbuster doesn't work as every response returns 200.

So fire up burp to intercept a request to `http://10.10.10.58:3000/`. Notice that there are 2 responses; one for `/` as expected but then another one for `/<uuid>`.

Navigate to the `/<uuid>` link from a browser through Burp. Go to the Target -> Site Map tabs and expand the 10.10.10.58 host. Notice the `/api/users/latest` file and you should see json content. If not then you can run:
```
$ curl -s http://10.10.10.58:3000/api/users/latest | json_pp
```
