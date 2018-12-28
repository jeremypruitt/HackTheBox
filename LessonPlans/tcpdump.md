## Node

#### OPTIONAL: Modify /etc/hosts to help tcpdump name resolution
```sh
$ vi /etc/hosts
```

#### Listen for packets on specific interface
```sh
$ tcpdump -i $INTERFACE
```

#### In separate window make TCP connection to target
```sh
$ nc $TARGET_HOST $TARGET_PORT
# Review tcpdump output. Notice the TCP handshake of SYN, SYN/ACK, ACK
```

#### Now try to curl a webpage
```sh
$ curl $TARGET_HOST/$URI_PATH
# Notice the TCP handshake, followed by HTTP push (P) and chunked receipt of data
```

#### Restart tcpdump to show ASCII or HEX data
```sh
# To show ASCII output for received packets, use `-A`:
$ tcpdump -i $INTERFACE -A

# To show ASCII& HEX output for received packets, use `-XX`:
$ tcpdump -i $INTERFACE -XX
```
