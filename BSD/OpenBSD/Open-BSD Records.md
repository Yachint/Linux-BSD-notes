# OpenBSD Notes

## Starting and stopping wireguard
```
# Start the interface
doas sh /etc/netstart wg0

# Stop the interface 
doas ifconfig wg0 destroy
```

Reference:
- https://rakhesh.com/linux-bsd/setting-up-wireguard-on-openbsd-and-linux/

## Set static DNS to Quad9
Disabling resolvd is necessary as mentioned in the reference otherwise it will keep re-writing our changes.

```
rcctl stop resolvd
```

Edit the resolv.conf file and put custom DNS

```
vi /etc/resolv.conf
```

Contents:
```
nameserver 9.9.9.9
nameserver 149.112.112.112
```

Test DNS:
```
nslookup www.google.com
```

References:
- https://www.reddit.com/r/openbsd/comments/qzbm9e/dhcpleased_with_ignore_dns/