## Node

#### Scan to discover open ports
```sh
nmap -sS -P0 -n -p- -T5 -oA nmap-tcp-allports-fast 10.10.10.87
```

#### Enumerate the discovered ports
```sh
nmap -sV -n -sC -p22,3000 -oA nmap-tcp-foundports 10.10.10.87
```

#### Find SUID programs now that we're Tom
```sh
$ find / -perm -4000 2>/dev/null
```
